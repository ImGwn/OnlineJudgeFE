<template>
  <div>
    <Panel>
      <div slot="title">
        {{$t('m.Problems_List')}}
      </div>
      <div slot="extra">
        <Input id="keyword" @on-enter="changeRoute" @on-click="changeRoute" v-model="keyword"
                   icon="ios-search-strong" placeholder="Keyword"/>
      </div>
      <Table v-if="contestRuleType == 'ACM' || OIContestRealTimePermission"
             :columns="ACMTableColumns"
             :data="problemListAll"
             @on-row-click="goContestProblem"
             :no-data-text="$t('m.No_Problems')"></Table>
      <Table v-else
             :data="problems"
             :columns="OITableColumns"
             @on-row-click="goContestProblem"
             no-data-text="$t('m.No_Problems')"></Table>
    </Panel>
  </div>

</template>

<script>
  import {mapState, mapGetters} from 'vuex'
  import {ProblemMixin} from '@oj/components/mixins'
  // added
  // import api from '@oj/api'
  // import utils from '@/utils/utils'

  export default {
    name: 'ContestProblemList',
    mixins: [ProblemMixin],
    data () {
      return {
        ACMTableColumns: [
          {
            title: '#',
            key: '_id',
            sortType: 'asc',
            width: 150
          },
          {
            title: this.$i18n.t('m.Title'),
            key: 'title'
          },
          {
            title: this.$i18n.t('m.Tag'),
            // key: 'tags'
            align: 'center',
            render: (h, params) => {
              let tags = []
              params.row.tags.forEach(tag => {
                tags.push(h('Tag', {}, tag))
              })
              return h('div', {
                style: {
                  margin: '8px 0'
                }
              }, tags)
            }
          },
          {
            title: this.$i18n.t('m.Total'),
            key: 'submission_number'
          },
          {
            title: this.$i18n.t('m.AC_Rate'),
            render: (h, params) => {
              if (params.row.isTitle) {
                return h('span', {
                  props: {
                    color: 'white'
                  }
                })
              }
              return h('span', this.getACRate(params.row.accepted_number, params.row.submission_number))
            }
          }
        ],
        OITableColumns: [
          {
            title: '#',
            key: '_id',
            width: 150
          },
          {
            title: this.$i18n.t('m.Title'),
            key: 'title'
          }
        ],
        // added
        problemListAll: [],
        tagList: [],
        keyword: '',
        tempList: []
      }
    },
    mounted () {
      this.getContestProblems()
    },
    methods: {
      getContestProblems () {
        this.$store.dispatch('getContestProblems').then(res => {
          if (this.isAuthenticated) {
            this.problemListAll = res.data.data
            // this.getTagList(res.data.data)
            if (this.contestRuleType === 'ACM') {
              this.addStatusColumn(this.ACMTableColumns, res.data.data)
            } else if (this.OIContestRealTimePermission) {
              this.addStatusColumn(this.ACMTableColumns, res.data.data)
            }
            // added
            console.log('this.ACMTableColumns')
            console.log(res.data.data)
          }
        })
      },
      goContestProblem (row) {
        this.$router.push({
          name: 'contest-problem-details',
          params: {
            contestID: this.$route.params.contestID,
            problemID: row._id
          }
        })
      },
      // added
      // pushRouter () {
      //   this.$router.push({
      //     name: 'contest-problem-details',
      //     query: utils.filterEmptyValue(this.query)
      //   })
      // },
      // divideByTags (tags, problems) {
      //   console.log('problems')
      //   console.log(problems)
      //   console.log(this.problemListAll)
      //   for (let i = 0; i < tags.length; i++) {
      //     this.problemListAll.push({
      //       'title': tags[i],
      //       'isTitle': true
      //     })
      //     for (let j = 0; j < problems.length; j++) {
      //       if (problems[j].tags[0] === tags[i]) {
      //         this.problemListAll.push(problems[j])
      //       }
      //     }
      //   }
      // },
      changeRoute () {
        console.log(this.keyword)
        if (this.keyword !== '') {
          for (let i = 0; i < this.problemListAll.length; i++) {
            // if (this.problemListAll[i].tags.indexOf(this.keyword) !== -1) {
            //   console.log(this.problemListAll[i].tags)
            //   this.tempList.push(this.problemListAll[i])
            // }
            for (let j = 0; j < this.problemListAll[i].tags.length; j++) {
              if (this.problemListAll[i].tags[j].indexOf(this.keyword) !== -1) {
                console.log(this.problemListAll[i].tags)
                this.tempList.push(this.problemListAll[i])
              }
            }
          }
        }
        // this.ACMTableColumns.splice(0, this.ACMTableColumns.length)
        this.problemListAll.splice(0, this.problemListAll.length)
        this.problemListAll = this.tempList
        console.log(this.tempList)
        console.log(this.problemListAll)
      },
      getTagList (problems) {
        for (let i = 0; i < problems.length; i++) {
          for (let j = 0; j < problems[i]['tags'].length; j++) {
            if (this.tagList.indexOf(problems[i]['tags'][j]) === -1) {
              this.tagList.push(problems[i]['tags'][j])
            }
          }
        }
        console.log(this.tagList)
      }
    },
    computed: {
      ...mapState({
        problems: state => state.contest.contestProblems
      }),
      ...mapGetters(['isAuthenticated', 'contestRuleType', 'OIContestRealTimePermission'])
    }
  }
</script>

<style scoped lang="less">
</style>
