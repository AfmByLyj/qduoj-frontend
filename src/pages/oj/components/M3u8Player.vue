<template>
  <div>
    <video ref="videoPlayer" controls autoplay="" name="media" width="640" height="480"></video>
  </div>
</template>
<script>
  import Hls from 'hls.js'
  export default {
    name: 'mPlayer',
    props: {
      url: {
        required: true,
        type: String
      }
    },
    mounted () {
      const videoElement = this.$refs.videoPlayer

      const hls = new Hls()
      // 设置视频源为 M3U8 文件的 URL
      hls.loadSource(this.url)
      hls.attachMedia(videoElement)

      // 开始播放视频
      videoElement.play()
    }
  }
</script>