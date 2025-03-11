<template>
    <div class="camera-container">
      <h2>实时监控</h2>
      <div class="camera-frame">
        <video ref="video" autoplay muted class="video-feed"></video>
        <div v-if="!isCameraActive" class="overlay">
          <p>摄像头未开启</p>
        </div>
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
  
  const video = ref(null);
  const stream = ref(null);
  const isCameraActive = ref(false);
  
  const startCamera = async () => {
    try {
      stream.value = await navigator.mediaDevices.getUserMedia({ video: true });
      video.value.srcObject = stream.value;
      isCameraActive.value = true;
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
  
  // 组件销毁前关闭摄像头
  onBeforeUnmount(() => {
    stopCamera();
  });
  </script>
  
  <style scoped>
  /* 主容器 */
  .camera-container {
    text-align: center;
    background: rgba(10, 25, 47, 0.8); /* 科幻风半透明背景 */
    padding: 20px;
    border-radius: 15px;
    box-shadow: 0 0 15px rgba(0, 255, 255, 0.7);
    margin: auto;
  }
  
  /* 监控画面框架 */
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
  
  /* 视频流样式 */
  .video-feed {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
  
  /* 摄像头未开启时的提示 */
  .overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.7);
    color: #00ffff;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 18px;
    font-weight: bold;
    text-shadow: 0 0 5px #00ffff;
  }
  
  /* 控制面板 */
  .control-panel {
    margin-top: 15px;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 15px;
  }
  
  /* 按钮样式 */
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
  
  /* 状态指示灯 */
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
  