<template>
  <a-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="用户名称" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <a-input v-decorator="[ 'name', validatorRules.name]" placeholder="请输入用户名称"></a-input>
        </a-form-item>
          

        <a-form-item label="手机号" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <a-input-number v-decorator="[ 'phone', validatorRules.phone]" placeholder="请输入手机号" style="width: 100%"/>
        </a-form-item>
          

        <a-form-item label="等级" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag  placeholder="请选择等级" dictCode="member" :triggerChange="true"  v-decorator="[ 'member', validatorRules.member]"/>
        </a-form-item>
          
        <a-form-item label="参与的人数" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'number', validatorRules.number]" placeholder="请输入参与的人数" style="width: 100%"/>
        </a-form-item>
          
<!--        <a-form-item label="哪次活动" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
<!--          <j-dict-select-tag type="list" v-decorator="['activity']" :trigger-change="true" dictCode="publishingActivities,activities,id" placeholder="请选择哪次活动"/>-->
<!--        </a-form-item>-->

        <a-form-item label="哪次活动" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <j-search-select-tag
            placeholder="请选择"
            v-decorator="[ 'activity', validatorRules.activity]"
            dict="publishingActivities,activities,id"
            :async="true">
          </j-search-select-tag>
        </a-form-item>
          
        <a-form-item label="活动地区" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'region', validatorRules.region]" placeholder="请输入活动地区"></a-input>
        </a-form-item>


<!--                <a-form-item label="活动地区" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
<!--                  <a-cascader :options="options"  :displayRender="displayRender" expandTrigger="hover" @change="onChange" placeholder="选择活动地区" />-->
<!--                </a-form-item>-->



          

        <a-form-item label="提醒时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择提醒时间" :showTime="true" dateFormat="YYYY-MM-DD HH:mm" v-decorator="[ 'remindertime', validatorRules.remindertime]" :trigger-change="true" style="width: 100%"/>
        </a-form-item>


        <a-form-item label="参与的部门" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-select-depart v-decorator="['department']" :trigger-change="true"/>
        </a-form-item>
          
        
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import JDate from '@/components/jeecg/JDate'  
  import JSelectDepart from '@/components/jeecgbiz/JSelectDepart'
  import JDictSelectTag from "@/components/dict/JDictSelectTag"

  //排重接口 手机号校验
  import { duplicateCheck } from '@/api/api'

  //搜索下拉
  import JSearchSelectTag from '@/components/dict/JSearchSelectTag'




  export default {
    name: "PersonnelModal",
    components: { 
      JDate,
      JSelectDepart,
      JDictSelectTag,
      JSearchSelectTag,
    },
    data () {

      return {


            options: [{
              value: '1',
              label: '河南',
              children: [{
                value: '2',
                label: '焦作',
                children: [{
                  value: '3',
                  label: '武陟',
                }],
              }],
            }, {
              value: '10',
              label: '上海',
              //设置是否禁用
              disabled: true,
              children: [{
                value: '20',
                label: '宝山',
                children: [{
                  value: '30',
                  label: '庙行',
                }],
              }],
            }],


        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        visible: false,
        model: {},


        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        confirmLoading: false,
        validatorRules:{
        name:{rules: [
            { required: true, message: '请输入名字!'},
          ]},
        phone:{rules: [{ required: true, message: '请输入手机号!'},{validator: this.validatePhone}]},
        member:{},
        number:{},
        activity:{rules: [
            { required: true, message: '请选择活动!'},]},
        region:{},
        remindertime:{},
        department:{},
        },
        url: {
          add: "/participate/personnel/add",
          edit: "/participate/personnel/edit",
        }
     
      }
    },
    created () {
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'name','phone','member','number','activity','region','remindertime','department'))
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }

            let formData = Object.assign(this.model, values);

            formData.region = formData.region.toString();

            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }
         
        })
      },
      handleCancel () {
        this.close()
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'name','phone','member','number','activity','region','remindertime','department'))
      },

      onChange(value) {
        // console.log(value.toString());
      },
      displayRender({ labels }) {
        return labels[labels.length - 1];
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