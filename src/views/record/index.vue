<template>
  <div class="banner-wrap">
    <canvas id="recordCanvas"></canvas>
    <div class="flex fixed w-full justify-center top-2">
      <div class="banner-wrap-group">jljkjl</div>
    </div>
    <div class="flex fixed w-full justify-center bottom-2">
      <div class="banner-wrap-group">123456</div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { PreviewManager } from "@/components/ModelPreview/preview/index";
import { onMounted, ref } from "vue";
import { useStore } from "@/store";

const store = useStore();

onMounted(async () => {
  let allDataConfig;
  try {
    allDataConfig = JSON.parse(localStorage.getItem("recordAllData") || "");
  } catch (error) {
    console.warn("没有配置");
  }

  await PreviewManager.ins.createEngine("recordCanvas", allDataConfig);

  store.toggleLoading(false);
});
</script>

<style scoped lang="scss">
.banner-wrap {
  width: 100%;
  height: 100vh;
  position: relative;
  overflow: hidden;
  background-size: cover;
  background: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAYAAADgdz34AAAABGdBTUEAALGPC/xhBQAAAFRJREFUSA1jtLOz+8+ABXz9+hWLKAMDNzc3VnFc6pmwqqai4KgFBAOT5kHEgiv2SU0tuNTT3AejFgyCVIQr9qmVukYjeeAjmeZxMFoWjYBIpnkqAgBYXhnO66bDGwAAAABJRU5ErkJggg==);
  &-group {
    height: 46px;
    padding: 10px 20px;
    background-color: var(--primary-color);
    border-radius: 46px;
  }
}
#recordCanvas {
  position: absolute;
  width: 100%;
  height: 100%;
  min-width: 40%;
  background: transparent;
}
</style>
