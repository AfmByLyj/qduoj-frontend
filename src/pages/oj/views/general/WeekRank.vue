<template>
  <div style="display: flex;">
    <div id="echarts">
      <ECharts :options="options" ref="chart" auto-resize></ECharts>
    </div>
    <div id="ranks">
      <div id="Top3">
        <div v-for="i in Math.min(3, weekRank.length)" @click="jump(weekRank[i - 1].userName)" class="pre">
          <div class="preimg">
            <img :src="weekRank[i - 1].avatar" style="width: 50px; height: 50px;" class="avatar">
          </div>
          <div class="uname">{{ weekRank[i - 1].userName }}</div>
        </div>
      </div>
      <div id="alls">
        <table>
          <tr v-for="wr, indexs in weekRank">
            <td>{{ indexs + 1 }}</td>
            <td style="width: 50px; height: 50px;">
              <div style="height: 100%; display: flex; align-items: center;">
                <img :src="wr.avatar" style="width: 40px; height: 40px;" class="avatar">
              </div>
            </td>
            <td>
              <div class="uname">{{ wr.userName }}</div>
              <div>AC: {{ wr.acNum }}</div>
            </td>
          </tr>
        </table>
      </div>
    </div>
  </div>
</template>
<script>
  import api from '@oj/api'
  import utils from '@/utils/utils'
  export default {
    name: 'WeekRank',
    data () {
      return {
        weekRank: [],
        weekTotal: 0,
        weekpage: 1,
        dailytotal: [],
        options: {
          tooltip: {
            trigger: 'axis'
          },
          grid: {
            x: '3%',
            x2: '3%'
          },
          toolbox: {
            show: true,
            feature: {
              magicType: {show: true, type: ['line', 'bar', 'pie']}
            },
            right: '10%'
          },
          calculable: true,
          xAxis: [
            {
              type: 'category',
              data: [],
              boundaryGap: true,
              axisLabel: {
                interval: 0,
                showMinLabel: true,
                showMaxLabel: true,
                align: 'center',
                formatter: (value, index) => {
                  return utils.breakLongWords(value, 14)
                }
              },
              axisTick: {
                alignWithLabel: true
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
              name: this.$i18n.t('m.AcNum'),
              type: 'line',
              data: [0],
              label: {
                show: true,
                position: 'top'
              }
            }
          ]
        }
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      init () {
        let bar = this.$refs.chart
        bar.showLoading({maskColor: 'rgba(250, 250, 250, 0.8)'})
        this.getWeekRank(7)
      },
      getWeekRank (dayFrame) {
        api.getDayRank(dayFrame).then(res => {
          let data = res.data.data
          if (this.weekRank.length === 0) {
            this.weekRank = data.rank
            this.weekTotal = data.total
          }
          this.dailytotal = data.data
          this.changeCharts()
        })
      },
      changeCharts () {
        let dailytotalLength = this.dailytotal.length
        let dailyT = new Array(dailytotalLength)
        let date = new Array(dailytotalLength)
        for (let i = 0; i < dailytotalLength; i++) {
          dailyT[i] = this.dailytotal[i].total
          date[i] = this.dailytotal[i].Time
        }
        dailyT.reverse()
        date.reverse()
        this.options.xAxis[0].data = date
        this.options.series[0].data = dailyT
        this.$refs.chart.hideLoading()
      },
      jump (uname) {
        this.$router.push(
          {
            name: 'user-home',
            query: {username: uname}
          })
      }
    }
  }
</script>

<style scoped>
  table {
    border-collapse: collapse;
    margin: 5px;
  }

  td {
    border-bottom: 1px solid #605f5f;
  }

  #echarts {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 300px;
    height: 500px;
  }
  #ranks {
    height: 500px;
    width: 200px;
    position: relative;
    top: 10px;
  }

  #alls {
    height: 300px;
    width: 200px;
    position: relative;
    top: 20px;
    border-radius: 20px;
    background-color: #fafafa;
    overflow: hidden;
  }

  #Top3 {
    width: 200px;
    height: 160px;
  }

  .pre {
    position: relative;
    width: 60px;
    height: 80px;
    cursor: pointer;
    display: block;
  }

  .uname {
    width: 60px;
    color: rgb(255, 161, 22);
  }

  .pre .uname {
    margin-top: 5px;
    text-align: center;
    overflow: hidden;
    text-overflow: ellipsis;
    background-color: white;
    border-radius: 5px;
    opacity: .9;
    box-shadow: 0 5px 15px 0 rgba(0, 0, 0, 0.08);
  }

  #ranks #Top3 .preimg {
    height: 60px;
    width: 60px;
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: center;
    box-shadow: 0 5px 15px 0 rgba(0, 0, 0, 0.08);
  }

  .pre:nth-child(1) {
    left: 70px;
  }

  .pre:nth-child(1) .preimg {
    background: linear-gradient(180deg, #e7e78b, #cbb508);
  }

  .pre:nth-child(2) .preimg {
    background: linear-gradient(180deg, #c6c6c1, #9d9c99);
  }

  .pre:nth-child(3) {
    left: 140px;
    top: -80px;
  }

  .pre:nth-child(3) .preimg {
    background: linear-gradient(180deg, #e7b462, #c37c0a);
  }

  .avatar {
    border-radius: 50%;
    border-width: 5px;
    border-style: solid;
  }
</style>