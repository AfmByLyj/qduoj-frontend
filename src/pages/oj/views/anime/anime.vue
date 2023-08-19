<template>
  <div class="Tbox">
    <div class="LeftBox">
      <div class="loadBox">
        <div v-for="item in LE_data" class="loadSingle" :class="{'selectL': item.key===LE_data[selectLoad].key, 'noSelect': item.key!==LE_data[selectLoad].key}">
          <span>Road {{ item.key }}</span>
        </div>
      </div>
      <div class="epBox" v-if="show">
        <div v-for="index in LE_data[selectLoad].value" :class="{'selectE': index.toString()===ep}" 
          @click="jumpNewUrl(LE_data[selectLoad].key, index)">
          {{ index }}
          <span>{{ getTitle(LE_data[selectLoad].key, index) }}</span>
        </div>
      </div>
    </div>
    <div class="RightBox" v-if="show">
      <span>{{ getTitle(load, ep) }}</span>
      <div v-if="url === ''"> 
        Please contact the administrator, the URL is empty!
      </div>
      <div v-else-if="IsIframe()">
        <iframe :src="url" allowfullscreen="allowfullscreen" width="640" height="480" scrolling="no" frameborder="0" sandbox="allow-top-navigation allow-same-origin allow-forms allow-scripts"></iframe>
      </div>
      <div v-else-if="IsM3u8()">
        <!-- <iframe :src="'https://tools.liumingye.cn/m3u8/#'+url" width="640" height="480" scrolling="no" frameborder="0" sandbox="allow-top-navigation allow-same-origin allow-forms allow-scripts"></iframe> -->
        <M3u8Player :url="url"></M3u8Player>
      </div>
      <div v-else-if="IsMp4()">
        <video controls="controls" autoplay="" name="media" width="640" height="480" >
          <source :src="url" type="video/mp4">
        </video>
      </div>
    </div>
  </div>
</template>
<script>
  import api from '@oj/api.js'
  import M3u8Player from '../../components/M3u8Player.vue'
  export default {
    name: 'animeDetail',
    components: {
      M3u8Player
    },
    props: [
      'load',
      'ep',
      'id'
    ],
    data () {
      return {
        formats: '',
        url: '',
        formatsList: ['iframe', 'mp4', 'm3u8'],
        LE_data: [],
        LE_detail: {},
        selectLoad: 0,
        show: false
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      init () {
        this.LE_data = []
        let params = {
          'id': this.id
        }
        api.getAnimeDetail(params).then(res => {
          let data = res.data.data.all.episodes
          let detail = res.data.data.detail
          this.LE_detail = {}
          for (let key in data) {
            this.LE_data.push({
              'key': key,
              'value': data[key]
            })
          }
          this.selectLoad = 0
          detail.forEach(e => {
            this.LE_detail[[e.father_road, e.father_episode]] = {
              'title': e.title
            }
          })
          this.show = true
        })
        this.getInfo()
      },
      getInfo () {
        let params = {
          'id': this.id,
          'load': this.load,
          'ep': this.ep
        }
        api.getAnimeDetail(params).then(res => {
          let data = res.data.data
          this.formats = data.format
          this.url = data.url
        })
      },
      IsIframe () {
        return this.formats === this.formatsList[0]
      },
      IsMp4 () {
        return this.formats === this.formatsList[1]
      },
      IsM3u8 () {
        return this.formats === this.formatsList[2]
      },
      jumpNewUrl (load, ep) {
        let url = `/animeDetail?id=${this.id}&load=${load}&ep=${ep}`
        this.ep = ep
        this.$router.push(url)
        window.location.reload()
      },
      getTitle (load, ep) {
        return this.LE_detail[[load, ep]].title
      }
    }
  }
</script>
<style scoped lang="less">
.Tbox {
  display: flex;
  .LeftBox {
    position: relative;
    height: 600px;
    width: 45%;
    min-width: 400px;
    border: 1px solid black;
    display: flex;
    .loadBox {
      width: 37.5%;
      min-width: 150px;
      height: 100%;
      border: 1px solid black;
      box-shadow: 0 0 5px rgba(83, 83, 83, 0.7);
      .loadSingle {
        height: 80px;
        display: flex;
        justify-content: center;
        align-items: center;
        background: #464c5b;
        .noSelect {
          border-bottom: 5px solid #530a01;
        }
        cursor: pointer;
        span {
          color: antiquewhite;
          font-size: medium;
        }
      }
    }
    .epBox {
      margin-left: 10px;
      width: calc(62.5% - 20px);
      min-width: 230px;
      div {
        height: 50px;
        width: 100%;
        line-height: 50px;
        font-weight: bold;
        font-size: medium;
        border-bottom: 2px solid #48d0cc;
        cursor: pointer;
      }
    }
  }
  .RightBox {
    position: relative;
    left: 5%;
    width: 50%;
    min-width: 700px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    span {
      font-size: larger;
      font-weight: bolder;
      margin-bottom: 20px;
    }
  }
}
.selectL {
  border-bottom: 5px solid #0149ff;
}
.selectE {
  color: #b01b07;
}
</style>