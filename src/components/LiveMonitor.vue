<template>
  <div class="camera-container">
    <h2>实时监控</h2>
    <div class="camera-frame">
      <video ref="video" autoplay muted class="video-feed"></video>
      <div v-if="!isCameraActive" class="overlay">
        <p>摄像头未开启</p>
      </div>
    </div>
    <div class="detection-info">
      <p><strong>检测结果:</strong> {{ label }}</p>
      <p><strong>概率分布:</strong> {{ probs }}</p>
      <p><strong>FPS:</strong> {{ fps }}</p>
    </div>
    <div class="control-panel">
      <button @click="startCamera" v-if="!isCameraActive">开启摄像头</button>
      <button @click="stopCamera" v-if="isCameraActive">关闭摄像头</button>
      <div class="status-indicator" :class="{ active: isCameraActive }"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onBeforeUnmount } from 'vue';
import axios from 'axios';

const video = ref(null);
const stream = ref(null);
const isCameraActive = ref(false);
const label = ref('等待推理...');
const probs = ref('N/A');
const fps = ref(0);
let intervalId = null;

const startCamera = async () => {
  try {
    stream.value = await navigator.mediaDevices.getUserMedia({ video: true });
    video.value.srcObject = stream.value;
    isCameraActive.value = true;
    startRealTimeInference();
  } catch (err) {
    console.error('无法访问摄像头:', err);
  }
};

const stopCamera = () => {
  if (stream.value) {
    stream.value.getTracks().forEach(track => track.stop());
    stream.value = null;
    isCameraActive.value = false;
  }
};

const startRealTimeInference = () => {
  const canvas = document.createElement('canvas');
  const context = canvas.getContext('2d');

  intervalId = setInterval(async () => {
    if (video.value && video.value.readyState === 4) {
      canvas.width = video.value.videoWidth;
      canvas.height = video.value.videoHeight;
      context.drawImage(video.value, 0, 0, canvas.width, canvas.height);
      const frameData = canvas.toDataURL('image/jpeg').replace(/^data:image\/jpeg;base64,/, '');
      
      try {
        const response = await axios.post('http://localhost:5000/predict', { frame: frameData });
        label.value = response.data.label;
        probs.value = response.data.probs.join(', ');
        fps.value = response.data.fps;
      } catch (error) {
        console.error('推理请求失败:', error);
      }
    }
  }, 100);
};

onBeforeUnmount(() => {
  if (intervalId) clearInterval(intervalId);
  stopCamera();
});
</script>

<style scoped>
.camera-container {
  text-align: center;
  background: rgba(10, 25, 47, 0.8);
  padding: 20px;
  border-radius: 15px;
  box-shadow: 0 0 15px rgba(0, 255, 255, 0.7);
  margin: auto;
}

.camera-frame {
  position: relative;
  width: 360px;
  height: 240px;
  border: 2px solid #00ffff;
  border-radius: 10px;
  overflow: hidden;
  background: black;
  box-shadow: inset 0 0 10px rgba(0, 255, 255, 0.5);
}

.video-feed {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.detection-info {
  margin-top: 10px;
  padding: 10px;
  background: rgba(0, 0, 0, 0.7);
  color: #00ffff;
  font-size: 14px;
  text-shadow: 0 0 3px #00ffff;
  border-radius: 8px;
  box-shadow: 0 0 5px rgba(0, 255, 255, 0.5);
}

.control-panel {
  margin-top: 15px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 15px;
}

button {
  background: linear-gradient(90deg, #003366, #00ffff);
  color: white;
  border: none;
  padding: 8px 15px;
  border-radius: 5px;
  font-size: 14px;
  cursor: pointer;
  transition: 0.3s;
  text-shadow: 0 0 5px #00ffff;
}

button:hover {
  background: linear-gradient(90deg, #00ffff, #003366);
  box-shadow: 0 0 8px rgba(0, 255, 255, 0.7);
}

.status-indicator {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: red;
  box-shadow: 0 0 10px rgba(255, 0, 0, 0.8);
  transition: 0.3s;
}

.status-indicator.active {
  background: #00ff00;
  box-shadow: 0 0 10px rgba(0, 255, 0, 0.8);
}
</style>
