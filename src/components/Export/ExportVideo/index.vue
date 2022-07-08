<template>
  <a-form :model="form" layout="vertical">
    <a-form-item label="视频格式">
      <a-select
        :style="{ width: '100%' }"
        placeholder="请选择导出格式"
        v-model:value="form.format"
      >
        <a-select-option v-for="(i, key) in VideoFormatEnum" :key="i" :value="i">{{key}}</a-select-option>
      </a-select>
    </a-form-item>
    <a-form-item label="FPS">
      <a-select :style="{ width: '100%' }" v-model:value="form.fps">
        <a-select-option :value="15">15</a-select-option>
        <a-select-option :value="24">24</a-select-option>
        <a-select-option :value="30">30</a-select-option>
        <a-select-option :value="48">48</a-select-option>
        <a-select-option :value="60">60</a-select-option>
      </a-select>
    </a-form-item>
  </a-form>
  <div class="flex justify-end">
    <a-button
      type="primary"
      @click="handleClick"
      style="background-color: #ff6900"
      >开始</a-button
    >
  </div>
</template>

<script lang="ts" setup>
import { defineComponent, reactive, ref } from "vue";
import { GameManager } from "@/oasis/index";
import { VideoFormatEnum } from './enum'


let form = reactive({
  format: VideoFormatEnum.MP4,
  fps: 30,
});

const handleClick = () => {};
</script>

<style scoped></style>
