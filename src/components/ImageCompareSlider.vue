<template>
  <div :class="sliderClass" ref="container">
    <div class="compare-img compare-img--before">
      <img :src="before" alt="before" />
    </div>
    <div class="compare-img compare-img--after" :style="afterStyle">
      <img :src="after" alt="after" />
    </div>
    <div class="compare-handle" :style="handleStyle" @mousedown="startDrag" @touchstart="startDrag">
      <template v-if="showMagnifier">
        <el-tooltip content="放大预览" placement="right">
          <el-button class="magnifier-btn" type="primary" circle size="small" @click.stop="showDialog = true">
            <el-icon><ZoomIn /></el-icon>
          </el-button>
        </el-tooltip>
      </template>
    </div>
    <el-dialog v-model="showDialog" fullscreen :close-on-click-modal="true" title="对比放大预览">
      <div style="display: flex; justify-content: center; height: 100vh; align-items: center;">
        <ImageCompareSlider :before="before" :after="after" :large="true" :showMagnifier="false" />
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { ZoomIn } from '@element-plus/icons-vue'
import { ElButton, ElDialog, ElTooltip, ElIcon } from 'element-plus'
export default {
  name: 'ImageCompareSlider',
  components: { ElButton, ElDialog, ElTooltip, ElIcon, ZoomIn },
  props: {
    before: { type: String, required: true },
    after: { type: String, required: true },
    large: { type: Boolean, default: false },
    showMagnifier: { type: Boolean, default: true }
  },
  data() {
    return {
      dragging: false,
      sliderX: 0.5, // 百分比
      showDialog: false
    }
  },
  computed: {
    afterStyle() {
      return {
        width: `${this.sliderX * 100}%`
      }
    },
    handleStyle() {
      return {
        left: `calc(${this.sliderX * 100}% - 12px)`
      }
    },
    sliderClass() {
      return this.large ? 'compare-slider compare-slider--large' : 'compare-slider'
    }
  },
  mounted() {
    this.$refs.container.addEventListener('mousemove', this.onDrag)
    this.$refs.container.addEventListener('touchmove', this.onDrag)
    window.addEventListener('mouseup', this.stopDrag)
    window.addEventListener('touchend', this.stopDrag)
  },
  beforeUnmount() {
    this.$refs.container.removeEventListener('mousemove', this.onDrag)
    this.$refs.container.removeEventListener('touchmove', this.onDrag)
    window.removeEventListener('mouseup', this.stopDrag)
    window.removeEventListener('touchend', this.stopDrag)
  },
  methods: {
    startDrag(e) {
      this.dragging = true
      e.preventDefault()
    },
    stopDrag() {
      this.dragging = false
    },
    onDrag(e) {
      if (!this.dragging) return
      let clientX
      if (e.touches && e.touches.length) {
        clientX = e.touches[0].clientX
      } else {
        clientX = e.clientX
      }
      const rect = this.$refs.container.getBoundingClientRect()
      let x = (clientX - rect.left) / rect.width
      x = Math.max(0, Math.min(1, x))
      this.sliderX = x
    }
  }
}
</script>

<style scoped>
.compare-slider {
  position: relative;
  width: 100%;
  max-width: 500px;
  height: 280px;
  background: #eee;
  overflow: hidden;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.06);
  user-select: none;
}
.compare-slider--large {
  max-width: 90vw;
  height: 80vh;
}
.compare-img {
  position: absolute;
  top: 0; left: 0; bottom: 0;
  height: 100%;
  width: 100%;
  overflow: hidden;
}
.compare-img img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  display: block;
}
.compare-img--after {
  z-index: 2;
  pointer-events: none;
  border-right: 2px solid #409eff;
}
.compare-img--before {
  z-index: 1;
}
.compare-handle {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 24px;
  background: rgba(64,158,255,0.7);
  border-radius: 12px;
  z-index: 10;
  cursor: ew-resize;
  transition: background 0.2s;
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
  display: flex;
  align-items: center;
  justify-content: center;
}
.compare-handle:hover {
  background: #409eff;
}
.magnifier-btn {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 20;
  box-shadow: 0 2px 8px rgba(0,0,0,0.12);
}
</style> 