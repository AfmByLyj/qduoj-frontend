<template>
  <div>
    <div style="display: flex; justify-content: center;">
      <div id="echarts">
        <div>
          <div style="font-size: larger; font-weight: bolder;">{{ $t('m.WeeklySubmissionRank') }}</div>
          <ECharts :options="options" ref="chart" auto-resize></ECharts>
        </div>
      </div>
      <div id="ranks">
        <div id="Top3">
          <div v-for="i in Math.min(3, weekRank.length)" @click="jump(weekRank[i - 1].userName)" class="pre">
            <div class="preimg">
              <img :src="weekRank[i - 1].avatar" style="width: 50px; height: 50px;" class="avatar">
            </div>
            <div class="uname" v-html="weekRank[i - 1].userSpan.replace('|', weekRank[i - 1].userName)"></div>
          </div>
        </div>
        <div id="alls">
          <table>
            <tr v-for="wr, indexs in weekRank" v-if="indexs + 1 > 3">
              <td style="width: 30px; text-align: center;">{{ indexs + 1 }}</td>
              <td style="width: 50px; height: 60px;">
                <div style="height: 100%; display: flex; align-items: center;" @click="jump(wr.userName)">
                  <img :src="wr.avatar" style="width: 40px; height: 40px;" class="avatar">
                </div>
              </td>
              <td>
                <div class="uname" @click="jump(wr.userName)" v-html="wr.userSpan.replace('|', wr.userName)"></div>
                <div>AC: {{ wr.acNum }}</div>
              </td>
            </tr>
          </table>
        </div>
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
  }

  td {
    border-bottom: 1px solid #605f5f;
  }

  #echarts {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 300px;
    height: 620px;
    border-radius: 20px;
    background-color: #fafafa;
    width: 750px;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
  }
  #ranks {
    height: 600px;
    width: 300px;
    position: relative;
    left: 50px;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  #alls {
    height: 420px;
    width: 300px;
    position: relative;
    top: 20px;
    border-radius: 20px;
    background-color: #fafafa;
    overflow: hidden;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
  }

  #Top3 {
    width: 300px;
    height: 200px;
    position: relative;
    /* overflow: hidden; */
    /* border: 1px solid black; */
    background-image: url('./trangle.png');
    background-size: cover;
    background-position: center;
  }

  /* #Top3::before {
    content: '';
    display: block;
    width: 300px;
    height: 300px;
    top: 20px;
    left: 0px;
    border-radius: 100px;
    position: absolute;
    transform: rotate(45deg);
    background-color: orange;
  } */

  .pre {
    position: absolute;
    width: 60px;
    height: 80px;
    cursor: pointer;
    display: block;
  }

  .uname {
    cursor: pointer;
    width: 60px;
    /* color: rgb(255, 161, 22); */
  }

  .pre .uname {
    margin-top: 5px;
    text-align: center;
    overflow: hidden;
    text-overflow: ellipsis;
    background-color: rgb(255, 255, 255);
    border-radius: 5px;
    border: 0.5px solid #9d9c99;
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
    box-shadow: 0 5px 15px 0 rgba(0, 0, 0, 0.2);
  }

  .pre:nth-child(1) {
    top: 5%;
    left: calc(50% - 30px);
  }

  .pre:nth-child(1) .preimg {
    background: linear-gradient(180deg, #e7e78b, #cbb508);
  }

  .pre:nth-child(2) {
    /* left: calc(25% - 30px); */
    left: 5%;
    bottom: 0;
  }

  .pre:nth-child(2) .preimg {
    background: linear-gradient(180deg, #c6c6c1, #9d9c99);
  }

  .pre:nth-child(3) {
    /* right: calc(25% - 30px); */
    right: 5%;
    bottom: 0;
  }

  .pre:nth-child(3) .preimg {
    background: linear-gradient(180deg, #e7b462, #c37c0a);
  }

  .avatar {
    border-radius: 50%;
    border-width: 5px;
    border-style: solid;
    cursor: pointer;
  }
</style>