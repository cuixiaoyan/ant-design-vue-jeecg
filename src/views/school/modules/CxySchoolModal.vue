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

        <a-form-item label="学校名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'name', validatorRules.name]" placeholder="请输入学校名称"></a-input>
        </a-form-item>
          
        <a-form-item label="最低年级" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'startGrade', validatorRules.startGrade]" placeholder="请输入最低年级" style="width: 100%"/>
        </a-form-item>
          
        <a-form-item label="最高年级" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'endGrade', validatorRules.endGrade]" placeholder="请输入最高年级" style="width: 100%"/>
        </a-form-item>
          
        <a-form-item label="地区" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'regionId', validatorRules.regionId]" placeholder="请输入地区"></a-input>
        </a-form-item>
          
        <a-form-item label="官网" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'website', validatorRules.website]" placeholder="请输入官网"></a-input>
        </a-form-item>
          
        <a-form-item label="联系电话" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'phone', validatorRules.phone]" placeholder="请输入联系电话"></a-input>
        </a-form-item>
          
        <a-form-item label="联系人" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'contacts', validatorRules.contacts]" placeholder="请输入联系人"></a-input>
        </a-form-item>
          
        <a-form-item label="地址" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-textarea v-decorator="['address']" rows="4" placeholder="请输入地址"/>
        </a-form-item>
          
        <a-form-item label="邮政编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'postCode', validatorRules.postCode]" placeholder="请输入邮政编码"></a-input>
        </a-form-item>
          
        <a-form-item label="图片" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-upload v-decorator="['pic']" :trigger-change="true"></j-upload>
        </a-form-item>
          
        <a-form-item label="简介" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'introduce', validatorRules.introduce]" placeholder="请输入简介"></a-input>
        </a-form-item>
          
        <a-form-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'remarks', validatorRules.remarks]" placeholder="请输入备注"></a-input>
        </a-form-item>
          
        <a-form-item label="文件存储名字" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'bucket', validatorRules.bucket]" placeholder="请输入文件存储名字"></a-input>
        </a-form-item>
          
        <a-form-item label="类别" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'schoolType', validatorRules.schoolType]" placeholder="请输入类别" style="width: 100%"/>
        </a-form-item>
          
        <a-form-item label="唯一标示" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'schoolSign', validatorRules.schoolSign]" placeholder="请输入唯一标示"></a-input>
        </a-form-item>
          
        <a-form-item label="学校状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'status', validatorRules.status]" placeholder="请输入学校状态" style="width: 100%"/>
        </a-form-item>
          
        
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import JUpload from '@/components/jeecg/JUpload'
  
  export default {
    name: "CxySchoolModal",
    components: { 
      JUpload,
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
        startGrade:{},
        endGrade:{},
        regionId:{},
        website:{},
        phone:{},
        contacts:{},
        address:{},
        postCode:{},
        pic:{},
        introduce:{},
        remarks:{},
        bucket:{},
        schoolType:{},
        schoolSign:{},
        status:{},
        },
        url: {
          add: "/school/cxySchool/add",
          edit: "/school/cxySchool/edit",
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
          this.form.setFieldsValue(pick(this.model,'name','startGrade','endGrade','regionId','website','phone','contacts','address','postCode','pic','introduce','remarks','bucket','schoolType','schoolSign','status'))
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
        this.form.setFieldsValue(pick(row,'name','startGrade','endGrade','regionId','website','phone','contacts','address','postCode','pic','introduce','remarks','bucket','schoolType','schoolSign','status'))
      }
      
    }
  }
</script>