<template>
  <div>
    <Panel title="Create new anime">
      <div>
        Name:
        <el-input v-model="anime_name" placeholder="Name"></el-input>
      </div>
      
      <div style="margin-top: 10px;">
        ID:
        <el-input v-model="anime_id" placeholder="ID"></el-input>
      </div>
      <div style="margin-top: 10px;">
        Description:
        <el-input 
          type="textarea"
          :rows="10"
          v-model="description" 
          placeholder="Description">
        </el-input>
      </div>
      <div style="margin-top: 10px;">
        Anime Status:
        <el-select
        v-model="showStatus">
          <el-option
          v-for="item in selectStatus"
          :key="item.value"
          :label="item.label"
          :value="item.value">
          </el-option>
        </el-select>
      </div>
      
      <div style="margin-top: 10px;">
        <el-button type="primary" @click="submitInfo" :loading="IsUploading">
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
    name: 'anime',
    props: [
      'type'
    ],
    data () {
      return {
        description: '',
        anime_name: '',
        anime_id: '',
        selectStatus: [{
          value: 'Not start',
          label: 'Not start'
        },
        {
          value: 'Serializing',
          label: 'Serializing'
        },
        {
          value: 'Completed',
          label: 'Completed'
        }],
        showStatus: 'Not start',
        showSubmit: ['el-icon-s-promotion', ''],
        showSubmitId: 0,
        IsUploading: false
      }
    },
    methods: {
      init () {
        this.description = ''
        this.anime_name = ''
        this.anime_id = ''
        this.showStatus = 'Not start'
        this.showSubmitId = 0
        this.IsUploading = false
      },
      submitInfo () {
        this.showSubmitId = 1
        this.IsUploading = true
        let data = {
          anime_name: this.anime_name,
          anime_id: this.anime_id,
          status: this.showStatus,
          description: this.description
        }
        console.log(data)
        if (this.type) data.category = 'animation'
        api.addAnime(data).then(res => {
          this.init()
        }, res => {
          this.showSubmitId = 0
          this.IsUploading = false
        })
      }
    }
  }
</script>