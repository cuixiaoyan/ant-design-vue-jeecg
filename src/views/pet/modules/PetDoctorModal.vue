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

        <a-form-item label="名字" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <a-input v-decorator="[ 'name', validatorRules.name]" placeholder="请输入名字"></a-input>
        </a-form-item>
          
        <a-form-item label="工作年限" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择工作年限" v-decorator="[ 'workinglife', validatorRules.workinglife]" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
          
        <a-form-item label="手机号" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <a-input v-decorator="[ 'iphone', validatorRules.iphone]" placeholder="请输入手机号"></a-input>
        </a-form-item>
          
        <a-form-item label="级别" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag type="list" v-decorator="['level']" :trigger-change="true" dictCode="level" placeholder="请选择级别"/>
        </a-form-item>
          


        <a-form-item label="照片" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-upload
            listType="picture-card"
            class="avatar-uploader"
            :showUploadList="false"
            :action="uploadAction"
            :data="{'isup':1}"
            :headers="headers"
            :beforeUpload="beforeUpload"
            @change="handleChange"
          >
            <img v-if="picUrl" :src="getAvatarView()" alt="照片" style="height:104px;max-width:300px"/>
            <div v-else>
              <a-icon :type="uploadLoading ? 'loading' : 'plus'" />
              <div class="ant-upload-text">点击上传</div>
            </div>
          </a-upload>
        </a-form-item>
          
        <a-form-item label="资格证" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-upload v-decorator="['qualificationsphoto']" :trigger-change="true"></j-upload>
        </a-form-item>
          

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="简介">
          <j-editor v-decorator="[ 'resume', {} ]" triggerChange></j-editor>
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
  //富文本编辑器
  import JEditor from '@/components/jeecg/JEditor'

  //图片框
  import moment from 'moment'
  import { getAction } from '@/api/manage'
  import {addUser,editUser,queryUserRole,queryall } from '@/api/api'
  import { disabledAuthFilter } from "@/utils/authFilter"
  import Vue from 'vue'
  import {ACCESS_TOKEN} from "@/store/mutation-types"

  //排重接口
  import { duplicateCheck } from '@/api/api'

  
  export default {
    name: "PetDoctorModal",
    components: { 
      JDate,
      JUpload,
      JDictSelectTag,
      JEditor,
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
        name:{ rules: [
            { required: true, message: '请输入名称!'},
            { min: 0, max: 10, message: '长度不超过 10 个字符', trigger: 'blur' },
            { validator: this.validateRoleCode}
          ]},
        workinglife:{},
        iphone:{rules: [{ required: true, message: '请输入手机号!'},{validator: this.validatePhone}]},
        level:{},
        photo:{},
        qualificationsphoto:{},
        resume:{},

          //图片框
          headers:{},
          uploadLoading:false,
          confirmLoading: false,
          picUrl: "",



        },
        url: {
          add: "/doctors/petDoctor/add",
          edit: "/doctors/petDoctor/edit",

          //图片框 上传文件和图片所调用的接口 更新码云测试
          fileUpload: window._CONFIG['domianURL']+"/sys/common/upload",
          imgerver: window._CONFIG['domianURL']+"/sys/common/view",
        }
     
      }
    },

    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token":token}

    },
    computed:{
      uploadAction:function () {
        return this.url.fileUpload;
      }
    },


    methods: {
      add () {
        //不想图片没有 就加这个
        this.picUrl = "";

        this.edit({});
      },
      edit (record) {
        this.form.resetFields();

        //不想图片没有 就加这个
        this.picUrl = "Has no pic url yet";

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

            //在这里替换一下图片 保存到后台数据库
            let photo = that.model.photo;

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
            formData.photo = photo;

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

      //图片上传方法
      beforeUpload: function(file){
        var fileType = file.type;
        if(fileType.indexOf('image')<0){
          this.$message.warning('请上传图片');
          return false;
        }
        //TODO 验证文件大小
      },
      handleChange (info) {
        this.picUrl = "";
        if (info.file.status === 'uploading') {
          this.uploadLoading = true;
          return
        }
        if (info.file.status === 'done') {
          var response = info.file.response;
          this.uploadLoading = false;
          console.log(response);
          if(response.success){
            this.model.photo = response.message;
            this.picUrl = "Has no pic url yet";
          }else{
            this.$message.warning(response.message);
          }
        }
      },
      //获取图片
      getAvatarView(){
        return this.url.imgerver +"/"+ this.model.photo;
      },


      handleCancel () {
        this.close()
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'name','workinglife','iphone','level','photo','qualificationsphoto','resume'))
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

      //唯一性校验规则
      validateRoleCode(rule, value, callback){
        // if(/[\u4E00-\u9FA5]/g.test(value)){
        //   callback("角色编码不可输入汉字!");
        // }else{
        var params = {
          //表名
          tableName: "petDoctor",
          //字段名
          fieldName: "name",
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
      },



      
    }
  }
</script>

<style scoped>
  .avatar-uploader > .ant-upload {
    width:104px;
    height:104px;
  }
  .ant-upload-select-picture-card i {
    font-size: 49px;
    color: #999;
  }

  .ant-upload-select-picture-card .ant-upload-text {
    margin-top: 8px;
    color: #666;
  }

  .ant-table-tbody .ant-table-row td{
    padding-top:10px;
    padding-bottom:10px;
  }

  .drawer-bootom-button {
    position: absolute;
    bottom: -8px;
    width: 100%;
    border-top: 1px solid #e8e8e8;
    padding: 10px 16px;
    text-align: right;
    left: 0;
    background: #fff;
    border-radius: 0 0 2px 2px;
  }
</style>

