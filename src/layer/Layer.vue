<script lang="ts" setup>
import { h, nextTick, onMounted, reactive, ref, watch } from 'vue';
import { QvMap } from './map/QvMap';
import { ipcRenderer } from 'electron';
import LayerLeft from './LayerLeft.vue';
import AttrRange from '../attr/AttrRange.vue';
import { ReadDiTuConfig, WriteDiTuConfig } from '../utils/FileUtils';
import { GetTdtToken } from './map/Tdt';
import jkl from './map/a.js';
import { setQvMap } from './map/ConstValue';
import TestView from '../test/TestLineRing.vue';

const map = ref<any>();
const token = ref(0);
let mapData = reactive({
  coordinates: [],
  zoom: -1,
  click: false,
  // 是否开启选择元素
  openSelect: false,
  // 选择要素的数据
  selectData: {},

  // 是否选中要素
  isSelect: false,
  // 是否开启坐标拾取
  isOpenCoordinatePicku: false,
});
const winSize = reactive({
  w: null,
  h: null,
});

let qvMap = new QvMap('map', mapData);
// 接受地图配置
ipcRenderer.on('map-config', function (event, arg) {
  console.log('event:', event);
  console.log('地图配置数据:', arg);
  let readDiTuConfig = ReadDiTuConfig();
  if (readDiTuConfig['token']) {
    readDiTuConfig['token'][arg.type] = arg.token;
  } else {
    readDiTuConfig['token'] = {};
    readDiTuConfig['token'][arg.type] = arg.token;
  }
  WriteDiTuConfig(readDiTuConfig);
  console.log('配置文件中 地图配置 = ', readDiTuConfig);
});
ipcRenderer.on('calc-windows-size', function (event, args) {
  winSize.w = args.w;
  winSize.h = args.h;
  attrShowSize.x = winSize.w - attrShowSize.initW - 20;
  attrShowSize.y = 35;
});
ipcRenderer.on('map_to_xy', function (event, arg) {
  console.log('event:', event);
  console.log('arg:', arg);
  qvMap.moveToXY(arg.x, arg.y);
});
ipcRenderer.on('file-select', function (event, args) {
  console.log('文件：', args);
});

ipcRenderer.on('gen-pointOrLine-show', function (event, args) {
  console.log('成图完毕');
  console.log('文件地址', args.fileName);
  console.log('数据', args.geo);
  qvMap.addGeoJsonForImport(args.uid, args.geo, args.type);
});

ipcRenderer.on('gen-shp-show', (event, args) => {
  console.log('shp成图');
  console.log(args);
  qvMap.addGeoJsonForImport(args.uid, args.geo, args.type);
});

ipcRenderer.on('gen-geojson-show', (event, args) => {
  console.log('geojson成图');
  console.log(args);
  qvMap.addGeojsonFile(args.uid, args.geo);
});

onMounted(() => {
  console.log('初始化地图');
  map.value = qvMap.initMap();
});
const fileOpen = () => {
  ipcRenderer.send('openDialog');
};
const a = () => {};
const b = () => {};
const d = reactive({
  x: 100,
  y: 100,
  h: 100,
  w: 100,
  active: false,
});
const attrShowSize = reactive({
  x: 300,
  y: 35,
  h: 100,
  w: 100,
  initW: 500,
  initH: 380,
  active: false,
});
const print = (e) => {
  console.log(e);
};
watch(mapData, (o, n) => {
  if (n.isSelect) {
    onceFeature.value = JSON.parse(n.selectData);
    // 发送事件获取窗口尺寸
    ipcRenderer.send('calc-windows-size');
    if (mapData.openSelect) {
      attrArrayDisplay.value = true;
    }
    n.isSelect = false;
  }
});
const onceFeature = ref({
  type: 'Feature',
  properties: {
    a: 3,
  },
  geometry: {
    coordinates: [[119.45436769887343, 29.2080525919085]],
    type: 'Point',
  },
});
const attrArrayDisplay = ref(false);
const mapConfig = ref({
  tdt_token: '',
});
const test = () => {
  mapConfig.value.tdt_token = GetTdtToken();
};

ipcRenderer.on('openOrCloseSelect', function (event, args) {
  console.log('开关选择器');

  qvMap.openOrClose();
});
ipcRenderer.on('openOrCloseCoordinatePickup', (event, args) => {
  console.log('开关地图拾取');
});
ipcRenderer.on('closeCoordinatePickup', (event, args) => {});

// 属性查看模式关闭
ipcRenderer.on('closeSelect', (event, args) => {
  qvMap.closeSelector();
});
const tt = () => {
  ipcRenderer.send('menu-item-state-changed');
};

ipcRenderer.on('curLayers', (event, args) => {
  console.log('当前图层', args);
});

ipcRenderer.on('curLayersGeojson', (event, args) => {
  console.log('当前geojson', args);
});

ipcRenderer.on('Transit-ex-geojson', (event, args) => {
  ipcRenderer.send('ex-geojson-end', args);
});
</script>

<template>
  <div id="map" ref="map" style="height: 100vh; width: 100%">
    <!--  todo: 尺寸动态 -->
    <Vue3DraggableResizable
      id="a"
      v-model:active="d.active"
      v-model:h="d.h"
      v-model:w="d.w"
      v-model:x="d.x"
      v-model:y="d.y"
      :draggable="true"
      :initH="600"
      :initW="200"
      :resizable="true"
      @activated="print('activated')"
      @deactivated="print('deactivated')"
      @dragging="print('dragging')"
      @resizing="print('resizing')"
      @drag-start="print('drag-start')"
      @resize-start="print('resize-start')"
      @drag-end="print('drag-end')"
      @resize-end="print('resize-end')"
    >
      <div>
        <LayerLeft :qv-map="qvMap"></LayerLeft>
      </div>
    </Vue3DraggableResizable>

    <div v-if="attrArrayDisplay">
      <Vue3DraggableResizable
        id="b"
        v-model:active="attrShowSize.active"
        v-model:h="attrShowSize.h"
        v-model:w="attrShowSize.w"
        v-model:x="attrShowSize.x"
        v-model:y="attrShowSize.y"
        :draggable="true"
        :initH="attrShowSize.initH"
        :initW="attrShowSize.initW"
        :resizable="true"
        @activated="print('activated')"
        @deactivated="print('deactivated')"
        @dragging="print('dragging')"
        @resizing="print('resizing')"
        @drag-start="print('drag-start')"
        @resize-start="print('resize-start')"
        @drag-end="print('drag-end')"
        @resize-end="print('resize-end')"
      >
        <div>
          <div style="position: relative">
            <!-- 此处是你的内容 -->
            <div class="close-button" @click="attrArrayDisplay = false">X</div>
          </div>
          <attr-range :geometry="onceFeature"></attr-range>
        </div>
      </Vue3DraggableResizable>
    </div>
    <div id="flood">
      <span>zoom:{{ mapData.zoom }}</span>
      <span>x:{{ mapData.coordinates[0] }}</span
      ><span>,</span><span>y:{{ mapData.coordinates[1] }}</span>
    </div>
  </div>
</template>

<style scoped>
#a {
  position: fixed;
  z-index: 999;
  background-color: #fff;
  height: 100%;
  overflow-y: auto;
}

#b {
  position: fixed;
  z-index: 999;
  background-color: #fff;
  height: 100%;
  overflow-y: auto;
}

#flood {
  position: absolute;
  bottom: 0;
  right: 0;
  width: 300px;
  height: 20px;
  background-color: rgba(238, 255, 0, 0.5);
}

.close-button {
  position: absolute;
  top: 0;
  right: 0;
  padding: 4px;
  background-color: #ccc;
  cursor: pointer;
}
</style>
