<template>
  <div class="banner-wrap">
    <canvas id="recordCanvas"></canvas>
    <!-- <div class="flex fixed w-full justify-center top-2">
      <div class="banner-wrap-group">jljkjl</div>
    </div> -->
    <div class="flex fixed w-full justify-center bottom-2">
      <div class="flex items-center banner-wrap-group">
        <div class="time">{{ timeText }}</div>
        <a-button
          type="primary"
          title="开始"
          @click="onRecord"
          :disabled="isRecording"
        >
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
import { computed } from "@vue/reactivity";
import { saveAs } from "file-saver";
import { ExportTypeEnum, VideoFormatEnum } from "@/components/Export/enum";
import { useRoute } from "vue-router";
import { createFFmpeg, fetchFile } from "@ffmpeg/ffmpeg";
const ffmpeg = createFFmpeg({ log: true });
const store = useStore();
const route = useRoute();
const { f, fps, type, enableMouse } = route.query;

let stream = ref<MediaStream>();
let recorder: MediaRecorder;

let isRecording = ref(false);
let blobs: Blob[] = [];
let time = ref(0);
let timeInterval: number | undefined;

let timeText = computed(() => {
  let second = time.value % 60;
  let minute = (time.value - second) / 60;
  return `${minute >= 10 ? minute : "0" + minute}:${
    second >= 10 ? second : "0" + second
  }`;
});

onMounted(async () => {
  let allDataConfig;
  try {
    allDataConfig = JSON.parse(localStorage.getItem("recordAllData") || "");
  } catch (error) {
    console.warn("没有配置");
  }

  await PreviewManager.ins.createEngine("recordCanvas", allDataConfig);
  PreviewManager.ins.setMouseControlCamera(enableMouse == "true");

  store.toggleLoading(false);
});

/**开启录制 */
function onRecord() {
  if (!stream.value) {
    // 将画布内容生成媒体流
    stream.value =
      PreviewManager.ins.engine.canvas._webCanvas.captureStream(fps);

    // 生成媒体录制器对象并传入一个媒体流以便录制
    stream.value &&
      (recorder = new MediaRecorder(stream.value, {
        mimeType: `video/webm;codecs=vp8`,
      }));
    // 有可录制的媒体资源事件
    recorder.ondataavailable = (event) => {
      blobs.push(event.data);
    };
    // 媒体录制器停止录制事件
    recorder.onstop = (event) => {
      isRecording.value = false;
      if (type == ExportTypeEnum.GIF) {
        transcode();
      } else {
        saveVideo();
      }
    };
  }
  // 媒体录制器开始录制
  isRecording.value = true;
  recorder.start();
  
  // 清空之前保存的数据流
  blobs = [];

  time.value = 0;
  timeInterval = setInterval(() => {
    time.value++;
  }, 1000);
}
function saveVideo() {
  saveAs(new Blob(blobs, { type: `video/${f}` }));
  // open(URL.createObjectURL(new Blob(blobs, { type: "video/mp4" })));
}

/**
 * 停止录制
 */
function stopRecord() {
  clearInterval(timeInterval);
  recorder.stop();
}

let count = 0;
let formatValue = ref("image/gif");
const transcode = async () => {
  const name = `test`;
  count++;
  !ffmpeg.isLoaded() && (await ffmpeg.load());
  ffmpeg.FS(
    "writeFile",
    name,
    await fetchFile(new Blob(blobs, { type: `${formatValue.value}` }))
  );
  // if (!formatValue.value.includes("gif")) {
  //   ffmpeg.FS("writeFile", 'audio', await fetchFile('./video/record.mp3'));
  // }
  // await ffmpeg.run('-i', name, '-i', 'audio', '-c:v', 'copy', '-c:a', 'aac', '-strict', 'experimental', '-map', '0:v:0', '-map', '1:a:0', `output.${formatValue.value.split("/")[1]}`);
  await ffmpeg.run("-i", name, `output.${formatValue.value.split("/")[1]}`);
  const data = ffmpeg.FS(
    "readFile",
    `output.${formatValue.value.split("/")[1]}`
  );
  // ffmpeg.FS('unlink', name);
  let b = new Blob([data.buffer], { type: formatValue.value });
  saveAs(b);
  // open(URL.createObjectURL(b));
};
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
      padding-right: 15px;
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
