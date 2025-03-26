<template>
  <div class="videoUploadContainer">
    <div class="videoUploadPanel">
      <div class="upBtn" v-if="!value">
        <label
          for="videoUploadInput"
          class="videoUploadInputArea"
          @dragenter.stop.prevent
          @dragover.stop.prevent
          @drop.stop.prevent="onDrop"
          >点击此处选择视频、或拖动视频到此</label
        >
        <input
          type="file"
          accept="video/*"
          id="videoUploadInput"
          @change="onVideoUploadInputChange"
        />
      </div>
      <div v-if="value" class="uploadInfoBox">
        <div class="previewBox">
          <video
            controls
            :src="value"
            width="100%"
            height="100%"
            preload="metadata"
            @click.stop
          ></video>
        </div>
        <span class="delBtn el-icon-close" @click.stop="deleteVideo"></span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  model: {
    prop: 'value',
    event: 'change'
  },
  props: {
    value: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      file: null
    }
  },
  methods: {
    // 视频选择事件
    onVideoUploadInputChange(e) {
      let file = e.target.files[0]
      this.selectVideo(file)
    },

    // 拖动上传视频
    onDrop(e) {
      let dt = e.dataTransfer
      let file = dt.files && dt.files[0]
      this.selectVideo(file)
    },

    // 选择视频
    selectVideo(file) {
      if (!file) return
      
      // 检查文件是否为视频
      if (!file.type.startsWith('video/')) {
        this.$message.error('请选择视频文件')
        return
      }
      
      this.file = file
      let fr = new FileReader()
      fr.readAsDataURL(file)
      fr.onload = e => {
        this.$emit('change', e.target.result)
      }
    },

    // 获取视频大小
    getSize() {
      return new Promise(resolve => {
        const video = document.createElement('video')
        video.preload = 'metadata'
        
        video.onloadedmetadata = () => {
          resolve({
            width: video.videoWidth,
            height: video.videoHeight
          })
        }
        
        video.onerror = () => {
          resolve({
            width: 320,
            height: 240
          })
        }
        
        video.src = this.value
      })
    },

    // 删除视频
    deleteVideo() {
      this.$emit('change', '')
      this.file = null
    }
  }
}
</script>

<style lang="less" scoped>
.videoUploadContainer {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgba(255, 255, 255, 0.9);
  z-index: 1000;

  .videoUploadPanel {
    position: relative;
    width: 100%;
    font-size: 22px;
    white-space: nowrap;
    color: #909090;
    cursor: default;
    user-select: none;

    .title {
      margin-bottom: 15px;
      font-size: 22px;
      font-weight: 700;
      color: hsla(218, 9%, 51%, 0.8);
    }

    .closeBtn {
      position: absolute;
      right: 25px;
      top: 32px;
      cursor: pointer;
    }

    .videoUploadInputArea {
      display: block;
      width: 100%;
      height: 200px;
      font-size: 20px;
      color: rgba(51, 51, 51, 0.4);
      background-color: hsla(0, 0%, 87%, 0.6);
      border: none;
      outline: none;
      cursor: pointer;
      text-align: center;
      display: flex;
      justify-content: center;
      align-items: center;
      white-space: normal;
      padding: 10px;
    }

    #videoUploadInput {
      display: none;
    }

    .uploadInfoBox {
      position: relative;
      width: 100%;
      height: 200px;
      background-color: hsla(0, 0%, 87%, 0.6);

      .previewBox {
        width: 100%;
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        
        video {
          max-width: 100%;
          max-height: 100%;
          background-color: #000;
        }
      }

      .delBtn {
        position: absolute;
        right: 0px;
        top: 0px;
        cursor: pointer;
        width: 20px;
        height: 20px;
        background-color: #fff;
      }
    }
  }
}
</style> 