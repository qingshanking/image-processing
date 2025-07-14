<template>
  <div class="side-by-side-container">
    <div class="side-image">
      <div class="side-title">原图</div>
      <el-image
        :src="before"
        :preview-src-list="[before]"
        fit="contain"
        style="max-width: 260px; max-height: 180px; border-radius: 4px; border: 1px solid #dcdfe6; background: #fff;"
      />
    </div>
    <div class="side-image">
      <div class="side-title">处理后</div>
      <el-image
        :src="after"
        :preview-src-list="[after]"
        fit="contain"
        style="max-width: 260px; max-height: 180px; border-radius: 4px; border: 1px solid #dcdfe6; background: #fff;"
      />
    </div>
  </div>
</template>

<script>
export default {
  name: 'ImageSideBySide',
  props: {
    before: { type: String, required: true },
    after: { type: String, required: true }
  }
}
</script>

<style scoped>
.side-by-side-container {
  display: flex;
  gap: 32px;
  justify-content: center;
  align-items: flex-start;
  margin: 24px 0;
}
.side-image {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #f8f9fa;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.06);
  padding: 16px 12px;
  min-width: 220px;
}
.side-title {
  font-size: 15px;
  color: #409eff;
  margin-bottom: 10px;
  font-weight: 500;
}
</style> 