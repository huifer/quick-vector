<script lang="ts" setup>
import { ipcRenderer } from 'electron';
import { onMounted, reactive, ref } from 'vue';
import { findNodeById, findNodeByLabel, removeNodesByLabel } from '../layer/map/NodeUtil';
import { getQvmap } from '../layer/map/ConstValue';
import { ElMessage } from 'element-plus';

onMounted(() => {
  ipcRenderer.send('getLayers');
});
const LayerTree = ref();
ipcRenderer.on('curLayers', (event, args) => {
  console.log('当前图层', args);
  LayerTree.value = removeNodesByLabel(args.data, ['标记图层', 'WMS/WMTS', '底图']);
  findNodeByLabel(LayerTree.value, '数据库图层').disabled = true;
  findNodeByLabel(LayerTree.value, '文件图层').disabled = true;
});

const data = reactive({
  unity: 'meters',
  size: 0,
  layerName: '',
  geojson: '',
});
const units = [
  { label: '米', value: 'meters' },
  { label: '毫米', value: 'millimeters' },
  { label: '厘米', value: 'centimeters' },
  { label: '千米', value: 'kilometers' },
  { label: '英亩', value: 'acres' },
  { label: '英里', value: 'miles' },
  { label: '海里', value: 'nauticalmiles' },
  { label: '英寸', value: 'inches' },
  { label: '码', value: 'yards' },
  { label: '英尺', value: 'feet' },
  { label: '弧度', value: 'radians' },
  { label: '度', value: 'degrees' },
  { label: '公顷', value: 'hectares' },
];

const ok = () => {
  let flg = false;

  if (!data.unity) {
    ElMessage({
      message: '单位必填',
      grouping: true,
      type: 'error',
    });

    return;
  }
  if (!data.size) {
    ElMessage({
      message: '长度必填',
      grouping: true,
      type: 'error',
    });

    return;
  }
  if (!data.layerName) {
    ElMessage({
      message: '图层必填',
      grouping: true,
      type: 'error',
    });

    return;
  }
  flg = true;
  if (flg) {
    ipcRenderer.send(
      'buffer-config-data-complete',
      JSON.parse(
        JSON.stringify({
          unity: data.unity,
          size: data.size,
          layerName: findNodeById(LayerTree.value, data.layerName).label,
          close: false,
          id: data.layerName,
        })
      )
    );
  }
};

const error = () => {
  ipcRenderer.send('buffer-config-data-complete', {
    close: true,
  });
};
const nodeClick = (node) => {
  console.log(node);
};
const rules = {
  layerName: [{ required: true, message: '必填', trigger: 'blur' }],
  size: [{ required: true, message: '必填', trigger: 'blur' }],
  unity: [{ required: true, message: '必填', trigger: 'blur' }],
};
</script>

<template>
  <div>图层缓冲区</div>
  <el-form :model="data" :rules="rules" label-width="120px">
    <el-form-item label="缓冲图层" prop="layerName">
      <el-tree-select
        v-model="data.layerName"
        :data="LayerTree"
        :node-click="nodeClick"
        :render-after-expand="false"
        node-key="id"
      />
    </el-form-item>
    <el-form-item label="缓冲长度" prop="size">
      <el-input-number v-model="data.size" :step="0.1" />
    </el-form-item>
    <el-form-item label="单位" prop="unity">
      <el-select v-model="data.unity" placeholder="请选择长度单位">
        <el-option v-for="unit in units" :key="unit.value" :label="unit.label" :value="unit.value"></el-option>
      </el-select>
    </el-form-item>
  </el-form>
  <div>
    <el-button @click="ok">确定</el-button>
    <el-button @click="error">取消</el-button>
  </div>
</template>

<style scoped></style>
