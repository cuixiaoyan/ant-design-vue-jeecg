<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="名称">
              <a-input placeholder="请输入名称查询" v-model="queryParam.activities"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="负责人">
              <a-input placeholder="请输入负责人查询" v-model="queryParam.personliable"></a-input>
            </a-form-item>
          </a-col>

          <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
            <a-col :md="6" :sm="24">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </a-col>
          </span>


        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->
    
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('发布活动')">导出</a-button>
      <!-- :headers="tokenHeader" 需要加上否则token验证失败 -->
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
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

        <div slot="filterDropdown">
          <a-card>
            <a-checkbox-group @change="onColSettingsChange" v-model="settingColumns" :defaultValue="settingColumns">
              <a-row>
                <template v-for="(item,index) in defColumns">
                  <template v-if="item.key!='rowIndex'&& item.dataIndex!='action'">
                    <a-col :span="12"><a-checkbox :value="item.dataIndex">{{ item.title }}</a-checkbox></a-col>
                  </template>
                </template>
              </a-row>
            </a-checkbox-group>
          </a-card>
        </div>
        <a-icon slot="filterIcon" type='setting' :style="{ fontSize:'16px',color:  '#108ee9' }" />
        
<!--        <template slot="imgSlot" slot-scope="text">-->
<!--          <span v-if="!text" style="font-size: 12px;font-style: italic;">无此图片</span>-->
<!--          <img v-else :src="getImgView(text)" height="25px" alt="图片不存在" style="max-width:80px;font-size: 12px;font-style: italic;"/>-->
<!--        </template>-->
<!--        <template slot="fileSlot" slot-scope="text">-->
<!--          <span v-if="!text" style="font-size: 12px;font-style: italic;">无此文件</span>-->
<!--          <a-button-->
<!--            v-else-->
<!--            :ghost="true"-->
<!--            type="primary"-->
<!--            icon="download"-->
<!--            size="small"-->
<!--            @click="uploadFile(text)">-->
<!--            下载-->
<!--          </a-button>-->
<!--        </template>-->

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

    <publishingActivities-modal ref="modalForm" @ok="modalFormOk"></publishingActivities-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import PublishingActivitiesModal from './modules/PublishingActivitiesModal'
  //数据字典模块
  import {initDictOptions, filterDictText} from '@/components/dict/JDictSelectUtil'
  //自定义列 需要的 大坑
  import Vue from 'vue'

  export default {
    name: "PublishingActivitiesList",
    mixins:[JeecgListMixin],
    components: {
      PublishingActivitiesModal,
    },
    data () {
      return {
        //字典数组缓存 活动类型
        acttypeDictOptions: [],
        description: '发布活动管理页面',
        //表头
        columns:[],
        //列设置
        settingColumns:[],
        //列定义
        defColumns: [
          {
            title: '序号',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: "center",
            customRender: function (t, r, index) {
              return parseInt(index) + 1;
            }
          },
          {
            title:'活动名称',
            align:"center",
            dataIndex: 'activities'
          },
          {
            title:'开始时间',
            align:"center",
            dataIndex: 'starttime'
          },
          {
            title:'结束时间',
            align:"center",
            dataIndex: 'endtime'
          },
          {
            title:'责任人',
            align:"center",
            dataIndex: 'personliable'
          },
          {
            title:'活动类型',
            align:"center",
            dataIndex: 'acttype',
            customRender: (text) => {
              //字典值替换通用方法
              return filterDictText(this.acttypeDictOptions, text);
            }

          },
          {
            title:'参与人数',
            align:"center",
            dataIndex: 'number'
          },
          {
            title:'电话号码',
            align:"center",
            dataIndex: 'telephone'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {
              filterDropdown: 'filterDropdown',
              filterIcon: 'filterIcon',
              customRender: 'action'},
          }
        ],
        url: {
          list: "/activities/publishingActivities/list",
          delete: "/activities/publishingActivities/delete",
          deleteBatch: "/activities/publishingActivities/deleteBatch",
          exportXlsUrl: "/activities/publishingActivities/exportXls",
          importExcelUrl: "activities/publishingActivities/importExcel",
        },
        dictOptions:{
        } 
      }
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },




    methods: {
      initDictConfig(){
        //初始化字典 - 活动类型
        initDictOptions('acttype').then((res) => {
          if (res.success) {
            this.acttypeDictOptions = res.result;
          }
        });
      },
      //列设置更改事件
      onColSettingsChange (checkedValues) {
        var key = this.$route.name+":colsettings";
        Vue.ls.set(key, checkedValues, 7 * 24 * 60 * 60 * 1000)
        this.settingColumns = checkedValues;
        const cols = this.defColumns.filter(item => {
          if(item.key =='rowIndex'|| item.dataIndex=='action'){
            return true
          }
          if (this.settingColumns.includes(item.dataIndex)) {
            return true
          }
          return false
        })
        this.columns =  cols;
      },
      initColumns(){
        //权限过滤（列权限控制时打开，修改第二个参数为授权码前缀）
        //this.defColumns = colAuthFilter(this.defColumns,'testdemo:');

        var key = this.$route.name+":colsettings";
        let colSettings= Vue.ls.get(key);
        if(colSettings==null||colSettings==undefined){
          let allSettingColumns = [];
          this.defColumns.forEach(function (item,i,array ) {
            allSettingColumns.push(item.dataIndex);
          })
          this.settingColumns = allSettingColumns;
          this.columns = this.defColumns;
        }else{
          this.settingColumns = colSettings;
          const cols = this.defColumns.filter(item => {
            if(item.key =='rowIndex'|| item.dataIndex=='action'){
              return true;
            }
            if (colSettings.includes(item.dataIndex)) {
              return true;
            }
            return false;
          })
          this.columns =  cols;
        }
      }


  },

    created() {
      this.initColumns();
    },

  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>