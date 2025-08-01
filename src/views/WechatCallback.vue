<template>
  <div class="callback-container">
    <div class="callback-content">
      <div class="loading">
        <el-icon class="is-loading">
          <i class="el-icon-loading"></i>
        </el-icon>
        <p>正在处理企业微信登录...</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "WechatCallback",
  data() {
    return {
      code: ''
    }
  },
  created() {
    // 从URL参数中获取code
    this.code = this.$route.query.code;
    if (this.code) {
      this.handleWechatLogin();
    } else {
      this.$message.error('未获取到授权码');
      this.$router.push('/login');
    }
  },
  methods: {
    handleWechatLogin() {
      this.$request.post('/integration/wechat/login', null, {
        params: {
          code: this.code
        }
      }).then(res => {
        if (res.code === '200') {
          // 处理企业微信登录成功
          const userInfo = res.data;
          
          // 这里应该根据企业微信用户信息创建或更新本地用户
          // 暂时使用模拟数据
          const localUser = {
            id: userInfo.UserId || 1,
            username: userInfo.UserId || 'wechat_user',
            name: userInfo.UserId || '企业微信用户',
            role: 'STAFF', // 默认员工角色
            avatar: '',
            token: 'wechat_token_' + Date.now()
          };
          
          localStorage.setItem("xm-user", JSON.stringify(localUser));
          this.$message.success('企业微信登录成功');
          this.$router.push('/');
        } else {
          this.$message.error('企业微信登录失败：' + res.msg);
          this.$router.push('/login');
        }
      }).catch(error => {
        this.$message.error('企业微信登录失败');
        this.$router.push('/login');
      });
    }
  }
}
</script>

<style scoped>
.callback-container {
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f5f5f5;
}

.callback-content {
  text-align: center;
  padding: 40px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}

.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.loading i {
  font-size: 40px;
  color: #409EFF;
}

.loading p {
  color: #666;
  font-size: 16px;
  margin: 0;
}
</style> 