<template>
  <div>
    <Row type="flex" justify="space-around">
      <Col :span="22">
      <Panel :padding="10">
        <div slot="title" class="titles">
          <span>{{$t('m.RL_Ranklist')}}</span>
          <div id="helpIcon" 
          @mouseover="tipBox = true"
          @mouseleave="tipBox = false"
          @click="RLbox=true">
            <Icon type="help" size="12"></Icon>
          </div>
          <div class="tipBox" v-show="tipBox">
            What is RL rule?
          </div>
        </div>
        <div class="Rank">
          <table>
            <thead>
              <tr>
                <th width="10%">{{ $t('m.RL_Rank') }}</th>
                <th width="30%">{{ $t('m.Username') }}</th>
                <th width="20%">{{ $t('m.mood') }}</th>
                <th width="30%">{{ $t('m.UserHomeSolved') }}</th>
                <th width="10%">RL</th>
              </tr>
            </thead>
            <tbody align="center">
              <tr v-for="data, idx in dataRank">
                <td width="10%">
                  <span>{{ index[idx] }}</span>
                </td>
                <td width="30%">
                  <span v-if="data.userSpan !== ''" v-html="data.userSpan.replace('|', data.user.username)" style="font-size: 16px;"></span>
                  <span v-else style="font-size: 16px;">{{ data.user.username }}</span>
                </td>
                <td width="20%" height="45px">{{ data.mood }}</td>
                <td width="30%">{{ data.accepted_number }}</td>
                <td width="10%">
                  <span v-if="data.userSpan !== ''" v-html="data.userSpan.replace('|', data.RL_score)"></span>
                  <span v-else>{{ data.RL_score }}</span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </Panel>
      <Pagination :total="total" :page-size.sync="limit" :current.sync="page"
                  @on-change="getRankData" show-sizer
                  @on-page-size-change="getRankData(1)"></Pagination>
      </Col>
    </Row>

    <Modal v-model="RLbox" :width="600">
      <RLRule></RLRule>
    </Modal>
  </div>
</template>

<script>
  import api from '@oj/api'
  import Pagination from '@oj/components/Pagination'
  import RLRule from '../../components/RLRule.vue'
  // import utils from '@/utils/utils'

  export default {
    name: 'rl-rank',
    components: {
      Pagination,
      RLRule
    },
    data () {
      return {
        page: 1,
        limit: 30,
        total: 0,
        loadingTable: false,
        dataRank: [],
        index: [],
        tipBox: false,
        RLbox: false
      }
    },
    mounted () {
      this.getRankData(1)
    },
    methods: {
      getRankData (page) {
        let offset = (page - 1) * this.limit
        this.loadingTable = true
        api.getRLRank(offset, this.limit).then(res => {
          this.loadingTable = false
          this.total = res.data.data.total
          this.dataRank = res.data.data.results
          this.index = [offset + 1]
          for (let i = 1; i < this.dataRank.length; i++) {
            this.index.push(this.index.slice(-1)[0] + (this.dataRank[i].RL_score !== this.dataRank[i - 1].RL_score ? 1 : 0))
          }
        }).catch(() => {
          this.loadingTable = false
        })
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
.titles {
  height: 20px;
}

#helpIcon {
  position: relative;
  top: -30px;
  height: 15px;
  width: 15px;
  border-radius: 50%;
  border-width: 2px;
  border-color: black;
  border-style: solid;
  left: 100px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}

.tipBox {
  position: absolute;
  height: 25px;
  width: 150px;
  border-radius: 5px;
  border: 2px solid #7a7a7a;
  background-color: rgb(240, 240, 240);
  box-shadow: 0 5px 5px rgba(125, 125, 125, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  text-shadow: 0 0 25px #7a7a7a;
  font-size: 16px;
  font-weight: bolder;
  top: 0px;
  left: 150px;
}

.Rank {
  table {
    border: 1px dashed #ccc;
    tr {
      th {
        border: 1px dashed #ccc;
      }

      td {
        border: 1px dashed #ccc;
      }
    }
    border-collapse: collapse;
  }
}

.rankTop {
  background: url(https://static.nowcoder.com/images/img/platform/rating-num.png) no-repeat;
}

</style>
