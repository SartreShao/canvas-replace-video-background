<template>
  <div>
    <video
      id="video"
      src="./assets/video.mp4"
      controls="true"
      crossorigin="anonymous"
      ref="video"
    />
  </div>
  <div>
    <canvas id="c1" width="160" height="96" ref="canvas1"></canvas>
    <canvas id="c2" width="160" height="96" ref="canvas2"></canvas>
    <canvas width="160" height="96" ref="canvas3"></canvas>
  </div>
  <div @click="drawCurrentVideoInCanvas">drawCurrentVideoInCanvas</div>
</template>

<script setup>
import { onMounted, ref } from "vue";
// 视频播放器
const video = ref(null);

// 画布 1：原始画布
const canvas1 = ref(null);

// 画布 2：将绿色变为透明色的画布
const canvas2 = ref(null);

// 画布 3：
const canvas3 = ref(null);

// eslint-disable-next-line no-unused-vars
const drawCurrentVideoInCanvas = () => {
  if (canvas3.value) {
    // 在这一步也可以创造一个 webgl 的东西
    const context = canvas3.value.getContext("2d");
    const width = video.value.videoWidth / 2;
    const height = video.value.videoHeight / 2;

    // 在 canvas3 上绘制 video 标签上当前播放的内容
    context.drawImage(video.value, 0, 0, width, height);
  } else {
    // doing nothing
  }
};

onMounted(() => {
  const computeFrame = (video, canvas1, canvas2, width, height) => {
    // 画布 1 上下文
    const context1 = canvas1.value.getContext("2d");
    // 画布 2 上下文
    const context2 = canvas2.value.getContext("2d");

    // 在画布 1 上绘制视频
    context1.drawImage(video.value, 0, 0, width, height);

    const frame = context1.getImageData(0, 0, width, height);
    const l = frame.data.length / 4;
    for (let i = 0; i < l; i++) {
      const r = frame.data[i * 4 + 0];
      const g = frame.data[i * 4 + 1];
      const b = frame.data[i * 4 + 2];
      if (g > 100 && r > 100 && b < 43) {
        frame.data[i * 4 + 3] = 0;
      }
    }
    context2.putImageData(frame, 0, 0);
    return;
  };

  const timerCallback = (video, canvas1, canvas2, width, height) => {
    if (video.value.paused || video.value.ended) {
      return;
    }
    computeFrame(video, canvas1, canvas2, width, height);
    // timerCallback(video, canvas1, canvas2, width, height);

    // 这一步的意思，大概是让这个 computeFrame 别阻塞住别的任务，让它最后再执行
    // So using setTimeout(..., 0) will basically make your code run after the current event loop tick
    // 不会卡死 UI
    // 通常 setTimeout 函数会有一个最小的值限定，所以 0 并不是真的 0，而是 4ms 或 10ms
    // 其实本质是帧数：它的意思是间隔多少秒在 canvas 上绘制一次的意思
    setTimeout(function () {
      timerCallback(video, canvas1, canvas2, width, height);
    }, 100);
  };

  // 监听 video 播放事件
  video.value.addEventListener(
    "play",
    () => {
      const width = video.value.videoWidth / 2;
      const height = video.value.videoHeight / 2;
      timerCallback(video, canvas1, canvas2, width, height);
    },
    false
  );
});
</script>

<style lang="scss">
body {
  background: black;
  color: #cccccc;
}
#c2 {
  background-image: url("./assets/foo.png");
  background-repeat: no-repeat;
}
div {
  float: left;
  border: 1px solid #444444;
  padding: 10px;
  margin: 10px;
  background: #3b3b3b;
}
</style>
