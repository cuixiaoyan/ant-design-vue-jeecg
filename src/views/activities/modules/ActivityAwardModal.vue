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


        <a-form-item label="第一名人数" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'diyirenshu', validatorRules.diyirenshu]" placeholder="请输入第一名人数" style="width: 100%"/>
        </a-form-item>
          
        <a-form-item label="第一名奖品" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'diyijiangpin', validatorRules.diyijiangpin]" placeholder="请输入第一名奖品"></a-input>
        </a-form-item>

          
        <a-form-item label="第二名人数" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'dierrenshu', validatorRules.dierrenshu]" placeholder="请输入第二名人数" style="width: 100%"/>
        </a-form-item>
          
        <a-form-item label="第二名奖品" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'direjiangpin', validatorRules.direjiangpin]" placeholder="请输入第二名奖品"></a-input>
        </a-form-item>
          

          
        <a-form-item label="第三名人数" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'disanrenshu', validatorRules.disanrenshu]" placeholder="请输入第三名人数" style="width: 100%"/>
        </a-form-item>
          
        <a-form-item label="第三名奖品" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'disanjiangpin', validatorRules.disanjiangpin]" placeholder="请输入第三名奖品"></a-input>
        </a-form-item>
          

          
        <a-form-item label="参与人数" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'canyurenshu', validatorRules.canyurenshu]" placeholder="请输入参与人数" style="width: 100%"/>
        </a-form-item>
          
        <a-form-item label="参与奖品" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'canyujiangpin', validatorRules.canyujiangpin]" placeholder="请输入参与奖品"></a-input>
        </a-form-item>
          


        <a-form-item label="哪次活动" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <j-search-select-tag
            placeholder="请输入哪次活动"
            v-decorator="[ 'huodong', validatorRules.huodong]"
            dict="publishingActivities,activities,id"
            :async="true">
          </j-search-select-tag>
        </a-form-item>
          
        <a-form-item label="是否结束" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <j-dict-select-tag type="list" v-decorator="['zhuangtai']" :trigger-change="true" dictCode="jieshu" placeholder="请选择是否结束"/>
        </a-form-item>
          
        <a-form-item label="结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <j-date placeholder="请选择结束时间" v-decorator="[ 'jieshushijian', validatorRules.jieshushijian]" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
          
        
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import JDate from '@/components/jeecg/JDate'  
  import JDictSelectTag from "@/components/dict/JDictSelectTag"

  //搜索下拉
  import JSearchSelectTag from '@/components/dict/JSearchSelectTag'
  
  export default {
    name: "ActivityAwardModal",
    components: { 
      JDate,
      JDictSelectTag,
      JSearchSelectTag,
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
        diyirenshu:{},
        diyijiangpin:{ rules: [
            { min: 0, max: 10, message: '长度不超过 10 个字符', trigger: 'blur' },
          ]},
        dierrenshu:{},
        direjiangpin:{rules: [
            { min: 0, max: 10, message: '长度不超过 10 个字符', trigger: 'blur' },
          ]},
        disanrenshu:{},
        disanjiangpin:{rules: [
            { min: 0, max: 10, message: '长度不超过 10 个字符', trigger: 'blur' },
          ]},
        canyurenshu:{},
        canyujiangpin:{rules: [
            { min: 0, max: 10, message: '长度不超过 10 个字符', trigger: 'blur' },
          ]},
        huodong:{rules: [
            { required: true, message: '请选择活动!'},]},
        zhuangtai:{},
        jieshushijian:{},
        },
        url: {
          add: "/activityAward/activityAward/add",
          edit: "/activityAward/activityAward/edit",
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
          this.form.setFieldsValue(pick(this.model,'diyirenshu','diyijiangpin','dierrenshu','direjiangpin','disanrenshu','disanjiangpin','canyurenshu','canyujiangpin','huodong','zhuangtai','jieshushijian'))
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
        this.form.setFieldsValue(pick(row,'diyirenshu','diyijiangpin','dierrenshu','direjiangpin','disanrenshu','disanjiangpin','canyurenshu','canyujiangpin','huodong','zhuangtai','jieshushijian'))
      }
      
    }
  }
</script>