<template>
  <el-dialog
    class="nodeVideoDialog"
    :title="$t('nodeVideo.title') || '设置视频'"
    :visible.sync="dialogVisible"
    :width="isMobile ? '90%' : '50%'"
    :top="isMobile ? '20px' : '15vh'"
    :modal-append-to-body="true"
    :append-to-body="true"
    :close-on-click-modal="false"
    :before-close="handleClose"
  >
    <div class="title">方式一</div>
    <VideoUpload
      ref="VideoUpload"
      v-model="video"
      style="margin-bottom: 12px;"
    ></VideoUpload>
    <div class="title">方式二</div>
    <div class="inputBox">
      <span class="label">请输入视频地址</span>
      <el-input
        v-model="videoUrl"
        size="mini"
        placeholder="http://xxx.com/xx.mp4"
        @keydown.native.stop
      ></el-input>
    </div>
    <div class="title">可选</div>
    <div class="inputBox">
      <span class="label">视频标题</span>
      <el-input v-model="videoTitle" size="mini" @keydown.native.stop></el-input>
    </div>
    <div class="inputBox">
      <span class="label">视频封面图URL</span>
      <el-input v-model="videoPoster" size="mini" @keydown.native.stop></el-input>
    </div>
    <div class="inputBox">
      <span class="label">视频位置</span>
      <el-select v-model="videoPlacement" size="mini">
        <el-option label="上方" value="top"></el-option>
        <el-option label="下方" value="bottom"></el-option>
        <el-option label="左侧" value="left"></el-option>
        <el-option label="右侧" value="right"></el-option>
      </el-select>
    </div>
    <span slot="footer" class="dialog-footer">
      <el-button @click="cancel">{{ $t('dialog.cancel') || '取消' }}</el-button>
      <el-button type="primary" @click="confirm">{{ $t('dialog.confirm') || '确定' }}</el-button>
    </span>
  </el-dialog>
</template>

<script>
import VideoUpload from '@/components/VideoUpload/index.vue'
import { isMobile } from 'simple-mind-map/src/utils/index'

// 节点视频内容设置
export default {
  components: {
    VideoUpload
  },
  data() {
    return {
      dialogVisible: false,
      video: '',
      videoUrl: '',
      videoTitle: '',
      videoPoster: '',
      videoPlacement: 'top',
      activeNodes: null,
      isMobile: isMobile()
    }
  },
  created() {
    this.$bus.$on('node_active', this.handleNodeActive)
    this.$bus.$on('showNodeVideo', this.handleShowNodeVideo)
  },
  beforeDestroy() {
    this.$bus.$off('node_active', this.handleNodeActive)
    this.$bus.$off('showNodeVideo', this.handleShowNodeVideo)
  },
  methods: {
    handleNodeActive(...args) {
      this.activeNodes = [...args[1]]
    },

    handleShowNodeVideo() {
      this.reset()
      if (this.activeNodes.length > 0) {
        let firstNode = this.activeNodes[0]
        let video = firstNode.getData('video') || ''
        if (video) {
          if (/^https?:\/\//.test(video)) {
            this.videoUrl = video
          } else {
            this.video = video
          }
        }
        this.videoTitle = firstNode.getData('videoTitle') || ''
        this.videoPoster = firstNode.getData('videoPoster') || ''
        this.videoPlacement = firstNode.getStyle('videoPlacement') || 'top'
      }
      this.dialogVisible = true
    },

    handleClose(done) {
      this.reset()
      done()
    },

    cancel() {
      this.dialogVisible = false
      this.reset()
    },

    reset() {
      this.video = ''
      this.videoTitle = ''
      this.videoUrl = ''
      this.videoPoster = ''
      this.videoPlacement = 'top'
    },

    async confirm() {
      try {
        // 删除视频
        if (!this.video && !this.videoUrl) {
          this.cancel()
          this.activeNodes.forEach(node => {
            node.setData({
              video: null,
              videoTitle: '',
              videoPoster: '',
              videoSize: null
            })
            node.setStyle({
              videoPlacement: 'top'
            })
          })
          return
        }
        
        let res = null
        let video = ''
        if (this.video) {
          video = this.video
          res = await this.$refs.VideoUpload.getSize()
        } else if (this.videoUrl) {
          video = this.videoUrl
          // 使用默认尺寸或尝试获取视频尺寸
          res = {
            width: 320,
            height: 240
          }
          
          // 创建一个视频元素来获取尺寸
          const videoEl = document.createElement('video')
          videoEl.preload = 'metadata'
          
          // 尝试获取视频尺寸，超时则使用默认尺寸
          try {
            const sizePromise = new Promise(resolve => {
              videoEl.onloadedmetadata = () => {
                resolve({
                  width: videoEl.videoWidth || 320,
                  height: videoEl.videoHeight || 240
                })
              }
              
              videoEl.onerror = () => {
                resolve({
                  width: 320,
                  height: 240
                })
              }
            })
            
            // 设置超时，避免视频加载过久
            const timeoutPromise = new Promise(resolve => {
              setTimeout(() => {
                resolve({
                  width: 320,
                  height: 240
                })
              }, 3000)
            })
            
            videoEl.src = video
            res = await Promise.race([sizePromise, timeoutPromise])
          } catch (error) {
            console.log('获取视频尺寸失败', error)
          }
        }
        
        this.activeNodes.forEach(node => {
          // 设置视频数据
          node.setData({
            video: video || 'none',
            videoTitle: this.videoTitle,
            videoPoster: this.videoPoster,
            videoSize: {
              width: res.width || 320,
              height: res.height || 240,
              custom: false
            }
          })
          
          // 设置视频位置样式
          node.setStyle({
            videoPlacement: this.videoPlacement
          })
        })
        
        this.cancel()
      } catch (error) {
        console.log(error)
      }
    }
  }
}
</script>

<style lang="less" scoped>
.nodeVideoDialog {
  .title {
    font-size: 18px;
    margin-bottom: 12px;
  }

  .inputBox {
    display: flex;
    align-items: center;
    margin-bottom: 10px;

    .label {
      width: 150px;
    }
  }
}
</style> 