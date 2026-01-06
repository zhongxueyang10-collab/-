<template>
  <div class="map-container">
   
    <div ref="cesiumContainer" style="width: 100vw; height: 100vh"></div>
     <!-- 利用prop将viewer数据传递给Test4组件 ，v-if防止地图渲染完成前Test4组件渲染，否则会报错-->
    <Test4  :viewer="viewer" ></Test4>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import * as Cesium from "cesium";
import Test4 from './test4-html标签渲染.vue';
import { cameraPositions } from "../utils/data.js";

const cesiumContainer = ref(null);
let viewer = null;

const token = "05be06461004055923091de7f3e51aa6";
let billboard = [];
const p1 = cameraPositions;

Cesium.Ion.defaultAccessToken =
  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIwZTEwODgwMS1iYTY0LTRhNmYtYWFhMS03MDEzMjlhYWNjOTciLCJpZCI6MzAwMTM5LCJpYXQiOjE3NDY1ODI5MTR9.P4bdCMYyoubNMaQ_-ZkU99mM8Da3o8HIo4A57stHRAg";

onMounted(() => {
  // 初始化Viewer
  viewer = new Cesium.Viewer(cesiumContainer.value, {
    baseLayer: false,
    baseLayerPicker: false,
    geocoder: false,
    homeButton: false,
    sceneModePicker: false,
    navigationHelpButton: false,
    animation: false,
    timeline: false,
    fullscreenButton: false,
  });

  // 隐藏logo
  viewer.cesiumWidget.creditContainer.style.display = "none";

  // 添加地图点击事件，点击后获取经纬度坐标
  viewer.screenSpaceEventHandler.setInputAction(function (e) {
    const cartesian = viewer.scene.pickPosition(e.position);

    if (Cesium.defined(cartesian)) {
      const cartographic = Cesium.Cartographic.fromCartesian(cartesian);
      const longitude = Cesium.Math.toDegrees(cartographic.longitude).toFixed(6);
      const latitude = Cesium.Math.toDegrees(cartographic.latitude).toFixed(6);
      const height = cartographic.height.toFixed(2);

      console.log(`经度: ${longitude}, 纬度: ${latitude}, 高度: ${height}米`);
    } else {
      console.log("无法获取有效坐标（可能点击了天空或地形外区域）");
    }
  }, Cesium.ScreenSpaceEventType.LEFT_CLICK);

  // 添加模型
  const modelEntity = viewer.entities.add({
    position: Cesium.Cartesian3.fromDegrees(104.6987, 31.538438, -20),

    orientation: Cesium.Transforms.headingPitchRollQuaternion(
      Cesium.Cartesian3.fromDegrees(104.6987, 31.538438, -20),
      new Cesium.HeadingPitchRoll(
        Cesium.Math.toRadians(180),
        Cesium.Math.toRadians(0),
        Cesium.Math.toRadians(0)
      )
    ),

    model: {
      uri: "/library.glb",
      scale: 1.0,
      minimumPixelSize: 1,
      maximumScale: 20000,
    },
  });

  // 创建摄像头广告牌
  for (let i = 0; i < 12; i++) {
    billboard[i] = viewer.entities.add({
      position: Cesium.Cartesian3.fromDegrees(p1[i].lon, p1[i].lat, p1[i].height),
      billboard: {
        image: '/image/摄像头(1).png',
        scale: 0.5,
        color: Cesium.Color.WHITE,
        width: 100,
        height: 100,
        horizontalOrigin: Cesium.HorizontalOrigin.CENTER,
        verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
      },
    });
  }

  // 方式一：使用HeadingPitchRange（推荐）
viewer.camera.lookAt(
  Cesium.Cartesian3.fromDegrees(104.6987, 31.538438,0),
  new Cesium.HeadingPitchRange(
    Cesium.Math.toRadians(0), // 方位角：30°（东北方向）
    Cesium.Math.toRadians(-20), // 俯仰角：-20°（俯视）
    500 // 距离目标点的距离（米）
  )
);
  initMap();
});

// 通过按键移动相机
document.addEventListener('keydown', (e) => {
  if (e.key == 'w') {
    viewer.camera.moveForward(100);
  } else if (e.key == 's') {
    viewer.camera.moveBackward(100);
  } else if (e.key == 'a') {
    viewer.camera.moveLeft(50);
  } else if (e.key == 'd') {
    viewer.camera.moveRight(50);
  }
});

function destroyViewer() {
  if (viewer) {
    viewer.destroy();
    viewer = null;
    console.log("Viewer实例已销毁");
  }
}

onUnmounted(() => {
  destroyViewer();
});

const initMap = () => {
  const tiandituProvider = new Cesium.WebMapTileServiceImageryProvider({
    url:
      "http://{s}.tianditu.gov.cn/img_w/wmts?service=wmts&request=GetTile&version=1.0.0&LAYER=img&tileMatrixSet=w&TileMatrix={TileMatrix}&TileRow={TileRow}&TileCol={TileCol}&style=default&format=tiles&tk=" +
      token,
    layer: "img",
    style: "default",
    format: "tiles",
    tileMatrixSetID: "w",
    subdomains: ["t0", "t1", "t2", "t3", "t4", "t5", "t6", "t7"],
    maximumLevel: 18,
    credit: new Cesium.Credit("天地图影像"),
  });

  const labelProvider = new Cesium.WebMapTileServiceImageryProvider({
    url:
      "http://{s}.tianditu.gov.cn/cia_w/wmts?service=wmts&request=GetTile&version=1.0.0&LAYER=cia&tileMatrixSet=w&tileMatrix={TileMatrix}&tileRow={TileRow}&tileCol={TileCol}&style=default&format=tiles&tk=" +
      token,
    layer: "img",
    style: "default",
    format: "tiles",
    tileMatrixSetID: "w",
    subdomains: ["t0", "t1", "t2", "t3", "t4", "t5", "t6", "t7"],
    maximumLevel: 18,
    credit: new Cesium.Credit("天地图标注"),
  });

  viewer.imageryLayers.addImageryProvider(tiandituProvider);
  viewer.imageryLayers.addImageryProvider(labelProvider);
};
</script>

<style scoped>
.map-container {
  width: 100vw;
  height: 100vh;
  margin: 0;
  padding: 0;
  overflow: hidden;
}
</style>
