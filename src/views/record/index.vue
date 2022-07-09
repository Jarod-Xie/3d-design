<template>
  <div class="banner-wrap">
    <canvas id="recordCanvas"></canvas>
    <div class="flex fixed w-full justify-center top-2">
      <div class="banner-wrap-group">jljkjl</div>
    </div>
    <div class="flex fixed w-full justify-center bottom-2">
      <div class="flex items-center banner-wrap-group">
        <div class="time">00:00</div>
        <a-button type="primary" @click="onRecord" :disabled="isRecording">
          <div class="record-icon"></div>
        </a-button>
        <a-button type="primary" @click="stopRecord" :disabled="!isRecording"
          >停止</a-button
        >
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { PreviewManager } from "@/components/ModelPreview/preview/index";
import { onMounted, ref } from "vue";
import { useStore } from "@/store";

const store = useStore();

let stream = ref<MediaStream>();
let recorder: MediaRecorder;

let isRecording = ref(false);
let blobs: Blob[] = [];

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

function onRecord() {
  if (!stream.value) {
    // 将画布内容生成媒体流
    stream.value =
      PreviewManager.ins.engine.canvas._webCanvas.captureStream(60);

    // 生成媒体录制器对象并传入一个媒体流以便录制
    stream.value &&
      (recorder = new MediaRecorder(stream.value, {
        mimeType: "video/webm;codecs=vp8",
      }));
    // 有可录制的媒体资源事件
    recorder.ondataavailable = (event) => {
      blobs.push(event.data);
    };
    // 媒体录制器停止录制事件
    recorder.onstop = (event) => {
      isRecording.value = false;
      saveVideo();
    };
  }
  // 媒体录制器开始录制
  isRecording.value = true;
  recorder.start();
  console.log(isRecording.value);
}
function saveVideo() {
  open(URL.createObjectURL(new Blob(blobs, { type: "video/mp4" })));
  blobs = [];
}

/**
 * 停止录制
 */
function stopRecord() {
  recorder.stop();
}
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
    .time {
      padding: 0 15px;
    }
    .record-icon {
      width: 14px;
      height: 14px;
      background-color: red;
      border-radius: 100%;
      border: 1px solid white;
    }
    .ant-btn-primary[disabled] {
      border: none;
    }
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
