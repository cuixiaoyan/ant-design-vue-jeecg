<template>
  <div>
    <v-chart :force-fit="true" :height="height" :padding="padding" :data="data" :scale="scale">
      <v-coord type="polar" />
      <v-axis :data-key="axis1Opts.dataKey" :label="axis1Opts.label" :tickLine="axis1Opts.tickLine" :line="axis1Opts.line" />
      <v-axis :data-key="axis2Opts.dataKey" :grid="axis2Opts.grid" :tickLine="axis2Opts.tickLine" :label="axis2Opts.label" />
      <v-legend dataKey="country" :itemWidth="50" />
      <v-interval :position="interval1Opts.position" :color="interval1Opts.color"
                  :label="interval1Opts.label" :v-style="interval1Opts.style" />
    </v-chart>
  </div>
</template>

<script>
  const data = [{country:'中国',cost:96},{country:'德国',cost:121},{country:'美国',cost:100},{country:'日本',cost:111},{country:'韩国',cost:102},{country:'法国',cost:124},{country:'意大利',cost:123},{country:'荷兰',cost:111},{country:'比利时',cost:123},{country:'英国',cost:109},{country:'加拿大',cost:115},{country:'俄罗斯',cost:99},{country:'墨西哥',cost:91},{country:'印度',cost:87},{country:'瑞士',cost:125},{country:'澳大利亚',cost:130},{country:'西班牙',cost:109},{country:'巴西',cost:123},{country:'泰国',cost:91},{country:'印尼',cost:83},{country:'波兰',cost:101},{country:'瑞典',cost:116},{country:'奥地利',cost:111},{country:'捷克',cost:107}];

  const scale = [{
    dataKey: 'cost',
    min: 0,
  }];

  const axis1Opts = {
    dataKey: 'cost',
    label: null,
    tickLine: null,
    line: {
      stroke: '#E9E9E9',
      lineDash: [ 3, 3 ]
    }
  };
  const axis2Opts = {
    dataKey: 'country',
    grid: {
      align: 'center'
    },
    tickLine: null,
    label: {
      Offset: 10,
      textStyle: {
        textAlign: 'center' // 设置坐标轴 label 的文本对齐方向
      }
    }
  };

  const interval1Opts = {
    position: 'country*cost',
    color: 'country',
    label: ['cost', {
      offset: -15,
      textStyle: {
        textAlign: 'center',
        fontSize: 11,
        shadowBlur: 2,
        shadowColor: 'rgba(0, 0, 0, .45)'
      }
    }],
    style: {
      lineWidth: 1,
      stroke: '#fff',
    },
  };

  export default {
    data() {
      return {
        height: 400,
        padding:[ 40, 40, 130, 40 ],
        data,
        scale,
        axis1Opts,
        axis2Opts,
        interval1Opts,
      };
    }
  };
</script>



<!--<template>-->
<!--  <div class="page-header-index-wide">-->
<!--    <a-row :gutter="24">-->
<!--      <a-col :sm="24" :md="12" :xl="6" :style="{ marginBottom: '24px' }">-->
<!--        <chart-card :loading="loading" title="总销售额" total="￥126,560">-->
<!--          <a-tooltip title="指标说明" slot="action">-->
<!--            <a-icon type="info-circle-o" />-->
<!--          </a-tooltip>-->
<!--          <div>-->
<!--            <trend flag="up" style="margin-right: 16px;">-->
<!--              <span slot="term">周同比</span>-->
<!--              12%-->
<!--            </trend>-->
<!--            <trend flag="down">-->
<!--              <span slot="term">日同比</span>-->
<!--              11%-->
<!--            </trend>-->
<!--          </div>-->
<!--          <template slot="footer">日均销售额<span>￥ 234.56</span></template>-->
<!--        </chart-card>-->
<!--      </a-col>-->
<!--      <a-col :sm="24" :md="12" :xl="6" :style="{ marginBottom: '24px' }">-->
<!--        <chart-card :loading="loading" title="访问量" :total="8846 | NumberFormat">-->
<!--          <a-tooltip title="指标说明" slot="action">-->
<!--            <a-icon type="info-circle-o" />-->
<!--          </a-tooltip>-->
<!--          <div>-->
<!--            <mini-area />-->
<!--          </div>-->
<!--          <template slot="footer">日访问量<span> {{ '1234' | NumberFormat }}</span></template>-->
<!--        </chart-card>-->
<!--      </a-col>-->
<!--      <a-col :sm="24" :md="12" :xl="6" :style="{ marginBottom: '24px' }">-->
<!--        <chart-card :loading="loading" title="支付笔数" :total="6560 | NumberFormat">-->
<!--          <a-tooltip title="指标说明" slot="action">-->
<!--            <a-icon type="info-circle-o" />-->
<!--          </a-tooltip>-->
<!--          <div>-->
<!--            <mini-bar :height="40" />-->
<!--          </div>-->
<!--          <template slot="footer">转化率 <span>60%</span></template>-->
<!--        </chart-card>-->
<!--      </a-col>-->
<!--      <a-col :sm="24" :md="12" :xl="6" :style="{ marginBottom: '24px' }">-->
<!--        <chart-card :loading="loading" title="运营活动效果" total="78%">-->
<!--          <a-tooltip title="指标说明" slot="action">-->
<!--            <a-icon type="info-circle-o" />-->
<!--          </a-tooltip>-->
<!--          <div>-->
<!--            <mini-progress color="rgb(19, 194, 194)" :target="80" :percentage="78" :height="8" />-->
<!--          </div>-->
<!--          <template slot="footer">-->
<!--            <trend flag="down" style="margin-right: 16px;">-->
<!--              <span slot="term">同周比</span>-->
<!--              12%-->
<!--            </trend>-->
<!--            <trend flag="up">-->
<!--              <span slot="term">日环比</span>-->
<!--              80%-->
<!--            </trend>-->
<!--          </template>-->
<!--        </chart-card>-->
<!--      </a-col>-->
<!--    </a-row>-->

<!--    <a-card :loading="loading" :bordered="false" :body-style="{padding: '0'}">-->
<!--      <div class="salesCard">-->
<!--        <a-tabs default-active-key="1" size="large" :tab-bar-style="{marginBottom: '24px', paddingLeft: '16px'}">-->
<!--          <div class="extra-wrapper" slot="tabBarExtraContent">-->
<!--            <div class="extra-item">-->
<!--              <a>今日</a>-->
<!--              <a>本周</a>-->
<!--              <a>本月</a>-->
<!--              <a>本年</a>-->
<!--            </div>-->
<!--            <a-range-picker :style="{width: '256px'}" />-->
<!--          </div>-->
<!--          <a-tab-pane loading="true" tab="销售额" key="1">-->
<!--            <a-row>-->
<!--              <a-col :xl="16" :lg="12" :md="12" :sm="24" :xs="24">-->
<!--                <bar title="销售额排行" :dataSource="barData"/>-->
<!--              </a-col>-->
<!--              <a-col :xl="8" :lg="12" :md="12" :sm="24" :xs="24">-->
<!--                <rank-list title="门店销售排行榜" :list="rankList"/>-->
<!--              </a-col>-->
<!--            </a-row>-->
<!--          </a-tab-pane>-->
<!--          <a-tab-pane tab="访问量" key="2">-->
<!--            <a-row>-->
<!--              <a-col :xl="16" :lg="12" :md="12" :sm="24" :xs="24">-->
<!--                <bar title="销售额趋势" :dataSource="barData"/>-->
<!--              </a-col>-->
<!--              <a-col :xl="8" :lg="12" :md="12" :sm="24" :xs="24">-->
<!--                <rank-list title="门店销售排行榜" :list="rankList"/>-->
<!--              </a-col>-->
<!--            </a-row>-->
<!--          </a-tab-pane>-->
<!--        </a-tabs>-->
<!--      </div>-->
<!--    </a-card>-->

<!--    <a-row>-->
<!--      <a-col :span="24">-->
<!--        <a-card :loading="loading" :bordered="false" title="最近一周访问次数统计" :style="{ marginTop: '24px' }">-->
<!--          <a-row>-->
<!--            <a-col :span="6">-->
<!--              <head-info title="今日访问IP数" :content="loginfo.todayIp"></head-info>-->
<!--            </a-col>-->
<!--            <a-col :span="2">-->
<!--              <a-spin class='circle-cust'>-->
<!--                <a-icon slot="indicator" type="environment" style="font-size: 24px"  />-->
<!--              </a-spin>-->
<!--            </a-col>-->
<!--            <a-col :span="6">-->
<!--              <head-info title="今日访问次数" :content="loginfo.todayVisitCount"></head-info>-->
<!--            </a-col>-->
<!--            <a-col :span="2">-->
<!--              <a-spin class='circle-cust'>-->
<!--                <a-icon slot="indicator" type="team" style="font-size: 24px"  />-->
<!--              </a-spin>-->
<!--            </a-col>-->
<!--            <a-col :span="6">-->
<!--              <head-info title="访问总次数" :content="loginfo.totalVisitCount"></head-info>-->
<!--            </a-col>-->
<!--            <a-col :span="2">-->
<!--              <a-spin class='circle-cust'>-->
<!--                <a-icon slot="indicator" type="rise" style="font-size: 24px"  />-->
<!--              </a-spin>-->
<!--            </a-col>-->
<!--          </a-row>-->
<!--          <line-chart-multid :fields="visitFields" :dataSource="visitInfo"></line-chart-multid>-->
<!--        </a-card>-->
<!--      </a-col>-->
<!--    </a-row>-->
<!--  </div>-->
<!--</template>-->

<!--<script>-->
<!--  import ChartCard from '@/components/ChartCard'-->
<!--  import ACol from "ant-design-vue/es/grid/Col"-->
<!--  import ATooltip from "ant-design-vue/es/tooltip/Tooltip"-->
<!--  import MiniArea from '@/components/chart/MiniArea'-->
<!--  import MiniBar from '@/components/chart/MiniBar'-->
<!--  import MiniProgress from '@/components/chart/MiniProgress'-->
<!--  import RankList from '@/components/chart/RankList'-->
<!--  import Bar from '@/components/chart/Bar'-->
<!--  import LineChartMultid from '@/components/chart/LineChartMultid'-->
<!--  import HeadInfo from '@/components/tools/HeadInfo.vue'-->

<!--  import Trend from '@/components/Trend'-->
<!--  import { getLoginfo,getVisitInfo } from '@/api/api'-->

<!--  const rankList = []-->
<!--  for (let i = 0; i < 7; i++) {-->
<!--    rankList.push({-->
<!--      name: '白鹭岛 ' + (i+1) + ' 号店',-->
<!--      total: 1234.56 - i * 100-->
<!--    })-->
<!--  }-->
<!--  const barData = []-->
<!--  for (let i = 0; i < 12; i += 1) {-->
<!--    barData.push({-->
<!--      x: `${i + 1}月`,-->
<!--      y: Math.floor(Math.random() * 1000) + 200-->
<!--    })-->
<!--  }-->
<!--  export default {-->
<!--    name: "Analysis",-->
<!--    components: {-->
<!--      ATooltip,-->
<!--      ACol,-->
<!--      ChartCard,-->
<!--      MiniArea,-->
<!--      MiniBar,-->
<!--      MiniProgress,-->
<!--      RankList,-->
<!--      Bar,-->
<!--      Trend,-->
<!--      LineChartMultid,-->
<!--      HeadInfo-->
<!--    },-->
<!--    data() {-->
<!--      return {-->
<!--        loading: true,-->
<!--        center: null,-->
<!--        rankList,-->
<!--        barData,-->
<!--        loginfo:{},-->
<!--        visitFields:['ip','visit'],-->
<!--        visitInfo:[],-->
<!--        indicator: <a-icon type="loading" style="font-size: 24px" spin />-->
<!--      }-->
<!--    },-->
<!--    created() {-->
<!--      setTimeout(() => {-->
<!--        this.loading = !this.loading-->
<!--      }, 1000)-->
<!--      this.initLogInfo();-->
<!--    },-->
<!--    methods: {-->
<!--      initLogInfo () {-->
<!--        getLoginfo(null).then((res)=>{-->
<!--          if(res.success){-->
<!--            Object.keys(res.result).forEach(key=>{-->
<!--              res.result[key] =res.result[key]+""-->
<!--            })-->
<!--            this.loginfo = res.result;-->
<!--          }-->
<!--        })-->
<!--        getVisitInfo().then(res=>{-->
<!--          if(res.success){-->
<!--             console.log("aaaaaa",res.result)-->
<!--             this.visitInfo = res.result;-->
<!--           }-->
<!--         })-->
<!--      },-->
<!--    }-->
<!--  }-->
<!--</script>-->

<!--<style lang="scss" scoped>-->
<!--  .circle-cust{-->
<!--    position: relative;-->
<!--    top: 28px;-->
<!--    left: -100%;-->
<!--  }-->
<!--  .extra-wrapper {-->
<!--    line-height: 55px;-->
<!--    padding-right: 24px;-->

<!--    .extra-item {-->
<!--      display: inline-block;-->
<!--      margin-right: 24px;-->

<!--      a {-->
<!--        margin-left: 24px;-->
<!--      }-->
<!--    }-->
<!--  }-->

<!--  /* 首页访问量统计 */-->
<!--  .head-info {-->
<!--    position: relative;-->
<!--    text-align: left;-->
<!--    padding: 0 32px 0 0;-->
<!--    min-width: 125px;-->

<!--    &.center {-->
<!--      text-align: center;-->
<!--      padding: 0 32px;-->
<!--    }-->

<!--    span {-->
<!--      color: rgba(0, 0, 0, .45);-->
<!--      display: inline-block;-->
<!--      font-size: .95rem;-->
<!--      line-height: 42px;-->
<!--      margin-bottom: 4px;-->
<!--    }-->
<!--    p {-->
<!--      line-height: 42px;-->
<!--      margin: 0;-->
<!--      a {-->
<!--        font-weight: 600;-->
<!--        font-size: 1rem;-->
<!--      }-->
<!--    }-->
<!--  }-->
<!--</style>-->