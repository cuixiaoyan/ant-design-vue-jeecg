<template>
  <a-modal
    :title="title"
    :width="1200"
    :visible="visible"
    :maskClosable="false"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel">
    <a-spin :spinning="confirmLoading">
      <!-- 主表单区域 -->
      <a-form :form="form">
        <a-row>

          <a-col :span="12">
            <a-form-item label="主人名字" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
              <a-input v-decorator="[ 'zhurenmingzi', validatorRules.zhurenmingzi]" placeholder="请输入主人名字"></a-input>
            </a-form-item>
          </a-col>
        
          <a-col :span="12">
            <a-form-item label="年龄" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="[ 'age', validatorRules.age]" placeholder="请输入年龄" style="width: 100%"/>
            </a-form-item>
          </a-col>
        
          <a-col :span="12">
            <a-form-item label="性别" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
              <j-dict-select-tag  placeholder="请选择性别" dictCode="sex" :triggerChange="true"  v-decorator="[ 'sex', validatorRules.sex]"/>
            </a-form-item>
          </a-col>



        
          <a-col :span="12">
            <a-form-item label="电话" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
              <a-input v-decorator="[ 'iphone', validatorRules.iphone]" placeholder="请输入电话"></a-input>
            </a-form-item>
          </a-col>
        
          <a-col :span="12">
            <a-form-item label="等级" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag  placeholder="请选择等级" dictCode="member" :triggerChange="true"  v-decorator="[ 'huiyuan', validatorRules.huiyuan]"/>
            </a-form-item>

          </a-col>
        
          <a-col :span="12">
            <a-form-item label="种类" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-multi-select-tag type="list_multi" v-decorator="['weiyangzhonglei']" :trigger-change="true" dictCode="zhonglei" placeholder="请选择种类"/>
            </a-form-item>
          </a-col>
        
        </a-row>
      </a-form>

      <!-- 子表单区域 -->
      <a-tabs v-model="activeKey" @change="handleChangeTabs">
        <a-tab-pane tab="宠物狗" :key="refKeys[0]" :forceRender="true">
          <j-editable-table
            :ref="refKeys[0]"
            :loading="petDogTable.loading"
            :columns="petDogTable.columns"
            :dataSource="petDogTable.dataSource"
            :maxHeight="300"
            :rowNumber="true"
            :rowSelection="true"
            :actionButton="true"/>
        </a-tab-pane>
        
        <a-tab-pane tab="宠物猫" :key="refKeys[1]" :forceRender="true">
          <j-editable-table
            :ref="refKeys[1]"
            :loading="petCatTable.loading"
            :columns="petCatTable.columns"
            :dataSource="petCatTable.dataSource"
            :maxHeight="300"
            :rowNumber="true"
            :rowSelection="true"
            :actionButton="true"/>
        </a-tab-pane>
        
      </a-tabs>

    </a-spin>
  </a-modal>
</template>

<script>

  import pick from 'lodash.pick'
  import { FormTypes,getRefPromise } from '@/utils/JEditableTableUtil'
  import { JEditableTableMixin } from '@/mixins/JEditableTableMixin'
  import JMultiSelectTag from "@/components/dict/JMultiSelectTag"
  //排重接口 手机号校验
  import { duplicateCheck } from '@/api/api'


  export default {
    name: 'PetOwnersModal',
    mixins: [JEditableTableMixin],
    components: {
      JMultiSelectTag,
    },
    data() {
      return {
        labelCol: {
          span: 6
        },
        wrapperCol: {
          span: 16
        },
        labelCol2: {
          span: 3
        },
        wrapperCol2: {
          span: 20
        },
        // 新增时子表默认添加几行空数据
        addDefaultRowNum: 1,
        validatorRules: {
          zhurenmingzi:{rules: [
              { required: true, message: '请输入名字!'},
            ]},
          age:{},
          sex:{},
          iphone:{rules: [{ required: true, message: '请输入手机号!'},{validator: this.validatePhone}]},
          huiyuan:{},
          weiyangzhonglei:{},
        },
        refKeys: ['petDog', 'petCat', ],
        tableKeys:['petDog', 'petCat', ],
        activeKey: 'petDog',
        // 宠物狗
        petDogTable: {
          loading: false,
          dataSource: [],
          columns: [
            {
              title: '狗名',
              key: 'dogname',
              type: FormTypes.input,
              defaultValue: '',
              placeholder: '请输入${title}',
              validateRules: [{ required: true, message: '${title}不能为空' }],
            },
            {
              title: '年龄',
              key: 'age',
              type: FormTypes.inputNumber,
              defaultValue: '',
              placeholder: '请输入${title}',
            },

            {
              title: '性别',
              key: 'sex',
              type: FormTypes.select,
              options: [ // 下拉选项
                { title: '公', value: '1' },
                { title: '母', value: '2' }
              ],
              defaultValue: '',
              placeholder: '请输入${title}',
            },
            {
              title: '品种',
              key: 'type',
              type: FormTypes.input,
              defaultValue: '',
              placeholder: '请输入${title}',
            },
            {
              title: '项目',
              key: 'xiangmu',
              type: FormTypes.input,
              defaultValue: '',
              placeholder: '请输入${title}',
            },
            {
              title: '费用',
              key: 'feiyong',
              type: FormTypes.inputNumber,
              defaultValue: '',
              placeholder: '请输入${title}',
            },
          ]
        },
        // 宠物猫
        petCatTable: {
          loading: false,
          dataSource: [],
          columns: [
            {
              title: '猫名',
              key: 'catname',
              type: FormTypes.input,
              defaultValue: '',
              placeholder: '请输入${title}',
              validateRules: [{ required: true, message: '${title}不能为空' }],
            },
            {
              title: '年龄',
              key: 'age',
              type: FormTypes.inputNumber,
              defaultValue: '',
              placeholder: '请输入${title}',
            },
            {
              title: '性别',
              key: 'sex',
              type: FormTypes.select,
              options: [ // 下拉选项
                { title: '公', value: '1' },
                { title: '母', value: '2' }
              ],
              defaultValue: '',
              placeholder: '请输入${title}',
            },
            {
              title: '品种',
              key: 'type',
              type: FormTypes.input,
              defaultValue: '',
              placeholder: '请输入${title}',
            },
            {
              title: '项目',
              key: 'xiangmu',
              type: FormTypes.input,
              defaultValue: '',
              placeholder: '请输入${title}',
            },
            {
              title: '费用',
              key: 'feiyong',
              type: FormTypes.inputNumber,
              defaultValue: '',
              placeholder: '请输入${title}',
            },
          ]
        },
        url: {
          add: "/pet/petOwners/add",
          edit: "/pet/petOwners/edit",
          petDog: {
            list: '/pet/petOwners/queryPetDogByMainId'
          },
          petCat: {
            list: '/pet/petOwners/queryPetCatByMainId'
          },
        }
      }
    },


    methods: {
      getAllTable() {
        let values = this.tableKeys.map(key => getRefPromise(this, key))
        return Promise.all(values)
      },
      /** 调用完edit()方法之后会自动调用此方法 */
      editAfter() {
        let fieldval = pick(this.model,'zhurenmingzi','age','sex','iphone','huiyuan','weiyangzhonglei')
        this.$nextTick(() => {
          this.form.setFieldsValue(fieldval)
        })
        // 加载子表数据
        if (this.model.id) {
          let params = { id: this.model.id }
          this.requestSubTableData(this.url.petDog.list, params, this.petDogTable)
          this.requestSubTableData(this.url.petCat.list, params, this.petCatTable)
        }
      },
      /** 整理成formData */
      classifyIntoFormData(allValues) {
        let main = Object.assign(this.model, allValues.formValue)

        return {
          ...main, // 展开
          petDogList: allValues.tablesValue[0].values,
          petCatList: allValues.tablesValue[1].values,
        }
      },
      validateError(msg){
        this.$message.error(msg)
      },
      //手机号码校验
      validatePhone(rule, value, callback){
        if(!value){
          callback()
        }else{
          if(new RegExp(/^1[3|4|5|7|8][0-9]\d{8}$/).test(value)){
            var params = {
              tableName: 'sys_user',
              fieldName: 'phone',
              fieldVal: value,
              dataId: this.userId
            };
            duplicateCheck(params).then((res) => {
              if (res.success) {
                callback()
              } else {
                //可以存入相同的手机号 因为不是用户非要限制唯一的手机号码
                callback()
                //callback("手机号已存在!")
              }
            })
          }else{
            callback("请输入正确格式的手机号码!");
          }
        }
      },

      
      
    }
  }
</script>

<style scoped>
</style>