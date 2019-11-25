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

        <a-form-item label="学校名称" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <a-input v-decorator="[ 'name', validatorRules.name]" placeholder="请输入学校名称"></a-input>
        </a-form-item>

        <a-form-item label="最低年级" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <a-input-number v-decorator="[ 'startGrade', validatorRules.startGrade]" placeholder="请输入最低年级" style="width: 100%"/>
        </a-form-item>

        <a-form-item label="最高年级" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <a-input-number v-decorator="[ 'endGrade', validatorRules.endGrade]" placeholder="请输入最高年级" style="width: 100%"/>
        </a-form-item>

        <a-form-item label="地区" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'regionId', validatorRules.regionId]" placeholder="请输入地区"></a-input>
        </a-form-item>

        <a-form-item label="官网" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'website', validatorRules.website]" placeholder="请输入官网"></a-input>
        </a-form-item>

        <a-form-item label="联系电话" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <a-input-number v-decorator="[ 'phone', validatorRules.phone]" placeholder="请输入联系电话" style="width: 100%"/>
        </a-form-item>



        <a-form-item label="联系人" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <j-search-select-tag
            placeholder="请做出你的选择"
            v-decorator="[ 'contacts', validatorRules.contacts]"
            dict="sys_user,realname,id"
            :async="true">
          </j-search-select-tag>
        </a-form-item>

<!--        <a-form-item label="联系人" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>-->
<!--        <j-select-user-by-dep v-decorator="[ 'contacts', validatorRules.contacts]" ></j-select-user-by-dep>-->
<!--        </a-form-item>-->



        <a-form-item label="地址" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'address', validatorRules.address]" placeholder="请输入地址"></a-input>
        </a-form-item>


        <a-form-item label="邮政编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'postCode', validatorRules.postCode]" placeholder="请输入邮政编码"></a-input>
        </a-form-item>

        <a-form-item label="图片" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-upload v-decorator="['pic']" :trigger-change="true"></j-upload>
        </a-form-item>


        <a-form-item label="简介" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-textarea v-decorator="[ 'introduce', validatorRules.introduce]" rows="4" placeholder="请输入简介"/>
        </a-form-item>

<!--        <a-form-item label="简介" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
<!--              <j-ellipsis v-decorator="[ 'introduce', validatorRules.introduce]" length="30" placeholder="请输入简介"/>-->
<!--        </a-form-item>-->



<!--        <a-form-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
<!--          <a-input v-decorator="[ 'remarks', validatorRules.remarks]" placeholder="请输入备注"></a-input>-->
<!--        </a-form-item>-->

        <a-form-item label="文件存储名字" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'bucket', validatorRules.bucket]" placeholder="请输入文件存储名字"></a-input>
        </a-form-item>


        <a-form-item label="类别" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag  placeholder="请选择类别" dictCode="schoolType" :triggerChange="true"  v-decorator="[ 'schoolType', validatorRules.schoolType]"/>
        </a-form-item>


<!--        <a-form-item label="唯一标示" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
<!--          <a-input v-decorator="[ 'schoolSign', validatorRules.schoolSign]" placeholder="请输入唯一标示"></a-input>-->
<!--        </a-form-item>-->

<!--        <a-form-item label="学校状态" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
<!--          <a-input-number v-decorator="[ 'status', validatorRules.status]" placeholder="请输入学校状态" style="width: 100%"/>-->
<!--        </a-form-item>-->


      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import JUpload from '@/components/jeecg/JUpload'

  //时间组件
  import JDate from '@/components/jeecg/JDate'
  //排重接口
  import { duplicateCheck } from '@/api/api'
  //搜索下拉
  import JSearchSelectTag from '@/components/dict/JSearchSelectTag'
  //过长裁剪
  import JEllipsis from '@/components/jeecg/JEllipsis'

  import JSelectUserByDep from '@/components/jeecgbiz/JSelectUserByDep'

  import moment from 'moment'
  import Vue from 'vue'
  import { ACCESS_TOKEN } from "@/store/mutation-types"
  import { getAction } from '@/api/manage'
  import {addUser,editUser,queryUserRole,queryall } from '@/api/api'
  import { disabledAuthFilter } from "@/utils/authFilter"

  export default {
    name: "CxySchoolModal",
    components: {
      JUpload,
      JSearchSelectTag,
      JEllipsis,
      JSelectUserByDep,
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
        userRealName: '',

        confirmLoading: false,
        validatorRules:{
        name:{
          rules: [
            { required: true, message: '请输入学校名称!'},
            { min: 0, max: 10, message: '长度不超过 10 个字符', trigger: 'blur' },
            { validator: this.validateRoleCode}
          ]},
        startGrade:{},
        endGrade:{},
        regionId:{},
        website:{},
        phone:{rules: [{ required: true, message: '请输入手机号!'},{validator: this.validatePhone}]},
        contacts:{rules: [
            { required: true, message: '请输入联系人名称!'},
          ]},
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
          tableName: "cxy_school",
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
      }



    }
  }
</script>