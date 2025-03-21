<template>
  <div class="upload-container">
    <h2>📹 上传视频进行检测</h2>

    <div class="upload-preview">
      <div class="upload-box">
        <input type="file" @change="handleVideoUpload" accept="video/mp4" class="file-input">
        <label for="fileInput" class="upload-btn">选择视频</label>
      </div>

      <div v-if="videoPreviewUrl" class="video-preview">
        <video ref="videoPlayer" :src="videoPreviewUrl" controls autoplay @error="handleVideoError"></video>
      </div>
    </div>

    <!-- 结果显示模块 -->
    <div class="detection-info" v-if="detectionResults.length || uploadStatus !== '⏳ 视频上传中...'">
      <h3>📊 检测结果</h3>
      <div class="info-box">
        <div class="info-item">
          <span>检测结果:</span>
          <span>{{ uploadStatus || '等待推理...' }}</span>
        </div>
        <div class="info-item">
          <span>概率分布:</span>
          <span v-if="detectionResults.length"> {{ detectionResults[0].probability }}%</span>
          <span v-else>N/A</span>
        </div>
        <div class="info-item">
          <span>FPS:</span>
          <span>{{ fps }} frames/s</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watchEffect, onMounted } from 'vue';
import axios from 'axios';

const uploadStatus = ref('');
const uploadProgress = ref(0);
const videoPreviewUrl = ref('');
const videoPlayer = ref(null);
const detectionResults = ref([]);
const fps = ref(0);  // FPS统计

let lastFrameTime = 0;
let frameCount = 0;

const handleVideoUpload = (event) => {
  const file = event.target.files[0];
  if (!file) return;

  console.log("📁 选择文件:", file.name);
  uploadStatus.value = '⏳ 视频上传中...';
  uploadProgress.value = 0;

  setTimeout(() => {
    if (videoPreviewUrl.value) {
      console.log("🗑 释放旧 Blob URL:", videoPreviewUrl.value);
      URL.revokeObjectURL(videoPreviewUrl.value);
    }
  }, 10000);

  videoPreviewUrl.value = URL.createObjectURL(file);
  console.log("✅ 生成的视频预览 URL:", videoPreviewUrl.value);

  processVideoFrames(file);
};

watchEffect(() => {
  if (videoPlayer.value && videoPreviewUrl.value) {
    console.log("🎥 重新设置 video.src:", videoPreviewUrl.value);
    videoPlayer.value.src = videoPreviewUrl.value;
    videoPlayer.value.load();
  }
});

const handleVideoError = () => {
  console.error("❌ 视频加载失败:", videoPreviewUrl.value);
  alert("视频加载失败，请检查文件格式或尝试更换浏览器！");
};

const processVideoFrames = (file) => {
  const videoElement = document.createElement('video');
  videoElement.src = URL.createObjectURL(file);
  videoElement.crossOrigin = 'anonymous';
  videoElement.autoplay = false;
  videoElement.muted = true;

  const canvas = document.createElement('canvas');
  const context = canvas.getContext('2d');
  detectionResults.value = [];

  videoElement.addEventListener('loadeddata', async () => {
    const frameRate = 5;
    let currentTime = 0;
    const startTime = performance.now();  // 计时开始

    while (currentTime < videoElement.duration) {
      videoElement.currentTime = currentTime;
      await new Promise((resolve) => videoElement.onseeked = resolve);

      canvas.width = videoElement.videoWidth;
      canvas.height = videoElement.videoHeight;
      context.drawImage(videoElement, 0, 0, canvas.width, canvas.height);
      const frameData = canvas.toDataURL('image/jpeg').replace(/^data:image\/jpeg;base64,/, '');

      try {
        const response = await axios.post('http://localhost:5000/predict', { frame: frameData });
        detectionResults.value.push({
          label: response.data.label,
          probability: (response.data.probs[0] * 100).toFixed(2)
        });
      } catch (error) {
        console.error('帧推理失败:', error);
      }
      
      frameCount++;
      const currentTimeNow = performance.now();
      const elapsedTime = (currentTimeNow - startTime) / 1000;  // 转为秒
      fps.value = (frameCount / elapsedTime).toFixed(2);  // 更新FPS

      currentTime += 1 / frameRate;
    }
    uploadStatus.value = '📊 逐帧检测完成，结果已生成！';
  });
};
</script>

<style scoped>
.upload-container {
  width: 600px;
  margin: auto;
  padding: 20px;
  text-align: center;
  border-radius: 15px;
  background: rgba(10, 25, 47, 0.8);
  box-shadow: 0 0 15px rgba(0, 255, 255, 0.7);
  overflow: hidden;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

h2 {
  color: #00ffff;
  text-shadow: 0 0 5px #00ffff;
}

.upload-preview {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
  max-width: 100%;
}

.video-preview {
  display: flex;
  justify-content:left; /* 水平居中 */
  align-items: center;    /* 垂直居中 */
  flex: 1;
  max-width: 100%;
  max-height: 100%;      /* 根据需要调整这个值 */
}

.video-preview video {
  justify-content: left; /* 水平居中 */
  max-width: 100%;
  max-height: 400px;       /* 保证视频内容在最大高度内 */
  object-fit: contain;
}

.upload-box {
  width: 40%;
  text-align: center;
  position: relative;
}

.file-input {
  opacity: 0;
  position: absolute;
  width: 100%;
  height: 80%;
  cursor: pointer;
}

.upload-btn {
  display: inline-block;
  padding: 10px 20px;
  color: #00ffff;
  border: 1px solid #00ffff;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 8px;
  cursor: pointer;
  transition: 0.3s;
}

.upload-btn:hover {
  background: rgba(0, 255, 255, 0.2);
}

/* 新的检测信息模块 */
.detection-info {
  margin-top: 20px;
  padding: 15px;
  background-color: rgba(0, 0, 0, 0.7);
  border-radius: 10px;
  box-shadow: 0 0 15px rgba(0, 255, 255, 0.7);
}

.detection-info h3 {
  color: #00ffff;
  margin-bottom: 10px;
}

.info-box {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.info-item {
  display: flex;
  justify-content: space-between;
  color: #ffffff;
  font-size: 14px;
}

.info-item span:first-child {
  font-weight: bold;
}
</style>
