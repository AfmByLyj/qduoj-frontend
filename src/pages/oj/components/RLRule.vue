<template>
  <div>
    <div class="modal">
      <div style="font-size: 16px; font-weight: bolder;">RL score settlement rules</div>
      <table>
        <thead>
          <tr>
            <th weight="20%">#</th>
            <th weight="30%" colspan="3">{{ $t('m.NavProblems') }}</th>
            <th weight="30%" colspan="3">{{ $t('m.Contests') }}&emsp;
              <span style="font-size: 10px; font-weight: lighter;">{{ $t('m.tipContest') }}</span>
            </th>
          </tr>
          <tr>
            <th weight="20%">{{ $t('m.RL_Score') }}</th>
            <th weight="10%">{{ $t('m.Easy_problem') }}</th>
            <th weight="10%">{{ $t('m.Mid_problem') }}</th>
            <th weight="10%">{{ $t('m.High_problem') }}</th>
            <th weight="10%">{{ $t('m.Contest10T') }}</th>
            <th weight="10%">{{ $t('m.Contest10L') }}</th>
            <th weight="10%">{{ $t('m.ContestM') }}</th>
            <th>Color</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in RLData">
            <td v-for="it, id in item" v-if="id < 7">{{ it }}</td>
            <td :style="{'background': item[7]}"></td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
<script>
export default {
  name: 'RL_rule',
  data () {
    return {
      RLData: [
        ['(-inf, 1000)', '+35', '+60', '+100', '+500', '-100', '+250', 'pink'],
        ['[1000, 2000)', '+24', '+35', '+55', '+(11-x)*(106-x)', '-(11-x)*(66-x)', '+100', 'green'],
        ['[2000, 3000)', '+12', '+45', '+65', '+(11-x)*(96-x)', '-(12-x)*(56-x)', '+80', 'brown'],
        ['[3000, 4000)', '+8', '+55', '+80', '+(11-x)*(76-x)', '-(13-x)*(61-x)', '+60', 'cyan'],
        ['[4000, 5000)', '+4', '+60', '+100', '+(11-x)*(62-x)', '-(21-x)*(81-x)', '+40', 'blue'],
        ['[5000, 8000)', '+1', '+80', '+200', '+(11-x)*(48-x)', '-(31-x)*(101-x)', '+30', 'purple'],
        ['[8000, 10000)', '+0', '+100', '+500', '+(11-x)*(156-x)', '-(51-x)*(201-x)', '+80', 'red'],
        ['[10000, inf)', '+0', '+0', '+0', '+0', '+0', '+0', 'linear-gradient(45deg, black, red)']
      ]
    }
  }
}
</script>
<style scoped lang="less">

table {
  margin-top: 10px;
  border: 1px dashed #ccc;
  tr {
    th {
      border: 1px dashed #ccc;
    }

    td {
      border: 1px dashed #ccc;
      text-align: center;
      border-radius: 5px;
      font-size: 14px;
    }
  }
  border-collapse: collapse;
}
</style>