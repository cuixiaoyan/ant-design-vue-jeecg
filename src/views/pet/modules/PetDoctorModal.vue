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

        <a-form-item label="名字" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'name', validatorRules.name]" placeholder="请输入名字"></a-input>
        </a-form-item>
          
        <a-form-item label="工作年限" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择工作年限" v-decorator="[ 'workinglife', validatorRules.workinglife]" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
          
        <a-form-item label="手机号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'iphone', validatorRules.iphone]" placeholder="请输入手机号"></a-input>
        </a-form-item>
          
        <a-form-item label="级别" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag type="list" v-decorator="['level']" :trigger-change="true" dictCode="level" placeholder="请选择级别"/>
        </a-form-item>
          
        <a-form-item label="照片" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-upload v-decorator="['photo']" :trigger-change="true"></j-upload>
        </a-form-item>
          
        <a-form-item label="资格证" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-upload v-decorator="['qualificationsphoto']" :trigger-change="true"></j-upload>
        </a-form-item>
          
        <a-form-item label="简介" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'resume', validatorRules.resume]" placeholder="请输入简介"></a-input>
        </a-form-item>
          
        
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import JDate from '@/components/jeecg/JDate'  
  import JUpload from '@/components/jeecg/JUpload'
  import JDictSelectTag from "@/components/dict/JDictSelectTag"
  
  export default {
    name: "PetDoctorModal",
    components: { 
      JDate,
      JUpload,
      JDictSelectTag,
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
        name:{},
        workinglife:{},
        iphone:{},
        level:{},
        photo:{},
        qualificationsphoto:{},
        resume:{},
        },
        url: {
          add: "/doctors/petDoctor/add",
          edit: "/doctors/petDoctor/edit",
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
          this.form.setFieldsValue(pick(this.model,'name','workinglife','iphone','level','photo','qualificationsphoto','resume'))
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
        this.form.setFieldsValue(pick(row,'name','workinglife','iphone','level','photo','qualificationsphoto','resume'))
      }
      
    }
  }
</script>