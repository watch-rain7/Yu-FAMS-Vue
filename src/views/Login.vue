<template>
  <div class="container">
    <div style="width: 100%; padding: 10px 20px; font-size: 24px; background-color: rgba(0,0,0,.3); color: white; position: absolute; top: 0">固定资产管理系统</div>
    <div style="width: 350px; padding: 30px; background-color: rgba(255, 255, 255, .8); border-radius: 5px;">
      <div style="text-align: center; font-size: 24px; margin-bottom: 30px; color: #333">欢迎使用</div>
      <el-form :model="form" :rules="rules" ref="formRef">
        <el-form-item prop="username">
          <el-input size="medium" prefix-icon="el-icon-user" placeholder="请输入账号" v-model="form.username"></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input  size="medium" prefix-icon="el-icon-lock" placeholder="请输入密码" show-password  v-model="form.password"></el-input>
        </el-form-item>
        <el-form-item prop="role">
          <el-select v-model='form.role'>
            <el-option value="ADMIN" label="管理员"></el-option>
            <el-option value="STAFF" label="员工"></el-option>
          </el-select>
        </el-form-item>
        
        <el-form-item>
          <div style="text-align: center; margin-bottom: 10px;">
            <span style="color: #666; font-size: 14px;">或使用第三方登录</span>
          </div>
          <div style="display: flex; justify-content: center; gap: 20px;">
            <el-button type="success" size="small" @click="wechatLogin" icon="el-icon-chat-dot-round">
              企业微信登录
            </el-button>
            <el-button type="primary" size="small" @click="dingTalkLogin" icon="el-icon-message">
              钉钉登录
            </el-button>
          </div>
        </el-form-item>
        <el-form-item>
          <div style="display: flex; align-items: center">
            <div style="flex: 1">
              <el-input size="medium" v-model="code" placeholder="请输入验证码"></el-input>
            </div>
            <Identify @click.native="refreshCode" :identify-code="identifyCode"/>
          </div>

        </el-form-item>
        <el-form-item>
          <el-button size="medium" style="width: 100%; background-color: #313131; border-color: #313131; color: white" @click="login">登 录</el-button>
        </el-form-item>
        <div style="display: flex; align-items: center">
          <div style="flex: 1"></div>
          <div style="flex: 1; text-align: right">
            还没有账号？请 <a href="/register">注册</a>
          </div>
        </div>
      </el-form>
    </div>
  </div>
</template>

<script>
import Identify from "@/components/Identify";
export default {
  name: "Login",
  components: {Identify},
  data() {
    return {
      form: { role: 'ADMIN' },
      code: '',
      // 图片验证码
      identifyCode: '',
      // 验证码规则
      identifyCodes: '3456789ABCDEFGHGKMNPQRSTUVWXY',
      rules: {
        username: [
          { required: true, message: '请输入账号', trigger: 'blur' },
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
        ]
      }
    }
  },
  created() {
    this.refreshCode()
  },
  methods: {
    login() {
      if(this.code.toLowerCase()!==this.identifyCode.toLowerCase()){
        this.$message.warning("验证码错误")
        this.refreshCode()
        this.code=''
        return
      }
      this.$refs['formRef'].validate((valid) => {
        if (valid) {
          // 验证通过
          this.$request.post('/login', this.form).then(res => {
            if (res.code === '200') {
              localStorage.setItem("xm-user", JSON.stringify(res.data))  // 存储用户数据
              this.$router.push('/')  // 跳转主页
              this.$message.success('登录成功')
            } else {
              this.refreshCode()
              this.code=''
              this.$message.error(res.msg)
            }
          })
        }
      })
    },
    // 企业微信登录
    wechatLogin() {
      this.$request.get('/integration/wechat/authUrl', {
        params: {
          redirectUri: encodeURIComponent(window.location.origin + '/wechat-callback')
        }
      }).then(res => {
        if (res.code === '200') {
          // 跳转到企业微信授权页面
          window.location.href = res.data;
        } else {
          this.$message.error('获取企业微信授权URL失败');
        }
      });
    },
    // 钉钉登录
    dingTalkLogin() {
      this.$message.info('钉钉登录功能待实现');
    },
    // 切换验证码
    refreshCode() {
      this.identifyCode = ''
      this.makeCode(this.identifyCodes, 4)
    },
    // 生成随机验证码
    makeCode(o, l) {
      for (let i = 0; i < l; i++) {
        this.identifyCode += this.identifyCodes[Math.floor(Math.random() * (this.identifyCodes.length))]
      }
    },
  }
}
</script>

<style scoped>
.container {
  height: 100vh;
  overflow: hidden;
  background-image: url("../assets/imgs/bg.jpg");
  background-size: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #666;
}
a {
  color: #2a60c9;
}
</style>