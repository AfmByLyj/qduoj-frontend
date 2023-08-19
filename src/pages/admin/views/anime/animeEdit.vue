<template>
  <div>
    <Panel title="Edit anime">
      <div>
        Name:
        <el-input v-model="anime_name" placeholder="Name"></el-input>
      </div>
      
      <div style="margin-top: 20px;">
        ID:
        <el-input v-model="anime_id" placeholder="ID"></el-input>
      </div>

      <div style="margin-top: 20px;">
        Description:
        <el-input 
          type="textarea"
          :rows="10"
          v-model="description" 
          placeholder="Description">
        </el-input>
      </div>

      <div style="margin-top: 20px;">
        Anime Status:
        <el-select
        v-model="showStatus"
        :disabled="Sdisabled">
          <el-option
          v-for="item in selectStatus"
          :key="item.value"
          :label="item.label"
          :value="item.value"
          :disabled="item.disabled">
          </el-option>
        </el-select>
      </div>

      <div style="margin-top: 20px;">
        <div>
          <div>Cover:</div>
          <div style="margin-left: 50px;">
            <div>
              <div>Old:</div>
              <el-image
                style="border: 1px solid #110; width: 150px; height: 150px;"
                :src="coverImg">
              </el-image>
            </div>
            <div>
              <div>New:</div>
              <el-image
                v-if="uploadImg !== ''"
                style="border: 1px solid #110; width: 150px; height: 150px;"
                :src="uploadImg">
              </el-image>
              <el-upload
                ref="uploadCover"
                action="action"
                :before-upload="on_upload"
                multiple
                :limit="1"
                accept=".png,.jpg,.jpeg,.bmp">
                <el-button type="primary">Upload<i class="el-icon-upload el-icon--right"></i></el-button>
              </el-upload>
            </div>
          </div>
        </div>
      </div>

      <div style="margin-top: 20px;">
        <div>Content:</div>
        <el-container style="height: 500px; border: 1px solid #eee;">
          <el-aside width="200px" style="background-color: rgb(238, 241, 246);">
            <el-menu>
              <el-menu-item v-for="item, index in LE_data" :key="index">
                Road {{ item.key }}
              </el-menu-item>
            </el-menu>
          </el-aside>
          <el-main>
            <el-table
              :data="LE_detail[selectKey]">
              <el-table-column
                prop="ep"
                width="80"
                label="#"></el-table-column>
              <el-table-column
                label="Title">
                <template slot-scope="props">
                  <el-input v-model="props.row.title"></el-input>
                </template>
              </el-table-column>
              <el-table-column
                label="Format">
                <template slot-scope="props">
                  <el-select v-model="props.row.format">
                    <el-option
                      v-for="item in selectFormats"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value">
                    </el-option>
                  </el-select>
                </template>
              </el-table-column>
              <el-table-column
                label="Url">
                <template slot-scope="props">
                  <el-input v-model="props.row.url"></el-input>
                </template>
              </el-table-column>
              <el-table-column
                width="160"
                label="Operation">
                <template slot-scope="props">
                  <icon-btn name="Delete" icon="remove"
                    @click.native="removeEp(selectKey, props.row.ep)"></icon-btn>
                </template>
              </el-table-column>
            </el-table>
            <div style="margin-top: 10px;">
              <icon-btn name='Add a new episode' icon="plus-square" v-if="showStatus !== 'Completed'"
                @click.native="addNewEp(selectKey)"></icon-btn>
            </div>
          </el-main>
        </el-container>
      </div>
      
      <div style="margin-top: 10px;">
        <el-button type="primary" @click="toUploadCover" :loading="IsUploading">
          Submit
          <i :class="showSubmit[showSubmitId]"></i>
        </el-button>
      </div>
    </Panel>
  </div>
</template>
<script>
  import api from '../../api'
  export default {
    name: 'animeEdit',
    props: [
      'id'
    ],
    data () {
      return {
        EditData: {},
        description: '',
        anime_name: '',
        anime_id: '',
        coverImg: '',
        uploadImg: '',
        uploadtype: '',
        uploadSize: 0,
        selectStatus: [{
          value: 'Not start',
          label: 'Not start',
          disabled: false
        },
        {
          value: 'Serializing',
          label: 'Serializing',
          disabled: false
        },
        {
          value: 'Completed',
          label: 'Completed',
          disabled: false
        }],
        selectFormats: [{
          value: 'iframe',
          label: 'iframe'
        },
        {
          value: 'mp4',
          label: 'mp4'
        },
        {
          value: 'm3u8',
          label: 'm3u8'
        }],
        showStatus: 'Not start',
        showSubmit: ['el-icon-s-promotion', ''],
        showSubmitId: 0,
        IsUploading: false,
        LE_data: [],
        LE_detail: {},
        selectKey: '1',
        Sdisabled: true,
        isCover: false
      }
    },
    computed: {
      isCompleted () {
        return this.isCover
      }
    },
    watch: {
      isCompleted (n) {
        if (n) {
          this.submitInfo()
        }
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      init () {
        this.description = ''
        this.anime_name = ''
        this.anime_id = ''
        this.showStatus = 'Not start'
        this.showSubmitId = 0
        this.IsUploading = false
        this.isCover = false
        this.uploadImg = ''
        this.uploadSize = 0
        this.uploadtype = ''
        this.getAnimeDetail()
      },
      beforeUpload (file) {
        const isJPG = file.type === 'image/jpeg'
        const isJPG2 = file.type === 'image/jpg'
        const isPNG = file.type === 'image/png'
        const isLt5M = file.size / 1024 / 1024 < 5
        if (!isJPG && !isJPG2 && !isPNG) {
          this.$message.warning('Only supports images in jpg or png format')
        }
        if (!isLt5M) {
          this.$message.warning('Please upload images within 5MB!')
        }
        return (isJPG || isJPG2 || isPNG) && isLt5M
      },
      on_upload (file) {
        let isOk = this.beforeUpload(file)
        if (!isOk) {
          return false
        }
        let fr = new window.FileReader()
        fr.onload = (e) => {
          this.uploadImg = e.target.result
          this.uploadSize = file.size
          this.uploadtype = file.type.split('/')[1]
        }
        fr.readAsDataURL(file)
        return false
      },
      toUploadCover () {
        let isOk = this.uploadImg === '' || this.uploadSize === 0 || this.uploadtype === ''
        if (isOk) {
          this.submitInfo()
          return
        }
        let data = {
          'anime_id': this.id,
          'cover': this.uploadImg,
          'size': this.uploadSize,
          'suffix': this.uploadtype
        }
        api.uploadCover(data).then(res => {
          this.isCover = true
        })
      },
      getAnimeDetail () {
        this.IsUploading = true
        let params = {
          'id': this.id
        }
        api.getAnimeDetail(params).then(res => {
          let data = res.data.data.all.episodes
          let detail = res.data.data.detail
          this.LE_data = []
          this.LE_detail = {}
          for (let key in data) {
            this.LE_data.push({
              'key': key,
              'value': data[key]
            })
          }
          if (this.LE_data.length > 0) {
            this.selectKey = this.LE_data[0].key
            this.monitor('1')
          }
          if (detail.length === 0) this.LE_detail[this.selectKey] = new Array(1)
          else {
            detail.forEach(e => {
              if (!(e.father_road in this.LE_detail)) {
                this.LE_detail[e.father_road] = new Array(data[e.father_road] + 1)
              }
              this.LE_detail[e.father_road][e.father_episode] = {
                'title': e.title,
                'format': e.anime_format,
                'url': e.anime_resource,
                'ep': e.father_episode
              }
            })
          }

          let All = res.data.data.all
          this.description = All.description
          this.anime_name = All.anime_name
          this.anime_id = All.anime_id
          this.showStatus = All.status
          this.coverImg = All.coverImg
          this.IsUploading = false
        }, res => {
          this.IsUploading = false
        })
      },
      submitInfo () {
        this.showSubmitId = 1
        this.IsUploading = true
        let data = {
          anime_name: this.anime_name,
          anime_id: this.anime_id,
          status: this.showStatus,
          description: this.description,
          episodes: this.makeEpDict(),
          detail: this.LE_detail,
          pid: this.id
        }
        api.modifyAnimeInfo(data).then(res => {
          window.location.reload()
        }, res => {
          this.showSubmitId = 0
          this.IsUploading = false
        })
      },
      makeEpDict () {
        let data = this.LE_data
        let res = {}
        data.forEach(e => {
          res[e.key] = e.value
        })
        return res
      },
      getTitle (road, ep) {
        return this.LE_detail[road][ep].title
      },
      getFormat (road, ep) {
        return this.LE_detail[road][ep].format
      },
      getUrl (road, ep) {
        return this.LE_detail[road][ep].url
      },
      addNewEp (road) {
        if (!this.LE_detail[road]) this.LE_detail[road] = [{}]
        let n = this.LE_detail[road].length
        this.LE_detail[road].push({
          'title': 'Unknow',
          'format': 'iframe',
          'url': '',
          'ep': n
        })
        this.LE_data[parseInt(road) - 1].value++
        this.monitor(road)
      },
      removeEp (road, ep) {
        this.LE_detail[road].splice(ep, 1)
        let n = this.LE_detail[road].length
        for (let i = ep; i < n; i++) {
          this.LE_detail[road][i].ep = i
        }
        this.LE_data[parseInt(road) - 1].value--
        this.monitor(road)
      },
      monitor (road) {
        if (this.LE_data[parseInt(road) - 1].value <= 0) {
          this.showStatus = 'Not start'
          this.Sdisabled = true
        } else {
          this.Sdisabled = false
          this.showStatus = 'Serializing'
          this.selectStatus[0].disabled = true
        }
      }
    }
  }
</script>
<style scoped lang="less">
</style>