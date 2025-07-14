<template>
  <el-container>
    <el-header class="header">
      <h1>图片处理工具</h1>
      <p>支持图片压缩、水印添加等功能</p>
    </el-header>
    <el-main>
      <el-row justify="center">
        <el-col :span="16">
          <ImageCompressor />
        </el-col>
      </el-row>
    </el-main>
  </el-container>
</template>

<script>
import ImageCompressor from './components/ImageCompressor.vue'

export default {
  name: 'App',
  components: {
    ImageCompressor
  }
}
</script>

<style>
body{
  margin: 0;
}
#app {
  font-family: 'Helvetica Neue', Helvetica, 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', '微软雅黑', Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin: 0;
  padding: 0;
}

.header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: rgb(8, 8, 8);
  text-align: center;
  /* padding: 20px; */
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}

.header h1 {
  margin: 0 0 10px 0;
  font-size: 2.5em;
  font-weight: 300;
}

.header p {
  margin: 0;
  font-size: 1.1em;
  opacity: 0.9;
}

.el-main {
  padding: 30px;
  background-color: #f5f7fa;
  min-height: calc(100vh - 120px);
}

.el-card {
  margin-bottom: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}
</style> 