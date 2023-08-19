<template>
  <div class="layout" :class="{'layout-hide-text': spanLeft < 5}">
    <Row type="flex">
      <i-col :span="spanLeft" class="layout-menu-left">
        <Menu active-name="1" theme="dark" width="auto">
        </Menu>
      </i-col>
      <i-col :span="spanRight">
        <div class="layout-header">
          <Page :total="total" :current.sync="cpage" @on-change="changePage" show-elevator
            style="margin-top: 10px; margin-right: 10px;"></Page>
        </div>
        <div class="layout-content">
          <div class="layout-content-main">
            <table>
              <thead>
                <tr>
                  <th>Cover</th>
                  <th>Title</th>
                  <th>Description</th>
                  <th>Episodes</th>
                  <th>State</th>
                  <th>Update time</th>
                  <th>Uploader</th>
                  <th>Link</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in data">
                  <td style="height: 150px; width: 150px;">
                    <img :src="item.coverImg">
                  </td>
                  <th>{{ item.anime_name }}</th>
                  <td>
                    <div class="description">
                      <textarea>
                        {{ item.description.trim() }}
                      </textarea>
                    </div>
                  </td>
                  <th>{{ item.episodes['1'] }}</th>
                  <th>{{ item.status }}</th>
                  <th>{{ item.update_time  }}</th>
                  <th>{{ item.uploader.username }}</th>
                  <th>
                    <div @click="jump(item)" :class="{'jumpEvents': item.episodes['1'] === 0 || item.status === 'Not start', 'jump': item.episodes['1'] !== 0 || item.status !== 'Not start'}">
                      Go to
                      <Icon type="ios-arrow-thin-right"></Icon>
                    </div>
                  </th>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
        <div class="layout-copy">
          2023 &copy; Newly added
        </div>
      </i-col>
    </Row>
  </div>
</template>
<script>
  import api from '@oj/api'
  export default {
    name: 'anime-list',
    props: {
      type: {
        type: String,
        required: false
      }
    },
    data () {
      return {
        spanLeft: 2,
        spanRight: 22,
        limit: 10,
        cpage: 1,
        total: 0,
        data: [],
        klot: ''
      }
    },
    computed: {
      iconSize () {
        return this.spanLeft === 5 ? 14 : 24
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      init () {
        this.getAnimeList()
      },
      changePage () {
        this.getAnimeList()
      },
      getAnimeList () {
        let params = {
          offset: (this.cpage - 1) * this.limit,
          limit: this.limit,
          type: this.type,
          klot: api.getNowKlot()
        }
        api.getAnimeList(params).then(res => {
          this.data = res.data.data.results
          this.total = res.data.data.total
        })
      },
      jump (e) {
        if (e.episodes['1'] === 0) return
        let url = `/animeDetail?id=${e.anime_id}&load=1&ep=${1}`
        window.open(url)
      }
    }
  }
</script>

<style lang="less" scoped>
  .jumpEvents {
    cursor: not-allowed;
  }
  .jump{
    color: #1855f0;
    text-decoration: none;
    cursor: pointer;
  }
  .description {
    height: 150px;
    width: 100%;
    overflow: hidden;
    
    textarea {
      width: 100%;
      height: 100%;
      resize: none;
      border: none;
      padding: 5px;
      text-overflow: ellipsis;
      overflow: hidden;
      pointer-events: none;
      white-space: pre-wrap;
    }
  }

  table {
    border: 1px dashed #ccc;
    tr {
      th {
        border: 1px dashed #ccc;
      }

      td {
        border: 1px dashed #ccc;

        img {
          max-width: 100%;
          max-height: 100%;
        }
      }
    }
    border-collapse: collapse;
  }
  .layout{
    border: 1px solid #d7dde4;
    background: #f5f7f9;
    position: relative;
    border-radius: 4px;
    overflow: hidden;
  }
  .layout-content{
    height: 500px;
    margin: 15px;
    background: #fff;
    border-radius: 4px;
  }
  .layout-content-main{
    height: 500px;
    padding: 10px;
    overflow-y: auto;
  }
  .layout-copy{
    text-align: center;
    padding: 10px 0 20px;
    color: #9ea7b4;
  }
  .layout-menu-left{
    background: #464c5b;
  }
  .layout-header{
    height: 60px;
    background: #fff;
    box-shadow: 0 1px 1px rgba(0,0,0,.1);
    display: flex;
    justify-content: right;
  }
  .layout-ceiling-main a{
    color: #9ba7b5;
  }
  .ivu-col{
    transition: width .2s ease-in-out;
  }
</style>