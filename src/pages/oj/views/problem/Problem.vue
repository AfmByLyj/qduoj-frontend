<template>
  <div class="flex-container">
    <div id="problem-main">
      <!--problem main-->
      <Panel shadow>
        <div style="text-align: center; font-size: 30px;">
          <span>{{problem.title}}</span>
        </div>
        <div style="text-align: center;">
          <div>
            <span>time limit per test:&emsp;</span>{{problem.time_limit}}ms
          </div>
          <div>
            <span>memory limit per test:&emsp;</span>{{problem.memory_limit}}MB
          </div>
          <div>
            <span>IO Mode:&emsp;</span>{{problem.io_mode.io_mode}}
          </div>
        </div>
        <div id="problem-content" class="markdown-body" v-katex >
          <p class="title">{{$t('m.Description')}}</p>
          <p class="content" v-html=problem.description style="margin-bottom: 0;"></p>
          <!-- {{$t('m.music')}} -->
          <p class="title">{{$t('m.Input')}} <span v-if="problem.io_mode.io_mode=='File IO'">({{$t('m.FromFile')}}: {{ problem.io_mode.input }})</span></p>
          <p class="content" v-html=problem.input_description></p>

          <p class="title">{{$t('m.Output')}} <span v-if="problem.io_mode.io_mode=='File IO'">({{$t('m.ToFile')}}: {{ problem.io_mode.output }})</span></p>
          <p class="content" v-html=problem.output_description></p>

          <div v-for="(sample, index) of problem.samples" :key="index">
            <div class="flex-container sample">
              <div class="sample-input">
                <p class="title">{{$t('m.Sample_Input')}} {{index + 1}}
                  <a class="copy"
                     v-clipboard:copy="sample.input"
                     v-clipboard:success="onCopy"
                     v-clipboard:error="onCopyError">
                    <Icon type="clipboard"></Icon>
                  </a>
                </p>
                <pre>{{sample.input}}</pre>
              </div>
              <div class="sample-output">
                <p class="title">{{$t('m.Sample_Output')}} {{index + 1}}
                  <a class="copy"
                     v-clipboard:copy="sample.input"
                     v-clipboard:success="onCopy"
                     v-clipboard:error="onCopyError">
                    <Icon type="clipboard"></Icon>
                  </a>
                </p>
                <pre>{{sample.output}}</pre>
              </div>
            </div>
          </div>

          <div v-if="problem.hint">
            <p class="title">{{$t('m.Hint')}}</p>
            <Card dis-hover>
              <div class="content" v-html=problem.hint></div>
            </Card>
          </div>

          <div v-if="problem.source">
            <p class="title">{{$t('m.Source')}}</p>
            <p class="content">{{problem.source}}</p>
          </div>

        </div>
      </Panel>
      <!--problem main end-->
      <Card :padding="20" id="submit-code" dis-hover>
        <CodeMirror :value.sync="code"
                    :languages="problem.languages"
                    :language="language"
                    :theme="theme"
                    @resetCode="onResetToTemplate"
                    @changeTheme="onChangeTheme"
                    @changeLang="onChangeLang"></CodeMirror>
        <Row type="flex" justify="space-between">
          <Col :span="10">
            <!-- <div class="status" v-if="statusVisible">
              <template v-if="!this.contestID || (this.contestID && OIContestRealTimePermission)">
                <span>{{$t('m.Status')}}</span>
                <Tag type="dot" :color="submissionStatus.color" @click.native="handleRoute('/status/'+submissionId)">
                {{ $t('m.' + submissionStatus.text.replace(/ /g, "_")) }}
                </Tag>
              </template>
              <template v-else-if="this.contestID && !OIContestRealTimePermission">
                <Alert type="success" show-icon>{{$t('m.Submitted_successfully')}}</Alert>
              </template>
            </div> -->
            <!-- <div v-else-if="problem.my_status === 0">
              <Alert type="success" show-icon>{{$t('m.You_have_solved_the_problem')}}</Alert>
            </div>
            <div v-else-if="this.contestID && !OIContestRealTimePermission && submissionExists">
              <Alert type="success" show-icon>{{$t('m.You_have_submitted_a_solution')}}</Alert>
            </div> -->
            <div v-if="contestEnded">
              <Alert type="warning" show-icon>{{$t('m.Contest_has_ended')}}</Alert>
            </div>  
          </Col>

          <Col :span="12">
            <template v-if="captchaRequired">
              <div class="captcha-container">
                <Tooltip v-if="captchaRequired" content="Click to refresh" placement="top">
                  <img :src="captchaSrc" @click="getCaptchaSrc"/>
                </Tooltip>
                <Input v-model="captchaCode" class="captcha-code"/>
              </div>
            </template>
            <Button type="warning" icon="edit" :loading="submitting" @click="submitCode"
                    :disabled="problemSubmitDisabled || submitted"
                    class="fl-right">
              <span v-if="submitting">{{$t('m.Submitting')}}</span>
              <span v-else>{{$t('m.Submit')}}</span>
            </Button>
          </Col>
        </Row>
      </Card>

      <Card>
        <div class="status" v-if="statusVisible">
          <template v-if="!this.contestID || (this.contestID && OIContestRealTimePermission)">
            <span>{{$t('m.Status')}}</span>
            <Tag type="dot" :color="submissionStatus.color" @click.native="handleRoute('/status/'+submissionId)">
            {{ $t('m.' + submissionStatus.text.replace(/ /g, "_")) }}
            </Tag>
            <div>
              <div v-for="sr in showResult().out">
                <span>{{ sr[0] }}</span>
                <pre class="outshown">{{ sr[1] }}</pre>
              </div>
            </div>
          </template>
          <template v-else-if="this.contestID && !OIContestRealTimePermission">
            <Alert type="success" show-icon>{{$t('m.Submitted_successfully')}}</Alert>
          </template>
        </div>
      </Card>
    </div>

    <div id="right-column">
      <div>
        <VerticalMenu @on-click="handleRoute">
          <template v-if="this.contestID">
            <VerticalMenu-item :route="{name: 'contest-problem-list', params: {contestID: contestID}}">
              <Icon type="ios-photos"></Icon>
              {{$t('m.Problems')}}
            </VerticalMenu-item>

            <VerticalMenu-item :route="{name: 'contest-announcement-list', params: {contestID: contestID}}">
              <Icon type="chatbubble-working"></Icon>
              {{$t('m.Announcements')}}
            </VerticalMenu-item>
          </template>

          <VerticalMenu-item v-if="!this.contestID || OIContestRealTimePermission" :route="submissionRoute">
            <Icon type="navicon-round"></Icon>
            {{$t('m.Submissions')}}
          </VerticalMenu-item>

          <template v-if="this.contestID">
            <VerticalMenu-item v-if="!this.contestID || OIContestRealTimePermission"
                              :route="{name: 'contest-rank', params: {contestID: contestID}}">
              <Icon type="stats-bars"></Icon>
              {{$t('m.Rankings')}}
            </VerticalMenu-item>
            <VerticalMenu-item :route="{name: 'contest-details', params: {contestID: contestID}}">
              <Icon type="home"></Icon>
              {{$t('m.View_Contest')}}
            </VerticalMenu-item>
          </template>
        </VerticalMenu>

        <Card id="info">
          <div slot="title" class="header">
            <Icon type="information-circled"></Icon>
            <span class="card-title">{{$t('m.Information')}}</span>
          </div>
          <ul>
            <!-- <li><p>ID</p>
              <p>{{problem._id}}</p></li>
            <li>
              <p>{{$t('m.Time_Limit')}}</p>
              <p>{{problem.time_limit}}MS</p></li>
            <li>
              <p>{{$t('m.Memory_Limit')}}</p>
              <p>{{problem.memory_limit}}MB</p></li>
            <li>
              <p>{{$t('m.IOMode')}}</p>
              <p>{{problem.io_mode.io_mode}}</p>
            </li>
            <li>
              <p>{{$t('m.Created')}}</p>
              <p>{{problem.created_by.username}}</p></li> -->
            <li v-if="problem.difficulty && !this.contestID">
              <p>{{$t('m.Level')}}</p>
              <p>{{$t('m.' + problem.difficulty)}}</p></li>
            <!-- <li v-if="problem.total_score">
              <p>{{$t('m.Score')}}</p>
              <p>{{problem.total_score}}</p>
            </li> -->
            <li>
              <p>{{$t('m.Tags')}}</p>
              <p>
                <Poptip trigger="hover" placement="left-end">
                  <a>{{$t('m.Show')}}</a>
                  <div slot="content" v-if="!this.contestID">
                    <Tag v-for="tag in problem.tags" :key="tag">{{tag}}</Tag>
                  </div>
                  <div slot="content" v-else>
                    <Tag :key="tag">None</Tag>
                  </div>
                </Poptip>
              </p>
            </li>
          </ul>
        </Card>

        <Card id="pieChart" :padding="0" v-if="!this.contestID || OIContestRealTimePermission">
          <div slot="title">
            <Icon type="ios-analytics"></Icon>
            <span class="card-title">{{$t('m.Statistic')}}</span>
            <Button type="ghost" size="small" id="detail" @click="graphVisible = !graphVisible">Details</Button>
          </div>
          <div style="display: flex; justify-content: center; align-items: center;">
            <div class="echarts" >
              <ECharts :options="pie"></ECharts>
            </div>
          </div>
        </Card>
      </div>

      <div id="right-bottom">
        <Card id="SubmissionStatus">
          <div>
            <Icon type="ios-analytics"></Icon>
            <span style="margin-left: 8px;">{{ $t('m.subStatus') }}</span>
          </div>
          <div v-if="historicSubmit.length > 0">
            <table class="rtable">
              <tr>
                <th>When</th>
                <th>Lang</th>
                <th>Verdict</th>
              </tr>
              <tr v-for="Sub in historicSubmit">
                <td style="text-align: center; font-size: small;">
                  <span>{{ Sub.time[0] }}</span>
                  <span>{{ Sub.time[1] }}</span>
                </td>
                <td style="text-align: center;">{{ Sub.lang }}</td>
                <td style="text-align: center;">
                  <Tag @click.native="handleRoute('/status/'+Sub.id)" :color="SingleStatus(Sub.result).color">
                    <!-- {{$t('m.' + SingleStatus(Sub.result).name.replace(/ /g, '_'))}} -->
                    {{ SingleStatus(Sub.result).text }}
                  </Tag>
                </td>
              </tr>
            </table>
            <Pagination :total="historicSubmitTotal" :page-size="historicSubmitLimit" @on-change="getSubmissions" :current.sync="historicSubmitPage"></Pagination>
          </div>
          <!-- <div id="testsStatus" v-if="statusVisible" style="margin-top: 12px; overflow-y: auto;">
            <div v-if="SubmissionTest !== undefined && SubmissionTest.length !== 0">
              <table border="1" align="center">
                <tr>
                  <th>TestId</th>
                  <th>Status</th>
                </tr>
                <tr v-for="Status in SubmissionTest">
                  <td style="text-align: center;">{{ Status.test_case }}</td>
                  <td style="text-align: center;" :style="{'color': SingleStatus(Status.result).color}">
                    {{$t('m.' + SingleStatus(Status.result).text.replace(/ /g, '_'))}}
                  </td>
                </tr>
              </table>
            </div>
          </div> -->
        </Card>
      </div>
    </div>


    <Modal v-model="graphVisible">
      <div id="pieChart-detail">
        <ECharts :options="largePie" :initOptions="largePieInitOpts"></ECharts>
      </div>
      <div slot="footer">
        <Button type="ghost" @click="graphVisible=false">{{$t('m.Close')}}</Button>
      </div>
    </Modal>

    <Modal v-model="resultDetail">
      <div id="pieChart-detail">
        <ECharts :options="largePie" :initOptions="largePieInitOpts"></ECharts>
      </div>
      <div slot="footer">
        <Button type="ghost" @click="graphVisible=false">{{$t('m.Close')}}</Button>
      </div>
    </Modal>

  </div>
</template>

<script>
  import {mapGetters, mapActions} from 'vuex'
  import {types} from '../../../../store'
  import CodeMirror from '@oj/components/CodeMirror.vue'
  import storage from '@/utils/storage'
  import {FormMixin} from '@oj/components/mixins'
  import {JUDGE_STATUS, CONTEST_STATUS, buildProblemCodeKey} from '@/utils/constants'
  import api from '@oj/api'
  import {pie, largePie} from './chartData'
  import Pagination from '@/pages/oj/components/Pagination'

  // 只显示这些状态的图形占用
  const filtedStatus = ['-1', '-2', '0', '1', '2', '3', '4', '8']

  export default {
    name: 'Problem',
    components: {
      CodeMirror,
      Pagination
    },
    mixins: [FormMixin],
    data () {
      return {
        currentTop: 0,
        historicSubmitLimit: 10,
        historicSubmitTotal: 0,
        historicSubmitPage: 1,
        historicSubmit: [],
        resultDetail: false,
        SubmissionTest: [],
        statusVisible: false,
        captchaRequired: false,
        graphVisible: false,
        submissionExists: false,
        captchaCode: '',
        captchaSrc: '',
        contestID: '',
        problemID: '',
        submitting: false,
        code: '',
        language: 'C++',
        theme: 'solarized',
        submissionId: '',
        submitted: false,
        result: {
          result: 9
        },
        problem: {
          title: '',
          description: '',
          hint: '',
          my_status: '',
          template: {},
          languages: [],
          created_by: {
            username: ''
          },
          tags: [],
          io_mode: {'io_mode': 'Standard IO'}
        },
        pie: pie,
        largePie: largePie,
        // echarts 无法获取隐藏dom的大小，需手动指定
        largePieInitOpts: {
          width: '500',
          height: '480'
        }
      }
    },
    beforeRouteEnter (to, from, next) {
      let problemCode = storage.get(buildProblemCodeKey(to.params.problemID, to.params.contestID))
      if (problemCode) {
        next(vm => {
          vm.language = problemCode.language
          vm.code = problemCode.code
          vm.theme = problemCode.theme
        })
      } else {
        next()
      }
    },
    mounted () {
      this.$store.commit(types.CHANGE_CONTEST_ITEM_VISIBLE, {menu: false})
      this.init()
      let his = document.querySelector('#right-bottom')
      this.currentTop = his.getBoundingClientRect().top
      window.addEventListener('scroll', this.handleScroll)
    },
    methods: {
      ...mapActions(['changeDomTitle']),
      handleScroll () {
        let his = document.querySelector('#right-bottom')
        let wt = window.scrollY
        if (this.currentTop <= wt) {
          his.style.marginTop = wt - this.currentTop + 100 + 'px'
        } else {
          his.style.marginTop = ''
        }
      },
      init () {
        this.$Loading.start()
        this.contestID = this.$route.params.contestID
        this.problemID = this.$route.params.problemID
        let func = this.$route.name === 'problem-details' ? 'getProblem' : 'getContestProblem'
        api[func](this.problemID, this.contestID).then(res => {
          this.$Loading.finish()
          let problem = res.data.data
          this.changeDomTitle({title: problem.title})
          api.submissionExists(problem.id).then(res => {
            this.submissionExists = res.data.data
          })
          problem.languages = problem.languages.sort()
          this.problem = problem
          if (problem.statistic_info) {
            this.changePie(problem)
          }

          // 在beforeRouteEnter中修改了, 说明本地有code，无需加载template
          if (this.code !== '') {
            return
          }
          // try to load problem template
          this.language = this.problem.languages[0]
          let template = this.problem.template
          if (template && template[this.language]) {
            this.code = template[this.language]
          }
        }, () => {
          this.$Loading.error()
        })
        this.getSubmissions()
      },
      changePie (problemData) {
        // 只显示特定的一些状态
        for (let k in problemData.statistic_info) {
          if (filtedStatus.indexOf(k) === -1) {
            delete problemData.statistic_info[k]
          }
        }
        let acNum = problemData.accepted_number
        let data = [
          {name: 'WA', value: problemData.submission_number - acNum},
          {name: 'AC', value: acNum}
        ]
        this.pie.series[0].data = data
        // 只把大图的AC selected下，这里需要做一下deepcopy
        let data2 = JSON.parse(JSON.stringify(data))
        data2[1].selected = true
        this.largePie.series[1].data = data2

        // 根据结果设置legend,没有提交过的legend不显示
        let legend = Object.keys(problemData.statistic_info).map(ele => JUDGE_STATUS[ele].short)
        if (legend.length === 0) {
          legend.push('AC', 'WA')
        }
        this.largePie.legend.data = legend

        // 把ac的数据提取出来放在最后
        let acCount = problemData.statistic_info['0']
        delete problemData.statistic_info['0']

        let largePieData = []
        Object.keys(problemData.statistic_info).forEach(ele => {
          largePieData.push({name: JUDGE_STATUS[ele].short, value: problemData.statistic_info[ele]})
        })
        largePieData.push({name: 'AC', value: acCount})
        this.largePie.series[0].data = largePieData
      },
      handleRoute (route) {
        this.$router.push(route)
      },
      onChangeLang (newLang) {
        if (this.problem.template[newLang]) {
          if (this.code.trim() === '') {
            this.code = this.problem.template[newLang]
          }
        }
        this.language = newLang
      },
      onChangeTheme (newTheme) {
        this.theme = newTheme
      },
      onResetToTemplate () {
        this.$Modal.confirm({
          content: this.$i18n.t('m.Are_you_sure_you_want_to_reset_your_code'),
          onOk: () => {
            let template = this.problem.template
            if (template && template[this.language]) {
              this.code = template[this.language]
            } else {
              this.code = ''
            }
          }
        })
      },
      checkSubmissionStatus () {
        // 使用setTimeout避免一些问题
        if (this.refreshStatus) {
          // 如果之前的提交状态检查还没有停止,则停止,否则将会失去timeout的引用造成无限请求
          clearTimeout(this.refreshStatus)
        }
        const checkStatus = () => {
          let id = this.submissionId
          api.getSubmission(id).then(res => {
            this.result = res.data.data
            // console.log(this.result)
            this.SubmissionTest = this.result.info.data
            if (Object.keys(res.data.data.statistic_info).length !== 0) {
              this.submitting = false
              this.submitted = false
              clearTimeout(this.refreshStatus)
              this.init()
            } else {
              this.refreshStatus = setTimeout(checkStatus, 2000)
            }
          }, res => {
            this.submitting = false
            clearTimeout(this.refreshStatus)
          })
        }
        this.refreshStatus = setTimeout(checkStatus, 2000)
      },
      submitCode () {
        if (this.code.trim() === '') {
          this.$error(this.$i18n.t('m.Code_can_not_be_empty'))
          return
        }
        this.submissionId = ''
        this.result = {result: 9}
        this.submitting = true
        let data = {
          problem_id: this.problem.id,
          language: this.language,
          code: this.code,
          contest_id: this.contestID
        }
        if (this.captchaRequired) {
          data.captcha = this.captchaCode
        }
        const submitFunc = (data, detailsVisible) => {
          this.statusVisible = true
          api.submitCode(data).then(res => {
            this.submissionId = res.data.data && res.data.data.submission_id
            // 定时检查状态
            this.submitting = false
            this.submissionExists = true
            if (!detailsVisible) {
              this.$Modal.success({
                title: this.$i18n.t('m.Success'),
                content: this.$i18n.t('m.Submit_code_successfully')
              })
              return
            }
            this.submitted = true
            this.checkSubmissionStatus()
          }, res => {
            this.getCaptchaSrc()
            if (res.data.data.startsWith('Captcha is required')) {
              this.captchaRequired = true
            }
            this.submitting = false
            this.statusVisible = false
          })
        }

        if (this.contestRuleType === 'OI' && !this.OIContestRealTimePermission) {
          if (this.submissionExists) {
            this.$Modal.confirm({
              title: '',
              content: '<h3>' + this.$i18n.t('m.You_have_submission_in_this_problem_sure_to_cover_it') + '<h3>',
              onOk: () => {
                // 暂时解决对话框与后面提示对话框冲突的问题(否则一闪而过）
                setTimeout(() => {
                  submitFunc(data, false)
                }, 1000)
              },
              onCancel: () => {
                this.submitting = false
              }
            })
          } else {
            submitFunc(data, false)
          }
        } else {
          submitFunc(data, true)
        }
      },
      onCopy (event) {
        this.$success('Code copied')
      },
      onCopyError (e) {
        this.$error('Failed to copy code')
      },
      SingleStatus (e) {
        return {
          text: JUDGE_STATUS[e]['short'],
          color: JUDGE_STATUS[e]['color'],
          name: JUDGE_STATUS[e]['name']
        }
      },
      getSubmissions () {
        let params = {
          myself: '1',
          result: '',
          username: '',
          page: 1,
          contest_id: this.contestID,
          problem_id: this.problemID
        }
        let offset = (this.historicSubmitPage - 1) * 10
        let func = this.contestID ? 'getContestSubmissionList' : 'getSubmissionList'
        api[func](offset, 10, params).then(res => {
          let data = res.data.data
          this.historicSubmitTotal = data.total
          this.historicSubmit = []
          for (let _ in data.results) {
            this.historicSubmit.push({
              time: data.results[_].create_time.split('.')[0].split('T'),
              id: data.results[_].id,
              lang: data.results[_].language,
              result: data.results[_].result
            })
          }
        })
      },
      showResult () {
        if (this.result.result === -2) {
          return {
            total: 1,
            out: [
              ['SystemOut', this.result.statistic_info.err_info]
            ]
          }
        } else if (!['5', '6', '7', '9'].includes(this.result.result)) {
          if (this.result.result === 0) {
            return {
              total: 1,
              out: [
                ['SystemOut', 'AC']
              ]
            }
          } else {
            for (let st of this.SubmissionTest) {
              if (st.result !== 0) {
                return {
                  total: 3,
                  out: [
                    ['SystemIn', this.contestID ? 'None' : st.input_sys],
                    ['SystemOut', this.contestID ? 'None' : st.output_sys],
                    ['UserOut', this.contestID ? 'None' : st.output_user]
                  ]
                }
              }
            }
          }
        }
        return {
          total: 1,
          out: [
            ['SystemOut', 'None']
          ]
        }
      }
    },
    computed: {
      ...mapGetters(['problemSubmitDisabled', 'contestRuleType', 'OIContestRealTimePermission', 'contestStatus']),
      contest () {
        return this.$store.state.contest.contest
      },
      contestEnded () {
        return this.contestStatus === CONTEST_STATUS.ENDED
      },
      submissionStatus () {
        return {
          text: JUDGE_STATUS[this.result.result]['name'],
          color: JUDGE_STATUS[this.result.result]['color']
        }
      },
      submissionRoute () {
        if (this.contestID) {
          return {name: 'contest-submission-list', query: {problemID: this.problemID}}
        } else {
          return {name: 'submission-list', query: {problemID: this.problemID}}
        }
      }
    },
    beforeRouteLeave (to, from, next) {
      // 防止切换组件后仍然不断请求
      clearInterval(this.refreshStatus)

      this.$store.commit(types.CHANGE_CONTEST_ITEM_VISIBLE, {menu: true})
      storage.set(buildProblemCodeKey(this.problem._id, from.params.contestID), {
        code: this.code,
        language: this.language,
        theme: this.theme
      })
      next()
    },
    watch: {
      '$route' () {
        this.init()
      }
    }
  }
</script>

<style lang="less" scoped>
  .card-title {
    margin-left: 8px;
  }

  .flex-container {
    #problem-main {
      flex: auto;
      margin-left: 100px;
      margin-right: 18px;
    }
    #right-column {
      flex: none;
      width: 300px;
      margin-right: 100px;
    }
  }

  #problem-content {
    margin-top: 0px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    .title {
      font-size: 20px;
      color: #3091f2;
      .copy {
        padding-left: 8px;
      }
    }
    p.content {
      text-align: center;
      margin-left: 25px;
      margin-right: 20px;
      font-size: 15px;
    }
    .sample {
      align-items: stretch;
      &-input {
        width: 50%;
        flex: 1 1 auto;
        display: flex;
        flex-direction: column;
        margin-right: 5%;
      }
      &-output {
        width: 50%;
        flex: 1 1 auto;
        display: flex;
        flex-direction: column;
      }
      pre {
        flex: 1 1 auto;
        align-self: stretch;
        border-style: solid;
        background: transparent;
      }
    }
  }

  #submit-code {
    margin-top: 20px;
    margin-bottom: 20px;
    .status {
      float: left;
      span {
        margin-right: 10px;
        margin-left: 10px;
      }
    }
    .captcha-container {
      display: inline-block;
      .captcha-code {
        width: auto;
        margin-top: -20px;
        margin-left: 20px;
      }
    }
  }

  #info {
    margin-bottom: 20px;
    margin-top: 20px;
    ul {
      list-style-type: none;
      li {
        border-bottom: 1px dotted #e9eaec;
        margin-bottom: 10px;
        p {
          display: inline-block;
        }
        p:first-child {
          width: 90px;
        }
        p:last-child {
          float: right;
        }
      }
    }
  }

  .fl-right {
    float: right;
  }

  #pieChart {
    .echarts {
      height: 250px;
      width: 210px;
    }
    #detail {
      position: absolute;
      right: 10px;
      top: 10px;
    }
  }

  #pieChart-detail {
    margin-top: 20px;
    width: 500px;
    height: 480px;
  }

  #SubmissionStatus {
    top: 20px;
  }

  #submitting {
    display: inline-block;
    border: 10px solid rgba(0, 0, 0, 0.1);
    border-left-color: #59a782;
    border-radius: 50%;
    width: 60px;
    height: 60px;
    animation: Submitting 1.2s linear infinite;
  }

  @keyframes Submitting {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }

  .rtable {
    border-collapse: collapse;
  }

  .rtable tr th{
    border: 1px solid black
  }
  .rtable tr td {
    border: 1px solid black
  }

  .outshown {
    border-radius: 5px;
    border: 1px solid black;
    box-shadow: inset 0 0 5px rgb(94, 93, 93);
    min-height: 80px;
    padding-left: 8px;
    padding-right: 8px;
    font-size: 16px;
  }

</style>

