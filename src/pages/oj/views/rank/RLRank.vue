<template>
  <Row type="flex" justify="space-around">
    <Col :span="22">
    <Panel :padding="10">
      <div slot="title">{{$t('m.RL_Ranklist')}}</div>
      <div class="echarts">
        <ECharts :options="options" ref="chart" auto-resize></ECharts>
      </div>
    </Panel>
    <Table :data="dataRank" :columns="columns" :loading="loadingTable" size="large"></Table>
    <Pagination :total="total" :page-size.sync="limit" :current.sync="page"
                @on-change="getRankData" show-sizer
                @on-page-size-change="getRankData(1)"></Pagination>
    </Col>
  </Row>
</template>

<script>
  import api from '@oj/api'
  import Pagination from '@oj/components/Pagination'
  import utils from '@/utils/utils'

  export default {
    name: 'rl-rank',
    components: {
      Pagination
    },
    data () {
      return {
        page: 1,
        limit: 30,
        total: 0,
        loadingTable: false,
        dataRank: [],
        columns: [
          {
            align: 'center',
            width: 60,
            render: (h, params) => {
              return h('span', {}, params.index + (this.page - 1) * this.limit + 1)
            }
          },
          {
            title: this.$i18n.t('m.User_User'),
            align: 'center',
            render: (h, params) => {
              return h('div', {
                style: {
                  'display': 'flex',
                  'align-items': 'center',
                  'justify-content': 'center'
                }
              }, [
                h('img', {
                  style: {
                    'height': '50px',
                    'width': '50px',
                    'border-radius': '50%',
                    'box-shadow': '0 0 5px rgba(0, 0, 0, 0.5)'
                  },
                  attrs: {
                    src: params.row.avatar
                  }
                }),
                h('div', {
                  style: {
                    'margin-left': '20px',
                    'font-size': '16px',
                    'font-weight': '600'
                  },
                  domProps: {
                    innerHTML: params.row.userSpan
                  },
                  on: {
                    click: () => {
                      this.$router.push({
                        name: 'user-home',
                        query: { username: params.row.user.username }
                      })
                    }
                  }
                })
              ])
            }
          },
          {
            title: this.$i18n.t('m.mood'),
            align: 'center',
            key: 'mood'
          },
          {
            title: this.$i18n.t('m.Score'),
            align: 'center',
            key: 'RL_score'
          },
          {
            title: this.$i18n.t('m.Trend'),
            align: 'center',
            render: (h, params) => {
              let t = this.getTrend(params.row.RL_get.date)
              return h('span', {
                style: {
                  'color': t[1],
                  'font-size': '16px',
                  'font-weight': '600'
                }
              }, t[0])
            }
          }
        ],
        options: {
          tooltip: {
            trigger: 'axis'
          },
          toolbox: {
            show: true,
            feature: {
              dataView: {show: true, readOnly: true},
              magicType: {show: true, type: ['line', 'bar']},
              saveAsImage: {show: true}
            },
            right: '10%'
          },
          calculable: true,
          xAxis: [
            {
              type: 'category',
              data: ['root'],
              axisLabel: {
                interval: 0,
                showMinLabel: true,
                showMaxLabel: true,
                align: 'center',
                formatter: (value, index) => {
                  return utils.breakLongWords(value, 10)
                }
              }
            }
          ],
          yAxis: [
            {
              type: 'value'
            }
          ],
          series: [
            {
              name: this.$i18n.t('m.Score'),
              type: 'bar',
              data: [0]
            }
          ]
        }
      }
    },
    mounted () {
      this.getRankData(1)
    },
    methods: {
      getRankData (page) {
        let offset = (page - 1) * this.limit
        let bar = this.$refs.chart
        bar.showLoading({maskColor: 'rgba(250, 250, 250, 0.8)'})
        this.loadingTable = true
        api.getRLRank(offset, this.limit).then(res => {
          this.loadingTable = false
          if (page === 1) {
            this.changeCharts(res.data.data.results.slice(0, 10))
          }
          this.total = res.data.data.total
          this.dataRank = res.data.data.results
          bar.hideLoading()
        }).catch(() => {
          this.loadingTable = false
          bar.hideLoading()
        })
      },
      changeCharts (rankData) {
        let [usernames, acData] = [[], []]
        rankData.forEach(ele => {
          usernames.push(ele.user.username)
          acData.push(ele.RL_score)
        })
        this.options.xAxis[0].data = usernames
        this.options.series[0].data = acData
      },
      getTrend (data) {
        if (!data) return ['0', 'black']
        let curDate = new Date()
        let cur = curDate.getFullYear() + '-' + (curDate.getMonth() + 1) + '-' + curDate.getDate()
        curDate.setDate(curDate.getDate() - 1)
        let yesterday = curDate.getFullYear() + '-' + (curDate.getMonth() + 1) + '-' + curDate.getDate()
        let score = (data[cur] ? data[cur] : 0) + (data[yesterday] ? data[yesterday] : 0)
        if (score >= 0) return ['+' + score, 'green']
        return [score, 'red']
      }
    }
  }
</script>

<style scoped lang="less">
  .echarts {
    margin: 0 auto;
    width: 95%;
    height: 400px;
  }
</style>
