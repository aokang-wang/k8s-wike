<template>
<div class="container-all">
  <el-alert
      style="margin:10px"
      title="游客模式登陆-请使用邮箱: test@ng2-oa.com 验证码: 123456登陆 "
      type="info">
  </el-alert>
  <el-form :model="ruleForm" :rules="rules" ref="ruleFormRef" label-width="150px" >
    <el-form-item label="用户邮箱"  prop="email" style="display: flex">
      <div style="display:flex;">
        <el-input v-model="ruleForm.email" style="margin-right:15px"></el-input>
        <el-button   @click="sendMsg">发送验证码</el-button>
      </div>
    </el-form-item>
    <el-form-item label="用户验证码" prop="code">
      <el-input v-model="ruleForm.code"></el-input>
    </el-form-item>

    <el-form-item class="right" style="margin-right:20px">
      <el-button type="primary" @click="submitForm()">立即登陆</el-button>
      <el-button @click="resetForm()">重置</el-button>
    </el-form-item>
  </el-form>
</div>
</template>

<script lang="ts">
import {defineComponent ,reactive,ref,toRefs} from 'vue'
import { ElMessage } from 'element-plus'
import {Login, sendMail} from "@/api/user";
import {doTo} from "@/router";
import {core} from "@/core/core";
export default defineComponent({
  name: 'view-login',
  components: {

  },
  setup(){
    let state=reactive({
      ruleForm:{
        email:"test@ng2-oa.com",
        code:"123456"
      },
    })
    const rules={
      email:[
        { required: true, message: '请输入邮箱地址', trigger: 'blur' },
        { type: 'email', message: '请输入正确的邮箱地址', trigger: ['blur', 'change'] }
      ],
      code:[
        { required: true, message: '请输入验证码', trigger: 'blur' },
      ]
    } //定义路由规则

    const ruleFormRef=ref(null) //设置refs

    function resetForm(){
      ruleFormRef.value.resetFields() //表单重置
    }

     //发送邮件
    function sendMsg(){
      ruleFormRef.value.validateField("email",async (errorMessage:string) => {
            if (errorMessage=="") {
              if (state.ruleForm.name == "test@ng2-oa.com") {
                ElMessage('请使用验证码123456登陆')
                return false;
              }

              try {
                let data = await sendMail(state.ruleForm.name)

                if (data.data.code == 200) {
                  ElMessage('您的验证码已经发送到你邮箱中，请登陆邮箱查看')
                } else {
                  ElMessage('您的邮箱未授权，或者邮箱错误')
                }
              } catch (e) {

              }
            
            }
          });
    }
    function submitForm(){
      ruleFormRef.value.validate(async (valid) => {
        if (valid) {
          try {
            let data=await  Login(state.ruleForm.email,state.ruleForm.code)
            if(data.data.code==200){
              localStorage.setItem('token', data.data.token);
              core.bus.emit("login",true)
              doTo("ns-list") //登录成功后跳转
            }
          }catch (e) {
            
          }
        }
      });
    }
    return {
      ...toRefs(state),
      rules,
      ruleFormRef,
      resetForm,
      submitForm,
      sendMsg,
    }
  }
})
</script>

<style scoped>

</style>
