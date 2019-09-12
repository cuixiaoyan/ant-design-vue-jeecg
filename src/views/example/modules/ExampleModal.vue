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

        <a-form-item label="用户名" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'name', validatorRules.name]" placeholder="请输入用户名"></a-input>
        </a-form-item>
          
        <a-form-item label="性别" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag type="radio" v-decorator="['sex']" :trigger-change="true" dictCode="sex" placeholder="请选择性别"/>
        </a-form-item>
          
        <a-form-item label="年龄" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'age', validatorRules.age]" placeholder="请输入年龄" style="width: 100%"/>
        </a-form-item>

        <a-form-item label="生日" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择生日" v-decorator="[ 'birthday', validatorRules.birthday]" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
          
        <a-form-item label="手机号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'userCode', validatorRules.userCode]" placeholder="请输入手机号"></a-input>
        </a-form-item>

        <a-form-item label="头像" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-upload v-decorator="['topPic']" :trigger-change="true" :headers="tokenHeader" ></j-upload>
        </a-form-item>
          
        <a-form-item label="简历" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-upload v-decorator="['fileKk']" :trigger-change="true" :headers="tokenHeader"></j-upload>
        </a-form-item>
          
        <a-form-item label="部门" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-select-depart v-decorator="['bumen']" :trigger-change="true"/>
        </a-form-item>
          
        <a-form-item label="领导" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-select-user-by-dep v-decorator="['lingdao']" :trigger-change="true"/>
        </a-form-item>

        <a-form-item label="描述" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-textarea v-decorator="[ 'descc', validatorRules.descc]" placeholder="请输入描述"></a-textarea>
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
  import JSelectDepart from '@/components/jeecgbiz/JSelectDepart'
  import JSelectUserByDep from '@/components/jeecgbiz/JSelectUserByDep'
  import JDictSelectTag from "@/components/dict/JDictSelectTag"

  export default {
    name: "ExampleModal",
    components: { 
      JDate,
      JUpload,
      JSelectDepart,
      JSelectUserByDep,
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
        sex:{},
        age:{},
        descc:{},
        birthday:{},
        userCode:{},
        topPic:{},
        fileKk:{},
        bumen:{},
        lingdao:{},
        },
        url: {
          add: "/examples/example/add",
          edit: "/examples/example/edit",
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
          this.form.setFieldsValue(pick(this.model,'name','sex','age','descc','birthday','userCode','topPic','fileKk','bumen','lingdao'))
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
        this.form.setFieldsValue(pick(row,'name','sex','age','descc','birthday','userCode','topPic','fileKk','bumen','lingdao'))
      }
      
    }
  }
</script>