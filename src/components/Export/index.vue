<template>
  <a-form :model="form" layout="vertical">
    <a-form-item label="导出类型">
      <a-select :style="{ width: '100%' }" v-model:value="form.exportType">
        <a-select-option v-for="(i, key) in ExportTypeEnum" :value="i">{{
          key
        }}</a-select-option>
      </a-select>
    </a-form-item>
    <template v-if="form.exportType == ExportTypeEnum.图片">
      <a-form-item label="图片宽度">
        <a-input-number
          v-model:value="form.width"
          placeholder="请输入图片宽度,范围1 - 2048"
          style="width: 100%"
          :min="1"
          :max="2048"
        />
      </a-form-item>
      <a-form-item label="图片高度">
        <a-input-number
          v-model:value="form.height"
          placeholder="请输入图片高度,范围1 - 2048"
          style="width: 100%"
          :min="1"
          :max="2048"
        />
      </a-form-item>
      <a-form-item label="导出格式">
        <a-select
          :style="{ width: '100%' }"
          placeholder="请选择导出格式"
          v-model:value="form.isPNG"
        >
          <a-select-option value="true">PNG格式</a-select-option>
          <a-select-option value="false">JPG格式</a-select-option>
        </a-select>
      </a-form-item>
      <a-form-item label="图片大小">
        <a-select :style="{ width: '100%' }" v-model:value="size">
          <a-select-option :value="0.5">0.5倍</a-select-option>
          <a-select-option :value="1">原图</a-select-option>
          <a-select-option :value="1.5">1.5倍</a-select-option>
          <a-select-option :value="2">2倍</a-select-option>
        </a-select>
      </a-form-item>
      <a-form-item label="jpg格式的图片质量" v-if="form.isPNG != 'true'">
        <a-slider v-model:value="form.jpgQuality" :max="1" :step="0.1" />
      </a-form-item>
      <div class="flex justify-end">
        <a-button
          type="primary"
          @click="handleClick"
          style="background-color: #ff6900"
          >导出图片</a-button
        >
      </div>
    </template>
    
    <template v-else-if="form.exportType == ExportTypeEnum.视频">
      <a-form-item label="视频格式">
        <a-select
          :style="{ width: '100%' }"
          placeholder="请选择导出格式"
          v-model:value="form.format"
        >
          <a-select-option
            v-for="(i, key) in VideoFormatEnum"
            :key="i"
            :value="i"
            >{{ key }}</a-select-option
          >
        </a-select>
      </a-form-item>
    </template>
    <template v-if="form.exportType == ExportTypeEnum.视频 || form.exportType == ExportTypeEnum.GIF">
      <a-form-item label="FPS">
        <a-select :style="{ width: '100%' }" v-model:value="form.fps">
          <a-select-option :value="15">15</a-select-option>
          <a-select-option :value="24">24</a-select-option>
          <a-select-option :value="30">30</a-select-option>
          <a-select-option :value="48">48</a-select-option>
          <a-select-option :value="60">60</a-select-option>
        </a-select>
      </a-form-item>
      <a-form-item label="鼠标操作">
        <a-switch v-model:checked="form.enableMouse" checked-children="开启" un-checked-children="关闭" />
      </a-form-item>
      <div class="flex justify-end">
        <a-button
          type="primary"
          @click="handleVideoGifClick"
          style="background-color: #ff6900"
          >开始</a-button
        >
      </div>
    </template>
  </a-form>
</template>

<script lang="ts" setup>
import { defineComponent, reactive, ref } from "vue";
import { GameManager } from "@/oasis/index";
import { ExportTypeEnum, VideoFormatEnum } from "./enum";
import { useRouter } from "vue-router";

const router = useRouter();


const props = defineProps({
  generateAllConfig: Function,
});

let form = reactive({
  // 公共
exportType: ExportTypeEnum.图片,
  // 图片配置
  width: 1024,
  height: 1024,
  isPNG: "true",
  jpgQuality: 1,

  // 视频配置
  format: VideoFormatEnum.MP4,
  fps: 30,
  enableMouse: true

});

let size = ref(1);

const handleClick = () => {
  let ispng = form.isPNG === "true";

  let width = form.width * size.value;
  let height = form.height * size.value;

  GameManager.ins.screenshot(width, height, ispng, form.jpgQuality);
};
const handleVideoGifClick = () => {
  if (props.generateAllConfig) {
    localStorage.setItem(
      "recordAllData",
      JSON.stringify(props.generateAllConfig())
    );
    const { format, fps, exportType, enableMouse } = form
    let query = `fps=${fps}&enableMouse=${enableMouse}`
    if(exportType == ExportTypeEnum.GIF) {
      query += `&type=${exportType}`
    } else {
      query += `&f=${format}`
    }
    window.open(router.resolve(`/record?${query}`).href);
  }
};
</script>

<style scoped></style>
