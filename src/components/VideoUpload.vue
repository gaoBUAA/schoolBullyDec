<template>
  <div class="upload-container">
    <h2>📹 上传视频进行检测</h2>

    <div class="upload-preview">
      <!-- 上传部分 -->
      <div class="upload-box">
        <input type="file" @change="handleVideoUpload" accept="video/mp4" class="file-input">
        <label for="fileInput" class="upload-btn">选择视频</label>
      </div>

      <!-- 预览部分 -->
      <div v-if="videoPreviewUrl" class="video-preview">
        <h3>🎬 预览</h3>
        <video ref="videoPlayer" :src="videoPreviewUrl" controls autoplay @error="handleVideoError"></video>
      </div>
    </div>

    <!-- 上传状态 -->
    <div v-if="uploadStatus" class="status-box">
      <p>{{ uploadStatus }}</p>
      <div v-if="uploadProgress > 0" class="progress-bar">
        <div class="progress" :style="{ width: uploadProgress + '%' }"></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watchEffect } from 'vue';

// 上传状态
const uploadStatus = ref('');
const uploadProgress = ref(0);
const videoPreviewUrl = ref('');
const videoPlayer = ref(null);

// 处理视频上传
const handleVideoUpload = (event) => {
  const file = event.target.files[0];
  if (!file) return;

  console.log("📁 选择文件:", file.name);
  uploadStatus.value = '⏳ 视频上传中...';
  uploadProgress.value = 0;

  // 确保释放旧 Blob URL
  setTimeout(() => {
    if (videoPreviewUrl.value) {
      console.log("🗑 释放旧 Blob URL:", videoPreviewUrl.value);
      URL.revokeObjectURL(videoPreviewUrl.value);
    }
  }, 10000); // 10s 后释放，避免影响播放

  videoPreviewUrl.value = URL.createObjectURL(file);
  console.log("✅ 生成的视频预览 URL:", videoPreviewUrl.value);

  mockUploadVideo(file);
};

// 监听 video URL 变化，手动触发加载
watchEffect(() => {
  if (videoPlayer.value && videoPreviewUrl.value) {
    console.log("🎥 重新设置 video.src:", videoPreviewUrl.value);
    videoPlayer.value.src = videoPreviewUrl.value;
    videoPlayer.value.load(); // 手动触发加载
  }
});

// 处理视频加载失败
const handleVideoError = () => {
  console.error("❌ 视频加载失败:", videoPreviewUrl.value);
  alert("视频加载失败，请检查文件格式或尝试更换浏览器！");
};

// 模拟视频上传
const mockUploadVideo = (file) => {
  const formData = new FormData();
  formData.append('video', file);

  let progress = 0;
  const interval = setInterval(() => {
    progress += 10;
    uploadProgress.value = progress;
    if (progress >= 100) {
      clearInterval(interval);
      uploadStatus.value = '✅ 视频上传成功！正在分析...';

      setTimeout(() => {
        uploadStatus.value = '📊 分析完成，结果已生成！';
      }, 2000);
    }
  }, 300);
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
}

h2 {
  color: #00ffff;
  text-shadow: 0 0 5px #00ffff;
}

/* 上传和预览水平排列 */
.upload-preview {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
  margin-top: 15px;
}

/* 上传按钮 */
.upload-box {
  width: 40%;
  text-align: center;
  position: relative;
}

.file-input {
  opacity: 0;
  position: absolute;
  width: 100%;
  height: 100%;
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

/* 预览窗口 */
.video-preview {
  width: 100%;
  text-align: center;
  height: 500px; /* 固定高度，调整根据需要 */
  overflow: hidden; /* 防止视频溢出 */
  position: relative;
}

.video-preview h3 {
  color: #00ffff;
}

/* 保持视频的比例并适应容器 */
video {
  width: 100%;
  height: 100%;
  object-fit: contain; /* 使视频适应容器，不会被拉伸 */
  border-radius: 10px;
  border: 1px solid #00ffff;
  box-shadow: 0 0 10px rgba(0, 255, 255, 0.5);
}


/* 上传状态 */
.status-box {
  margin-top: 15px;
  color: #00ffff;
}

.progress-bar {
  width: 100%;
  height: 10px;
  background: rgba(0, 255, 255, 0.2);
  border-radius: 5px;
  overflow: hidden;
  margin-top: 5px;
}

.progress {
  height: 100%;
  background: #00ffff;
  transition: width 0.3s ease-in-out;
}

/* 响应式：当屏幕较小时改为垂直布局 */
@media (max-width: 768px) {
  .upload-preview {
    flex-direction: column;
    gap: 15px;
  }

  .upload-box,
  .video-preview {
    width: 100%;
  }

  .upload-btn {
    width: 80%;
  }

  video {
    width: 100%;
  }
}
</style>
