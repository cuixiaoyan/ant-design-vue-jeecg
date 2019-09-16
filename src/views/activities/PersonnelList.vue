<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="参与者名字">
              <a-input placeholder="请输入参与者名字" v-model="queryParam.name"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="12" :sm="16">
            <a-form-item label="提醒时间">
              <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="queryParam.remindertime_begin" :showTime="true" dateFormat="YYYY-MM-DD HH:mm:ss"></j-date>
              <span class="query-group-split-cust"></span>
              <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="queryParam.remindertime_end" :showTime="true" dateFormat="YYYY-MM-DD HH:mm:ss"></j-date>
            </a-form-item>
          </a-col>


          <a-col :md="6" :sm="8" >
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>

            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->
    
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('参加活动人员表')">导出</a-button>
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

    <personnel-modal ref="modalForm" @ok="modalFormOk"></personnel-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import PersonnelModal from './modules/PersonnelModal'
  import JDate from '@/components/jeecg/JDate.vue'
  import {initDictOptions, filterMultiDictText} from '@/components/dict/JDictSelectUtil'

  //引入自定义排序
  import { filterObj } from '@/utils/util';

  export default {
    name: "PersonnelList",
    mixins:[JeecgListMixin],
    components: {
      JDate,
      PersonnelModal,
      filterObj
    },
    data () {
      return {
        description: '参加活动人员表管理页面',

        // 表头
        columns: [
          {
            title: '序号',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'用户名称',
            align:"center",
            dataIndex: 'name'
          },
          {
            title:'手机号',
            align:"center",
            dataIndex: 'phone'
          },
          {
            title:'等级',
            align:"center",
            dataIndex: 'member_dictText',
          },
          {
            title:'人数',
            align:"center",
            dataIndex: 'number',
            //自定义筛选
            sorter: true,
          },
          {
            title:'活动',
            align:"center",
            dataIndex: 'activity',
            customRender:(text)=>{
              if(!text){
                return ''
              }else{
                return filterMultiDictText(this.dictOptions['activity'], text+"")
              }
            }
          },
          {
            title:'地区',
            align:"center",
            dataIndex: 'region'
          },
          {
            title:'提醒时间',
            align:"center",
            dataIndex: 'remindertime',
            // customRender:function (text) {
            //   return !text?"":(text.length>10?text.substr(0,10):text)
            // }
          },
          {
            title:'参与的部门',
            align:"center",
            dataIndex: 'department',
            customRender:(text)=>{
              if(!text){
                return ''
              }else{
                return filterMultiDictText(this.dictOptions['department'], text+"")
              }
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
          list: "/participate/personnel/list",
          delete: "/participate/personnel/delete",
          deleteBatch: "/participate/personnel/deleteBatch",
          exportXlsUrl: "/participate/personnel/exportXls",
          importExcelUrl: "participate/personnel/importExcel",
        },
        dictOptions:{
         activity:[],
         department:[],
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
            this.$set(this.dictOptions, 'activity', res.result)
          }
        })
        initDictOptions('sys_depart,depart_name,id').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'department', res.result)
          }
        })
      },

      //设置列自动筛选 会和高级查询起冲突
      getQueryParams(){
        var param = Object.assign({}, this.queryParam,this.isorter);
        param.field = this.getQueryField();
        param.pageNo = this.ipagination.current;
        param.pageSize = this.ipagination.pageSize;
        return filterObj(param);
      },
       
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>