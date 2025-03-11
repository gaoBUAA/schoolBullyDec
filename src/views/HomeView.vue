<template>
  <div class="dashboard">
    <h1 class="title">校园霸凌监控检测系统</h1>

    <div class="content">
      <!-- 左侧部分：视频上传 + 监控画面 -->
      <div class="left-section">
        <div class="section monitor-section">
          <LiveMonitor />
        </div>
        <div class="section chart-section">
          <ViolenceChart />
        </div>
      </div>

      <!-- 右侧部分：数据可视化分析 -->
      <div class="right-section"><div class="section upload-section">
          <VideoUpload @videoUploaded="handleVideoUploaded" />
        </div>
        
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import VideoUpload from '@/components/VideoUpload.vue';
import LiveMonitor from '@/components/LiveMonitor.vue';
import ViolenceChart from '@/components/ViolenceChart.vue';

const handleVideoUploaded = () => {
  console.log('视频上传成功！');
};
</script>

<style scoped>
.dashboard {
  background: #0a1f2f;
  color: #00ffff;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  font-family: 'Arial', sans-serif;
  padding: 20px;
  width: 100vw;
  overflow: hidden;
}

.title {
  color: #00ffff;
  text-shadow: 0 0 5px #00ffff;
  font-size: 3rem;
  margin-bottom: 20px;
}

.content {
  display: flex;
  justify-content: space-between;
  gap: 20px;
  width: 95vw; /* 调整整体宽度，让内容更宽 */
  height: 85vh; /* 让内容区更大 */
}

.left-section {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 40%; /* 左侧区域缩小 */
  height: 100%;
}

.right-section {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 60%; /* 右侧区域加大 */
  height: 100%;
}

.section {
  flex: 1;
  padding: 20px;
  border-radius: 15px;
  background: rgba(10, 25, 47, 0.9);
  box-shadow: 0 0 20px rgba(0, 255, 255, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.upload-section {
  width: 100%;
  height: 50vh; /* 增大上传组件的高度 */
  background: rgba(10, 25, 47, 0.8);
  border: 1px solid #00ffff;
  box-shadow: 0 0 10px rgba(0, 255, 255, 0.7);
}

.monitor-section {
  height: 50vh; /* 让监控画面更大 */
}

.chart-section {
  height: 60vh; /* 让数据可视化组件更大 */
}

@media (max-width: 768px) {
  .content {
    flex-direction: column;
    height: auto;
  }

  .left-section, .right-section {
    width: 100%;
  }

  .upload-section {
    height: auto;
    min-height: 300px;
  }
}

</style>
