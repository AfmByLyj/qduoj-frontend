<template>
  <div>
    <Panel title="Anime List">
      <el-pagination
        background
        layout="prev, pager, next"
        :total="total"
        :current-page.sync="currentPage"
        @current-change="currentChange">
      </el-pagination>
      <el-table
        v-loading="loading"
        element-loading-text="loading"
        stripe
        :data="animeList"
        height="500"
        style="margin-top: 10px;">
        <el-table-column
          fixed
          width="80"
          label="Cover">
          <template slot-scope="props">
            <img :src="props.row.coverImg" class="cover">
          </template>
        </el-table-column>
        <el-table-column
          prop="anime_id"
          width="80"
          label="ID">
        </el-table-column>
        <el-table-column
          prop="anime_name"
          width="160"
          label="Title">
        </el-table-column>
        <el-table-column
          prop="status"
          width="160"
          label="Status">
        </el-table-column>
        <el-table-column
          prop="update_time"
          width="240"
          label="Update">
        </el-table-column>
        <el-table-column
          fixed="right"
          width="320"
          label="Operation">
          <div slot-scope="scope">
            <icon-btn name="Edit" icon="edit" @click.native="goEdit(scope.row.anime_id)"></icon-btn>
          </div>
        </el-table-column>
      </el-table>
    </Panel>
  </div>
</template>
<script>
  import api from '../../api'
  export default {
    name: 'animeList',
    props: {
      type: {
        type: String,
        required: false
      },
      page: {
        type: Number,
        required: true
      }
    },
    data () {
      return {
        loading: false,
        animeList: [],
        total: 0,
        limit: 10,
        currentPage: this.page,
        currentId: 1
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      init () {
        this.currentPage = this.page
        this.getAnimeList(this.currentPage)
      },
      currentChange (page) {
        this.currentPage = page
        this.page = page
        this.getAnimeList(page)
        let url = '/animeList?page=' + page
        if (this.type) url += '&type=' + this.type
        this.$router.push(url)
      },
      getAnimeList (page) {
        this.loading = true
        let params = {
          'klot': api.getNowKlot(),
          'type': this.type,
          'offset': (page - 1) * this.limit,
          'limit': this.limit
        }
        api.getAnimeList(params).then(res => {
          this.loading = false
          this.total = res.data.data.total
          this.animeList = res.data.data.results
        }, res => {
          this.loading = false
        })
      },
      goEdit (id) {
        let url = `/admin/animeEdit?id=${id}`
        window.open(url)
      }
    }
  }
</script>
<style scoped lang="less">
.cover {
  height: 80px;
  width: 80px;
}
</style>