<template>
    <div class="chart-container">
      <h2>暴力倾向指数分析</h2>
      <div ref="chartContainer" class="echart-box"></div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted, onUnmounted } from 'vue';
  import * as echarts from 'echarts';
  
  const chartContainer = ref(null);
  let chartInstance = null;
  
  // 生成模拟数据
  const generateMockData = () => {
    return Array.from({ length: 20 }, (_, i) => ({
      time: `T-${20 - i}`,
      violenceIndex: Math.floor(Math.random() * 50) + 30, // 模拟30~80的波动
    }));
  };
  
  // 初始化图表
  const initChart = () => {
    if (chartContainer.value) {
      chartInstance = echarts.init(chartContainer.value);
  
      const option = {
        backgroundColor: 'rgba(10, 25, 47, 0.8)',
        title: {
          text: '实时暴力倾向指数',
          textStyle: { color: '#00ffff' },
          left: 'center'
        },
        tooltip: {
          trigger: 'axis',
          backgroundColor: '#001a33',
          borderColor: '#00ffff',
          textStyle: { color: '#00ffff' }
        },
        xAxis: {
          type: 'category',
          data: generateMockData().map((d) => d.time),
          axisLine: { lineStyle: { color: '#00ffff' } },
        },
        yAxis: {
          type: 'value',
          min: 0,
          max: 100,
          axisLine: { lineStyle: { color: '#00ffff' } },
          splitLine: { lineStyle: { color: 'rgba(0, 255, 255, 0.2)' } },
        },
        series: [
          {
            data: generateMockData().map((d) => d.violenceIndex),
            type: 'line',
            smooth: true,
            lineStyle: { color: '#00ffff', width: 3 },
            areaStyle: { color: 'rgba(0, 255, 255, 0.2)' },
            symbol: 'none',
          },
        ],
      };
  
      chartInstance.setOption(option);
    }
  };
  
  // 更新数据
  const updateChart = () => {
    if (chartInstance) {
      let newData = generateMockData();
      chartInstance.setOption({
        xAxis: { data: newData.map((d) => d.time) },
        series: [{ data: newData.map((d) => d.violenceIndex) }],
      });
    }
  };
  
  onMounted(() => {
    initChart();
    setInterval(updateChart, 2000); // 每 2 秒更新数据
  });
  
  onUnmounted(() => {
    if (chartInstance) chartInstance.dispose();
  });
  </script>
  
  <style scoped>
  .chart-container {
    width: 600px;
    margin: auto;
    text-align: center;
    padding: 20px;
    border-radius: 15px;
    box-shadow: 0 0 15px rgba(0, 255, 255, 0.7);
  }
  
  h2 {
    color: #00ffff;
    text-shadow: 0 0 5px #00ffff;
  }
  
  .echart-box {
    width: 100%;
    height: 300px;
  }
  </style>
  