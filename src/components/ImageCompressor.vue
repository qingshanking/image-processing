<template>
  <el-card class="compressor-card">
    <template #header>
      <div class="card-header">
        <el-icon><Picture /></el-icon>
        <span>图片压缩</span>
      </div>
    </template>
    
    <div class="upload-area">
      <el-upload
        ref="uploadRef"
        :auto-upload="false"
        :show-file-list="false"
        :on-change="handleFileChange"
        accept="image/*"
        drag
      >
        <el-icon class="el-icon--upload"><upload-filled /></el-icon>
        <div class="el-upload__text">
          将文件拖到此处，或<em>点击上传</em>
        </div>
        <template #tip>
          <div class="el-upload__tip">
            支持 jpg/png/gif/webp 格式图片
            <br>
            <span style="color: #409eff;">💡 提示：也可以直接粘贴图片（Ctrl+V）</span>
          </div>
        </template>
      </el-upload>
    </div>

    <div v-if="originalImage" class="image-preview">
      <h4>原始图片</h4>
      <div class="image-info">
        <img :src="originalImage" alt="原始图片" />
        <div class="info-text">
          <p>大小: {{ formatFileSize(originalFile.size) }}</p>
          <p>尺寸: {{ originalDimensions.width }} × {{ originalDimensions.height }}</p>
        </div>
      </div>
    </div>
    <!-- 并排对比预览：仅当原图和压缩图都存在时显示 -->
    <div v-if="originalImage && compressedImage" style="margin: 24px 0; display: flex; flex-direction: column; align-items: center;">
      <h4 style="color: #409eff;">对比预览</h4>
      <ImageSideBySide :before="originalImage" :after="compressedImage" />
    </div>

    <div v-if="originalImage" class="compression-settings">
      <h4>压缩设置</h4>
      <el-form :model="compressionSettings" label-width="100px">
        <el-form-item label="质量">
          <el-slider
            v-model="compressionSettings.quality"
            :min="0.1"
            :max="1"
            :step="0.1"
            show-input
            input-size="small"
          />
        </el-form-item>
        <el-form-item label="最大宽度">
          <el-input-number
            v-model="compressionSettings.maxWidth"
            :min="100"
            :max="4000"
            size="small"
          />
        </el-form-item>
        <el-form-item label="最大高度">
          <el-input-number
            v-model="compressionSettings.maxHeight"
            :min="100"
            :max="4000"
            size="small"
          />
        </el-form-item>
        
        <!-- 水印设置 -->
        <el-form-item label="添加水印">
          <el-switch v-model="compressionSettings.addWatermark" />
        </el-form-item>
        
        <div v-if="compressionSettings.addWatermark" class="watermark-settings">
          <el-tabs v-model="activeWatermarkTab" type="card">
            <el-tab-pane label="文字水印" name="text">
              <el-form :model="textWatermark" label-width="80px">
                <el-form-item label="水印文字">
                  <el-input v-model="textWatermark.text" placeholder="请输入水印文字" size="small" />
                </el-form-item>
                <el-form-item label="字体大小">
                  <el-slider
                    v-model="textWatermark.fontSize"
                    :min="12"
                    :max="48"
                    :step="2"
                    show-input
                    input-size="small"
                  />
                </el-form-item>
                <el-form-item label="字体颜色">
                  <el-color-picker v-model="textWatermark.color" size="small" />
                </el-form-item>
                <el-form-item label="透明度">
                  <el-slider
                    v-model="textWatermark.opacity"
                    :min="0.1"
                    :max="1"
                    :step="0.1"
                    show-input
                    input-size="small"
                  />
                </el-form-item>
                <el-form-item label="旋转角度">
                  <el-slider
                    v-model="textWatermark.rotation"
                    :min="-180"
                    :max="180"
                    :step="5"
                    show-input
                    input-size="small"
                  />
                </el-form-item>
                <el-form-item label="位置">
                  <el-select v-model="textWatermark.position" placeholder="选择位置" size="small">
                    <el-option label="左上角" value="top-left" />
                    <el-option label="右上角" value="top-right" />
                    <el-option label="左下角" value="bottom-left" />
                    <el-option label="右下角" value="bottom-right" />
                    <el-option label="居中" value="center" />
                  </el-select>
                </el-form-item>
              </el-form>
            </el-tab-pane>
            
            <el-tab-pane label="图标水印" name="icon">
              <el-form :model="iconWatermark" label-width="80px">
                <el-form-item label="选择图标">
                  <el-select v-model="iconWatermark.icon" placeholder="选择图标" size="small" style="width: 100%">
                    <el-option
                      v-for="icon in availableIcons"
                      :key="icon.name"
                      :label="icon.label"
                      :value="icon.name"
                    >
                      <div style="display: flex; align-items: center; gap: 8px;">
                        <el-icon><component :is="icon.name" /></el-icon>
                        <span>{{ icon.label }}</span>
                      </div>
                    </el-option>
                  </el-select>
                </el-form-item>
                <el-form-item label="图标大小">
                  <el-slider
                    v-model="iconWatermark.size"
                    :min="20"
                    :max="100"
                    :step="10"
                    show-input
                    input-size="small"
                  />
                </el-form-item>
                <el-form-item label="图标颜色">
                  <el-color-picker v-model="iconWatermark.color" size="small" />
                </el-form-item>
                <el-form-item label="透明度">
                  <el-slider
                    v-model="iconWatermark.opacity"
                    :min="0.1"
                    :max="1"
                    :step="0.1"
                    show-input
                    input-size="small"
                  />
                </el-form-item>
                <el-form-item label="旋转角度">
                  <el-slider
                    v-model="iconWatermark.rotation"
                    :min="-180"
                    :max="180"
                    :step="5"
                    show-input
                    input-size="small"
                  />
                </el-form-item>
                <el-form-item label="位置">
                  <el-select v-model="iconWatermark.position" placeholder="选择位置" size="small">
                    <el-option label="左上角" value="top-left" />
                    <el-option label="右上角" value="top-right" />
                    <el-option label="左下角" value="bottom-left" />
                    <el-option label="右下角" value="bottom-right" />
                    <el-option label="居中" value="center" />
                  </el-select>
                </el-form-item>
              </el-form>
            </el-tab-pane>
            
            <el-tab-pane label="图片水印" name="image">
              <el-form :model="imageWatermark" label-width="80px">
                <el-form-item label="水印图片">
                  <el-upload
                    ref="watermarkUploadRef"
                    :auto-upload="false"
                    :show-file-list="false"
                    :on-change="handleWatermarkImageChange"
                    accept="image/*"
                    class="watermark-upload"
                  >
                    <el-button type="primary" size="small">
                      <el-icon><Upload /></el-icon>
                      选择水印图片
                    </el-button>
                    <template #tip>
                      <div class="el-upload__tip">
                        支持 jpg/png/gif/webp 格式，建议使用透明背景的PNG图片
                      </div>
                    </template>
                  </el-upload>
                </el-form-item>
                
                <div v-if="imageWatermark.image" class="watermark-preview">
                  <h5>水印预览</h5>
                  <img :src="imageWatermark.image" alt="水印图片" style="max-width: 100px; max-height: 60px; border: 1px solid #dcdfe6; border-radius: 4px;" />
                </div>
                
                <el-form-item label="水印大小">
                  <el-slider
                    v-model="imageWatermark.size"
                    :min="20"
                    :max="200"
                    :step="10"
                    show-input
                    input-size="small"
                  />
                </el-form-item>
                <el-form-item label="透明度">
                  <el-slider
                    v-model="imageWatermark.opacity"
                    :min="0.1"
                    :max="1"
                    :step="0.1"
                    show-input
                    input-size="small"
                  />
                </el-form-item>
                <el-form-item label="旋转角度">
                  <el-slider
                    v-model="imageWatermark.rotation"
                    :min="-180"
                    :max="180"
                    :step="5"
                    show-input
                    input-size="small"
                  />
                </el-form-item>
                <el-form-item label="位置">
                  <el-select v-model="imageWatermark.position" placeholder="选择位置" size="small">
                    <el-option label="左上角" value="top-left" />
                    <el-option label="右上角" value="top-right" />
                    <el-option label="左下角" value="bottom-left" />
                    <el-option label="右下角" value="bottom-right" />
                    <el-option label="居中" value="center" />
                  </el-select>
                </el-form-item>
              </el-form>
            </el-tab-pane>
            
            <el-tab-pane label="全屏水印" name="fullscreen">
              <el-form :model="fullscreenWatermark" label-width="80px">
                <el-form-item label="水印文字">
                  <el-input v-model="fullscreenWatermark.text" placeholder="例如：此照片仅做**使用" size="small" />
                </el-form-item>
                <el-form-item label="字体大小">
                  <el-slider
                    v-model="fullscreenWatermark.fontSize"
                    :min="16"
                    :max="60"
                    :step="2"
                    show-input
                    input-size="small"
                  />
                </el-form-item>
                <el-form-item label="字体颜色">
                  <el-color-picker v-model="fullscreenWatermark.color" size="small" />
                </el-form-item>
                <el-form-item label="透明度">
                  <el-slider
                    v-model="fullscreenWatermark.opacity"
                    :min="0.05"
                    :max="0.5"
                    :step="0.05"
                    show-input
                    input-size="small"
                  />
                </el-form-item>
                <el-form-item label="旋转角度">
                  <el-slider
                    v-model="fullscreenWatermark.rotation"
                    :min="-45"
                    :max="45"
                    :step="5"
                    show-input
                    input-size="small"
                  />
                </el-form-item>
                <el-form-item label="间距">
                  <el-slider
                    v-model="fullscreenWatermark.spacing"
                    :min="50"
                    :max="200"
                    :step="10"
                    show-input
                    input-size="small"
                  />
                </el-form-item>
              </el-form>
            </el-tab-pane>
          </el-tabs>
        </div>
        
        <el-form-item>
          <el-button type="primary" @click="compressImage" :loading="compressing">
            <el-icon><Download /></el-icon>
            压缩并添加水印
          </el-button>
        </el-form-item>
      </el-form>
    </div>

    <div v-if="compressedImage" class="compressed-result">
      <h4>压缩结果</h4>
      <div class="image-info">
        <img :src="compressedImage" alt="压缩后图片" />
        <div class="info-text">
          <p>大小: {{ formatFileSize(compressedFile.size) }}</p>
          <p>压缩率: {{ compressionRatio }}%</p>
          <p>尺寸: {{ compressedDimensions.width }} × {{ compressedDimensions.height }}</p>
        </div>
      </div>
      <el-button type="success" @click="downloadCompressed">
        <el-icon><Download /></el-icon>
        下载压缩图片
      </el-button>
    </div>
  </el-card>
</template>

<script>
import imageCompression from 'browser-image-compression'
import { saveAs } from 'file-saver'
import ImageCompareSlider from './ImageCompareSlider.vue'
import ImageSideBySide from './ImageSideBySide.vue'

export default {
  name: 'ImageCompressor',
  components: {
    ImageCompareSlider,
    ImageSideBySide
  },
  data() {
    return {
      originalFile: null,
      originalImage: null,
      originalDimensions: { width: 0, height: 0 },
      compressedFile: null,
      compressedImage: null,
      compressedDimensions: { width: 0, height: 0 },
      compressing: false,
      compressionSettings: {
        quality: 0.8,
        maxWidth: 1920,
        maxHeight: 1080,
        addWatermark: false
      },
      activeWatermarkTab: 'text',
      textWatermark: {
        text: '水印文字',
        fontSize: 20,
        color: '#ff0000',
        opacity: 0.7,
        rotation: -45,
        position: 'bottom-right'
      },
      iconWatermark: {
        icon: 'Star',
        size: 50,
        color: '#ff0000',
        opacity: 0.7,
        rotation: 0,
        position: 'top-right'
      },
      imageWatermark: {
        image: null,
        imageFile: null,
        size: 80,
        opacity: 0.7,
        rotation: 0,
        position: 'bottom-right'
      },
      fullscreenWatermark: {
        text: '',
        fontSize: 20,
        color: '#000000',
        opacity: 0.5,
        rotation: 0,
        spacing: 100
      },
      availableIcons: [
        { name: 'Star', label: '星星' },
        { name: 'Heart', label: '爱心' },
        { name: 'Check', label: '对勾' },
        { name: 'Warning', label: '警告' },
        { name: 'InfoFilled', label: '信息' },
        { name: 'CircleCheck', label: '圆形对勾' },
        { name: 'CircleClose', label: '圆形关闭' },
        { name: 'CirclePlus', label: '圆形加号' },
        { name: 'Location', label: '位置' },
        { name: 'Phone', label: '电话' },
        { name: 'Message', label: '消息' },
        { name: 'User', label: '用户' },
        { name: 'Setting', label: '设置' },
        { name: 'HomeFilled', label: '首页' },
        { name: 'Calendar', label: '日历' }
      ]
    }
  },
  computed: {
    compressionRatio() {
      if (!this.originalFile || !this.compressedFile) return 0
      const ratio = ((this.originalFile.size - this.compressedFile.size) / this.originalFile.size * 100)
      return Math.round(ratio)
    }
  },
  mounted() {
    // 添加粘贴事件监听
    document.addEventListener('paste', this.handlePaste)
  },
  beforeUnmount() {
    // 移除粘贴事件监听
    document.removeEventListener('paste', this.handlePaste)
  },
  methods: {
    handlePaste(e) {
      const items = e.clipboardData?.items
      if (!items) return
      
      for (let i = 0; i < items.length; i++) {
        const item = items[i]
        if (item.type.indexOf('image') !== -1) {
          const file = item.getAsFile()
          if (file) {
            this.handleFileChange({ raw: file })
            this.$message.success('已从剪贴板粘贴图片')
            break
          }
        }
      }
    },
    handleFileChange(file) {
      this.originalFile = file.raw
      this.originalImage = URL.createObjectURL(file.raw)
      this.getImageDimensions(file.raw).then(dimensions => {
        this.originalDimensions = dimensions
      })
      this.compressedFile = null
      this.compressedImage = null
    },
    
    handleWatermarkImageChange(file) {
      this.imageWatermark.imageFile = file.raw
      this.imageWatermark.image = URL.createObjectURL(file.raw)
    },
    
    getImageDimensions(file) {
      return new Promise((resolve) => {
        const img = new Image()
        img.onload = () => {
          resolve({ width: img.width, height: img.height })
        }
        img.src = URL.createObjectURL(file)
      })
    },
    
    async compressImage() {
      if (!this.originalFile) return
      
      this.compressing = true
      try {
        const options = {
          maxSizeMB: 1,
          maxWidthOrHeight: Math.max(this.compressionSettings.maxWidth, this.compressionSettings.maxHeight),
          useWebWorker: true,
          quality: this.compressionSettings.quality
        }
        
        const compressedFile = await imageCompression(this.originalFile, options)
        
        // 如果需要添加水印
        if (this.compressionSettings.addWatermark) {
          const watermarkedBlob = await this.addWatermarkToImage(compressedFile)
          this.compressedFile = new File([watermarkedBlob], compressedFile.name, { type: 'image/jpeg' })
          this.compressedImage = URL.createObjectURL(watermarkedBlob)
        } else {
          this.compressedFile = compressedFile
          this.compressedImage = URL.createObjectURL(compressedFile)
        }
        
        this.getImageDimensions(this.compressedFile).then(dimensions => {
          this.compressedDimensions = dimensions
        })
        
        this.$message.success('图片压缩完成！')
      } catch (error) {
        console.error('压缩失败:', error)
        this.$message.error('图片压缩失败，请重试')
      } finally {
        this.compressing = false
      }
    },
    
    downloadCompressed() {
      if (!this.compressedFile) return
      
      const fileName = this.originalFile.name.replace(/\.[^/.]+$/, '') + '_compressed.jpg'
      saveAs(this.compressedFile, fileName)
    },
    
    async addWatermarkToImage(file) {
      return new Promise((resolve, reject) => {
        const canvas = document.createElement('canvas')
        const ctx = canvas.getContext('2d')
        const img = new Image()
        
        img.onload = async () => {
          canvas.width = img.width
          canvas.height = img.height
          
          // 绘制原始图片
          ctx.drawImage(img, 0, 0)
          
          // 根据当前选中的标签页添加水印
          if (this.activeWatermarkTab === 'text') {
            this.addTextWatermark(ctx, canvas.width, canvas.height)
          } else if (this.activeWatermarkTab === 'icon') {
            this.addIconWatermark(ctx, canvas.width, canvas.height)
          } else if (this.activeWatermarkTab === 'image') {
            await this.addImageWatermark(ctx, canvas.width, canvas.height)
          } else if (this.activeWatermarkTab === 'fullscreen') {
            await this.addFullscreenWatermark(ctx, canvas.width, canvas.height)
          }
          
          // 转换为blob
          canvas.toBlob((blob) => {
            resolve(blob)
          }, 'image/jpeg', 0.9)
        }
        
        img.onerror = reject
        img.src = URL.createObjectURL(file)
      })
    },
    
    addTextWatermark(ctx, width, height) {
      const { text, fontSize, color, opacity, rotation, position } = this.textWatermark
      
      ctx.save()
      ctx.globalAlpha = opacity
      ctx.font = `${fontSize}px Arial, sans-serif`
      ctx.fillStyle = color
      ctx.textAlign = 'center'
      ctx.textBaseline = 'middle'
      
      // 计算位置
      const positionCoords = this.getPositionCoords(position, width, height, fontSize, text)
      
      // 旋转
      ctx.translate(positionCoords.x, positionCoords.y)
      ctx.rotate((rotation * Math.PI) / 180)
      
      // 绘制文字
      ctx.fillText(text, 0, 0)
      ctx.restore()
    },
    
    addIconWatermark(ctx, width, height) {
      const { icon, size, color, opacity, rotation, position } = this.iconWatermark
      
      ctx.save()
      ctx.globalAlpha = opacity
      ctx.fillStyle = color
      ctx.strokeStyle = color
      ctx.lineWidth = 2
      
      // 计算位置
      const positionCoords = this.getPositionCoords(position, width, height, size, '')
      
      // 旋转
      ctx.translate(positionCoords.x, positionCoords.y)
      ctx.rotate((rotation * Math.PI) / 180)
      
      // 根据图标名称绘制不同的图形
      this.drawIcon(ctx, icon, size)
      
      ctx.restore()
    },
    
    drawIcon(ctx, iconName, size) {
      const halfSize = size / 2
      
      switch (iconName) {
        case 'Star':
          this.drawStar(ctx, 0, 0, halfSize, 5)
          break
        case 'Heart':
          this.drawHeart(ctx, 0, 0, halfSize)
          break
        case 'Check':
          ctx.beginPath()
          ctx.moveTo(-halfSize, 0)
          ctx.lineTo(-halfSize/3, halfSize/2)
          ctx.lineTo(halfSize, -halfSize/2)
          ctx.stroke()
          break
        case 'CircleCheck':
          ctx.beginPath()
          ctx.arc(0, 0, halfSize, 0, 2 * Math.PI)
          ctx.stroke()
          ctx.beginPath()
          ctx.moveTo(-halfSize/3, 0)
          ctx.lineTo(0, halfSize/3)
          ctx.lineTo(halfSize/2, -halfSize/3)
          ctx.stroke()
          break
        case 'CircleClose':
          ctx.beginPath()
          ctx.arc(0, 0, halfSize, 0, 2 * Math.PI)
          ctx.stroke()
          ctx.beginPath()
          ctx.moveTo(-halfSize/2, -halfSize/2)
          ctx.lineTo(halfSize/2, halfSize/2)
          ctx.moveTo(halfSize/2, -halfSize/2)
          ctx.lineTo(-halfSize/2, halfSize/2)
          ctx.stroke()
          break
        default:
          // 默认绘制圆形
          ctx.beginPath()
          ctx.arc(0, 0, halfSize, 0, 2 * Math.PI)
          ctx.fill()
      }
    },
    
    drawStar(ctx, cx, cy, radius, spikes) {
      let rot = Math.PI / 2 * 3
      let x = cx
      let y = cy
      const step = Math.PI / spikes
      
      ctx.beginPath()
      ctx.moveTo(cx, cy - radius)
      
      for (let i = 0; i < spikes; i++) {
        x = cx + Math.cos(rot) * radius
        y = cy + Math.sin(rot) * radius
        ctx.lineTo(x, y)
        rot += step
        
        x = cx + Math.cos(rot) * (radius * 0.5)
        y = cy + Math.sin(rot) * (radius * 0.5)
        ctx.lineTo(x, y)
        rot += step
      }
      
      ctx.lineTo(cx, cy - radius)
      ctx.closePath()
      ctx.fill()
    },
    
    drawHeart(ctx, x, y, size) {
      ctx.beginPath()
      ctx.moveTo(x, y + size * 0.3)
      ctx.bezierCurveTo(x, y, x - size, y, x - size, y + size * 0.3)
      ctx.bezierCurveTo(x - size, y + size * 0.6, x, y + size, x, y + size)
      ctx.bezierCurveTo(x, y + size, x + size, y + size * 0.6, x + size, y + size * 0.3)
      ctx.bezierCurveTo(x + size, y, x, y, x, y + size * 0.3)
      ctx.fill()
    },
    
    getPositionCoords(position, width, height, size, text) {
      const padding = 20
      let x, y
      
      // 如果是文字水印，需要计算文字的实际宽度
      let textWidth = 0
      if (text) {
        // 创建临时canvas来测量文字宽度
        const tempCanvas = document.createElement('canvas')
        const tempCtx = tempCanvas.getContext('2d')
        tempCtx.font = `${this.textWatermark.fontSize}px Arial, sans-serif`
        textWidth = tempCtx.measureText(text).width
      }
      
      switch (position) {
        case 'top-left':
          x = padding + Math.max(size/2, textWidth/2)
          y = padding + size/2
          break
        case 'top-right':
          x = width - padding - Math.max(size/2, textWidth/2)
          y = padding + size/2
          break
        case 'bottom-left':
          x = padding + Math.max(size/2, textWidth/2)
          y = height - padding - size/2
          break
        case 'bottom-right':
          x = width - padding - Math.max(size/2, textWidth/2)
          y = height - padding - size/2
          break
        case 'center':
        default:
          x = width / 2
          y = height / 2
          break
      }
      
      return { x, y }
    },
    
    async addImageWatermark(ctx, width, height) {
      if (!this.imageWatermark.imageFile) return
      
      return new Promise((resolve, reject) => {
        const watermarkImg = new Image()
        
        watermarkImg.onload = () => {
          const { size, opacity, rotation, position } = this.imageWatermark
          
          ctx.save()
          ctx.globalAlpha = opacity
          
          // 计算位置
          const positionCoords = this.getPositionCoords(position, width, height, size, '')
          
          // 计算缩放比例，保持宽高比
          const scale = size / Math.max(watermarkImg.width, watermarkImg.height)
          const scaledWidth = watermarkImg.width * scale
          const scaledHeight = watermarkImg.height * scale
          
          // 旋转和绘制
          ctx.translate(positionCoords.x, positionCoords.y)
          ctx.rotate((rotation * Math.PI) / 180)
          ctx.drawImage(
            watermarkImg,
            -scaledWidth / 2,
            -scaledHeight / 2,
            scaledWidth,
            scaledHeight
          )
          
          ctx.restore()
          resolve()
        }
        
        watermarkImg.onerror = reject
        watermarkImg.src = URL.createObjectURL(this.imageWatermark.imageFile)
      })
    },

    async addFullscreenWatermark(ctx, width, height) {
      const { text, fontSize, color, opacity, rotation, spacing } = this.fullscreenWatermark
      
      if (!text.trim()) return
      
      ctx.save()
      ctx.globalAlpha = opacity
      ctx.font = `${fontSize}px Arial, sans-serif`
      ctx.fillStyle = color
      ctx.textAlign = 'center'
      ctx.textBaseline = 'middle'
      
      // 计算文字的实际宽度
      const textWidth = ctx.measureText(text).width
      const textHeight = fontSize * 1.2
      
      // 计算水印间距
      const horizontalSpacing = Math.max(textWidth + spacing, 100)
      const verticalSpacing = Math.max(textHeight + spacing, 80)
      
      // 计算需要绘制的行数和列数
      const cols = Math.ceil(width / horizontalSpacing) + 2
      const rows = Math.ceil(height / verticalSpacing) + 2
      
      // 计算起始位置，确保水印覆盖整个图片
      const startX = -horizontalSpacing
      const startY = -verticalSpacing
      
      // 绘制全屏水印网格
      for (let row = 0; row < rows; row++) {
        for (let col = 0; col < cols; col++) {
          const x = startX + col * horizontalSpacing
          const y = startY + row * verticalSpacing
          
          ctx.save()
          ctx.translate(x, y)
          ctx.rotate((rotation * Math.PI) / 180)
          ctx.fillText(text, 0, 0)
          ctx.restore()
        }
      }
      
      ctx.restore()
    },
    
    formatFileSize(bytes) {
      if (bytes === 0) return '0 Bytes'
      const k = 1024
      const sizes = ['Bytes', 'KB', 'MB', 'GB']
      const i = Math.floor(Math.log(bytes) / Math.log(k))
      return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
    }
  }
}
</script>

<style scoped>
.compressor-card {
  height: 100%;
}

.card-header {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 16px;
  font-weight: 600;
}

.upload-area {
  margin-bottom: 20px;
}

.image-preview, .compressed-result {
  margin: 20px 0;
  padding: 15px;
  background-color: #f8f9fa;
  border-radius: 6px;
}

.image-preview h4, .compressed-result h4 {
  margin: 0 0 15px 0;
  color: #409eff;
}

.image-info {
  display: flex;
  gap: 15px;
  align-items: flex-start;
}

.image-info img {
  max-width: 200px;
  max-height: 150px;
  border-radius: 4px;
  border: 1px solid #dcdfe6;
}

.info-text p {
  margin: 5px 0;
  font-size: 14px;
  color: #606266;
}

.compression-settings {
  margin: 20px 0;
  padding: 15px;
  background-color: #f8f9fa;
  border-radius: 6px;
}

.compression-settings h4 {
  margin: 0 0 15px 0;
  color: #409eff;
}

.compressed-result .el-button {
  margin-top: 15px;
}

.watermark-settings {
  margin-top: 15px;
  padding: 15px;
  background-color: #f0f9ff;
  border-radius: 6px;
  border: 1px solid #e0f2fe;
}

.watermark-settings .el-tabs {
  margin-top: 10px;
}

.watermark-settings .el-form-item {
  margin-bottom: 12px;
}

.watermark-upload {
  margin-bottom: 10px;
}

.watermark-preview {
  margin: 15px 0;
  padding: 10px;
  background-color: #f8f9fa;
  border-radius: 4px;
  text-align: center;
}

.watermark-preview h5 {
  margin: 0 0 10px 0;
  color: #606266;
  font-size: 14px;
}
</style> 