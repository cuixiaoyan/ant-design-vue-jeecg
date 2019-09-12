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

        <a-form-item label="活动名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'activities', validatorRules.activities]" placeholder="请输入活动名称"></a-input>
        </a-form-item>

        <a-form-item label="开始时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择开始时间" :showTime="true" dateFormat="YYYY-MM-DD HH:mm" v-decorator="[ 'starttime', validatorRules.starttime]" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
          
        <a-form-item label="结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择结束时间" :showTime="true" dateFormat="YYYY-MM-DD HH:mm" v-decorator="[ 'endtime', validatorRules.endtime]" :trigger-change="true" style="width: 100%"/>
        </a-form-item>

        <a-form-item label="活动类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag  placeholder="请选择活动类型" dictCode="acttype" :triggerChange="true"  v-decorator="[ 'acttype', validatorRules.acttype]"/>
        </a-form-item>

        <a-form-item label="责任人" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'personliable', validatorRules.personliable]" placeholder="请输入责任人"></a-input>
        </a-form-item>
          
        <a-form-item label="参与人数" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'number', validatorRules.number]" placeholder="请输入参与人数" style="width: 100%"/>
        </a-form-item>
          
        <a-form-item label="电话号码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'telephone', validatorRules.telephone]" placeholder="请输入电话号码" style="width: 100%"/>
        </a-form-item>
          
        
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  //时间组件
  import JDate from '@/components/jeecg/JDate'
  //排重接口
  import { duplicateCheck } from '@/api/api'


  export default {
    name: "PublishingActivitiesModal",
    components: { 
      JDate,
    },
    data () {
      return {
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
        activities:{
          rules: [
            { required: true, message: '请输入活动名称!'},
            { min: 0, max: 5, message: '长度不超过 10 个字符', trigger: 'blur' },
            { validator: this.validateRoleCode}
          ]},
        starttime:{},
        endtime:{},
        personliable:{rules: [
        { required: true, message: '请输入责任人名称!'},
        { min: 0, max: 5, message: '长度不超过 5 个字符', trigger: 'blur' }
      ]},
          acttype:{},
        number:{},
        telephone:{rules: [{validator: this.validatePhone}]},
        },
        url: {
          add: "/activities/publishingActivities/add",
          edit: "/activities/publishingActivities/edit",
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
          this.form.setFieldsValue(pick(this.model,'activities','starttime','endtime','personliable','acttype','number','telephone'))
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
        this.form.setFieldsValue(pick(row,'activities','starttime','endtime','personliable','acttype','number','telephone'))
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
                callback("手机号已存在!")
              }
            })
          }else{
            callback("请输入正确格式的手机号码!");
          }
        }
      },


      //唯一性校验规则
      validateRoleCode(rule, value, callback){
        // if(/[\u4E00-\u9FA5]/g.test(value)){
        //   callback("角色编码不可输入汉字!");
        // }else{
          var params = {
            //表名
            tableName: "publishingactivities",
            //字段名
            fieldName: "activities",
            fieldVal: value,
            //表的id
            dataId: this.model.id,
          };
          duplicateCheck(params).then((res)=>{
            if(res.success){
              callback();
            }else{
              callback(res.message);
            }
          });
        // }
      }
      
    }
  }
</script>