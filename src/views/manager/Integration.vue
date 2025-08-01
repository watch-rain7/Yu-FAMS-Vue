<template>
  <div>
    <el-tabs v-model="activeTab" type="card">
      <!-- 企业微信集成 -->
      <el-tab-pane label="企业微信集成" name="wechat">
        <el-card>
          <div slot="header">
            <span>企业微信配置</span>
            <el-button style="float: right; padding: 3px 0" type="text" @click="testWechatConfig">
              测试配置
            </el-button>
          </div>
          
          <el-form :model="wechatForm" label-width="120px" :rules="wechatRules" ref="wechatFormRef">
            <el-form-item label="企业ID" prop="corpId">
              <el-input v-model="wechatForm.corpId" placeholder="请输入企业ID"></el-input>
            </el-form-item>
            <el-form-item label="应用Secret" prop="agentSecret">
              <el-input v-model="wechatForm.agentSecret" placeholder="请输入应用Secret" show-password></el-input>
            </el-form-item>
            <el-form-item label="应用ID" prop="agentId">
              <el-input v-model="wechatForm.agentId" placeholder="请输入应用ID"></el-input>
            </el-form-item>
            <el-form-item label="回调Token" prop="token">
              <el-input v-model="wechatForm.token" placeholder="请输入回调Token"></el-input>
            </el-form-item>
            <el-form-item label="回调EncodingAESKey" prop="encodingAesKey">
              <el-input v-model="wechatForm.encodingAesKey" placeholder="请输入回调EncodingAESKey"></el-input>
            </el-form-item>
            <el-form-item label="是否启用">
              <el-switch v-model="wechatForm.enabled"></el-switch>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="saveWechatConfig">保存配置</el-button>
              <el-button @click="resetWechatForm">重置</el-button>
            </el-form-item>
          </el-form>
        </el-card>

        <el-card style="margin-top: 20px">
          <div slot="header">
            <span>企业微信功能测试</span>
          </div>
          
          <el-form :model="testForm" label-width="120px">
            <el-form-item label="用户ID">
              <el-input v-model="testForm.userId" placeholder="请输入要发送消息的用户ID"></el-input>
            </el-form-item>
            <el-form-item label="消息内容">
              <el-input v-model="testForm.content" type="textarea" placeholder="请输入要发送的消息内容"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="success" @click="sendWechatMessage">发送测试消息</el-button>
              <el-button @click="getWechatAuthUrl">获取授权URL</el-button>
            </el-form-item>
          </el-form>
        </el-card>
      </el-tab-pane>

      <!-- 钉钉集成 -->
      <el-tab-pane label="钉钉集成" name="dingtalk">
        <el-card>
          <div slot="header">
            <span>钉钉配置</span>
            <el-button style="float: right; padding: 3px 0" type="text" @click="testDingTalkConfig">
              测试配置
            </el-button>
          </div>
          
          <el-form :model="dingTalkForm" label-width="120px" :rules="dingTalkRules" ref="dingTalkFormRef">
            <el-form-item label="应用Key" prop="appKey">
              <el-input v-model="dingTalkForm.appKey" placeholder="请输入应用Key"></el-input>
            </el-form-item>
            <el-form-item label="应用Secret" prop="appSecret">
              <el-input v-model="dingTalkForm.appSecret" placeholder="请输入应用Secret" show-password></el-input>
            </el-form-item>
            <el-form-item label="企业ID" prop="corpId">
              <el-input v-model="dingTalkForm.corpId" placeholder="请输入企业ID"></el-input>
            </el-form-item>
            <el-form-item label="回调Token" prop="token">
              <el-input v-model="dingTalkForm.token" placeholder="请输入回调Token"></el-input>
            </el-form-item>
            <el-form-item label="回调AESKey" prop="aesKey">
              <el-input v-model="dingTalkForm.aesKey" placeholder="请输入回调AESKey"></el-input>
            </el-form-item>
            <el-form-item label="是否启用">
              <el-switch v-model="dingTalkForm.enabled"></el-switch>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="saveDingTalkConfig">保存配置</el-button>
              <el-button @click="resetDingTalkForm">重置</el-button>
            </el-form-item>
          </el-form>
        </el-card>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
export default {
  name: "Integration",
  data() {
    return {
      activeTab: 'wechat',
      wechatForm: {
        corpId: '',
        agentSecret: '',
        agentId: '',
        token: '',
        encodingAesKey: '',
        enabled: false
      },
      dingTalkForm: {
        appKey: '',
        appSecret: '',
        corpId: '',
        token: '',
        aesKey: '',
        enabled: false
      },
      testForm: {
        userId: '',
        content: ''
      },
      wechatRules: {
        corpId: [
          { required: true, message: '请输入企业ID', trigger: 'blur' }
        ],
        agentSecret: [
          { required: true, message: '请输入应用Secret', trigger: 'blur' }
        ],
        agentId: [
          { required: true, message: '请输入应用ID', trigger: 'blur' }
        ]
      },
      dingTalkRules: {
        appKey: [
          { required: true, message: '请输入应用Key', trigger: 'blur' }
        ],
        appSecret: [
          { required: true, message: '请输入应用Secret', trigger: 'blur' }
        ],
        corpId: [
          { required: true, message: '请输入企业ID', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 企业微信相关方法
    testWechatConfig() {
      this.$request.post('/integration/wechat/validate', this.wechatForm).then(res => {
        if (res.code === '200') {
          this.$message.success('配置验证成功');
        } else {
          this.$message.error(res.msg);
        }
      });
    },
    
    saveWechatConfig() {
      this.$refs.wechatFormRef.validate((valid) => {
        if (valid) {
          this.$message.success('企业微信配置保存成功');
        }
      });
    },
    
    resetWechatForm() {
      this.$refs.wechatFormRef.resetFields();
    },
    
    sendWechatMessage() {
      if (!this.testForm.userId || !this.testForm.content) {
        this.$message.warning('请填写用户ID和消息内容');
        return;
      }
      
      this.$request.post('/integration/wechat/sendMessage', null, {
        params: {
          userId: this.testForm.userId,
          content: this.testForm.content
        }
      }).then(res => {
        if (res.code === '200') {
          this.$message.success('消息发送成功');
        } else {
          this.$message.error(res.msg);
        }
      });
    },
    
    getWechatAuthUrl() {
      this.$request.get('/integration/wechat/authUrl', {
        params: {
          redirectUri: encodeURIComponent(window.location.origin + '/wechat-callback')
        }
      }).then(res => {
        if (res.code === '200') {
          this.$message.success('授权URL获取成功');
          console.log('企业微信授权URL:', res.data);
        } else {
          this.$message.error(res.msg);
        }
      });
    },
    
    // 钉钉相关方法
    testDingTalkConfig() {
      this.$message.info('钉钉配置验证功能待实现');
    },
    
    saveDingTalkConfig() {
      this.$refs.dingTalkFormRef.validate((valid) => {
        if (valid) {
          this.$message.success('钉钉配置保存成功');
        }
      });
    },
    
    resetDingTalkForm() {
      this.$refs.dingTalkFormRef.resetFields();
    }
  }
}
</script>

<style scoped>
.el-card {
  margin-bottom: 20px;
}
</style> 