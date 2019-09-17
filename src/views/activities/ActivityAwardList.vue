<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :md="12" :sm="16">
            <a-form-item label="结束时间">
              <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="queryParam.jieshushijian_begin"></j-date>
              <span class="query-group-split-cust"></span>
              <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="queryParam.jieshushijian_end"></j-date>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8" >
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>

            </span>
          </a-col>

          <a-col :md="6" :sm="24">
            <template v-if="superQueryFlag">
              <a-tooltip title="已有高级查询条件生效!">
                <button :disabled="false" class="ant-btn ant-btn-primary" @click="superQuery">
                  <a-icon type="appstore" theme="twoTone" :spin="true" ></a-icon>
                  <span>高级查询</span>
                </button>
              </a-tooltip>
            </template>
            <a-button v-else type="primary" @click="superQuery" icon="filter">高级查询</a-button>

          </a-col>


        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->
    
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('活动奖励')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>

        <span style="float:right;">
          <a @click="loadData()"><a-icon type="sync" />刷新</a>
        </span>

      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">
        
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无此图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="图片不存在" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无此文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="uploadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <activityAward-modal ref="modalForm" @ok="modalFormOk"></activityAward-modal>

    <!-- 高级查询区域 -->
    <j-super-query :fieldList="fieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>

  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import ActivityAwardModal from './modules/ActivityAwardModal'
  import JDate from '@/components/jeecg/JDate.vue'
  import {initDictOptions, filterMultiDictText} from '@/components/dict/JDictSelectUtil'

  //高级查询 区域
  import JSuperQuery from '@/components/jeecg/JSuperQuery.vue';


  //高级查询modal需要参数
  const superQueryFieldList=[{
    type:"string",
    value:"diyijiangpin",
    text:"第一名奖品"
  },{
    type:"string",
    value:"direjiangpin",
    text:"第二名奖品"
  },{
    type:"string",
    value:"disanjiangpin",
    text:"第三名奖品"
  }
  ]


  export default {
    name: "ActivityAwardList",
    mixins:[JeecgListMixin],
    components: {
      JDate,
      ActivityAwardModal,
      JSuperQuery,
    },
    data () {
      return {
        description: '活动奖励管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },

          {
            title:'第一名人数',
            align:"center",
            dataIndex: 'diyirenshu'
          },
          {
            title:'第一名奖品',
            align:"center",
            dataIndex: 'diyijiangpin'
          },

          {
            title:'第二名人数',
            align:"center",
            dataIndex: 'dierrenshu'
          },
          {
            title:'第二名奖品',
            align:"center",
            dataIndex: 'direjiangpin'
          },

          {
            title:'第三名人数',
            align:"center",
            dataIndex: 'disanrenshu'
          },
          {
            title:'第三名奖品',
            align:"center",
            dataIndex: 'disanjiangpin'
          },

          {
            title:'参与人数',
            align:"center",
            dataIndex: 'canyurenshu'
          },
          {
            title:'参与奖品',
            align:"center",
            dataIndex: 'canyujiangpin'
          },
          {
            title:'哪次活动',
            align:"center",
            dataIndex: 'huodong',
            customRender:(text)=>{
              if(!text){
                return ''
              }else{
                return filterMultiDictText(this.dictOptions['huodong'], text+"")
              }
            }
          },
          {
            title:'是否结束',
            align:"center",
            dataIndex: 'zhuangtai',
            customRender:(text)=>{
              if(!text){
                return ''
              }else{
                return filterMultiDictText(this.dictOptions['zhuangtai'], text+"")
              }
            }
          },
          {
            title:'结束时间',
            align:"center",
            dataIndex: 'jieshushijian',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
        url: {
          list: "/activityAward/activityAward/list",
          delete: "/activityAward/activityAward/delete",
          deleteBatch: "/activityAward/activityAward/deleteBatch",
          exportXlsUrl: "/activityAward/activityAward/exportXls",
          importExcelUrl: "activityAward/activityAward/importExcel",
        },

        fieldList:superQueryFieldList,
        dictOptions:{
         zhuangtai:[],
          huodong:[],
        } 
      }
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {
      initDictConfig(){
        initDictOptions('publishingActivities,activities,id').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'huodong', res.result)
          }
        }),

        initDictOptions('jieshu').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'zhuangtai', res.result)
          }
        })
      }
       
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>