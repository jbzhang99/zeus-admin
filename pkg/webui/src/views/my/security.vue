<template>
  <div v-if="accountInfo.is_open === 0">
    <div class="card twofactor-manager">
      <div class="card-header">
        <h4 class="card-title">两步验证</h4>
        <div class="card-tip">
          <p>要开启二步验证，您需要先安装 <a href="javascript:" class="qr-modal" @click="showGa">谷歌身份验证器</a>；</p>
        </div>
      </div>
      <div class="card-content with-border">
        <div class="twofactor-bind">
          <div class="twofactor-qrcode"><img :src="accountInfo.code" class="avatar"></div>
          <div class="bind-validator is-bfc">
            <div class="form-inline input-button">
              <el-form ref="form" :model="form" label-width="80px">
                <el-input v-model="form.code" class="bind-validator-code" placeholder="验证码"></el-input>
                <el-button type="primary" @click="onSubmit">立即绑定</el-button>
              </el-form>
            </div>
            <div class="card-tip">用身份验证器扫描左边的二维码，即可获得验证码</div>
          </div>
        </div>
        <div class="twofactor-account-info">
          <h4>帐户信息</h4>
          <div class="card-tip">若扫描二维码失败，您可以在您的身份验证器中输入以下帐户信息。</div>
          <div class="form-account-info">
            <el-form :model="form" label-width="200px">
              <el-form-item label-width="100px" label="帐户名称">
                <el-input v-model="form.account" :disabled="true" :placeholder="accountInfo.account"></el-input>
              </el-form-item>
              <el-form-item label-width="100px" label="密钥">
                <el-input v-model="form.account" :disabled="true" :placeholder="accountInfo.secret" ></el-input>
              </el-form-item>
              <el-form-item label-width="100px" label="算法">
                <el-input v-model="form.account" :disabled="true" placeholder="基于时间"></el-input>
              </el-form-item>
            </el-form>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div v-else class="filter-container">
    <h4 class="margin-resp-bottom">您已经开启了两步验证</h4>
    <el-form :model="form" class="filter-item">
      <el-row v-if="state === true">
        <el-button type="danger" @click="changeState">关闭</el-button>
      </el-row>
      <el-row v-else>
        <el-input v-model="form.code" class="filter-item" placeholder="输入身份验证器中的验证码"></el-input>
        <el-button type="danger" class="filter-item" style="display: inline-block;" @click="onClose">确认关闭</el-button>
      </el-row>
    </el-form>
    <!-- <a class="btn btn-lg btn-text">关闭后，可以再次开启</a> -->
  </div>
</template>
<script>
import { security, bindcode, close2fa } from '@/api/user'
export default {
  data() {
    return {
      form: {
        code: ''
      },
      accountInfo: [],
      state: true
    }
  },
  mounted() {
    this.getAccountInfo()
  },
  methods: {
    showGa() {
      this.$alert('<div class="qr-modal"><div class="qr-image-goog-auth"></div><p class="qr-tip">扫描二维码，立即下载</p></div>', '下载谷歌身份验证器', {
        dangerouslyUseHTMLString: true
      })
    },
    getAccountInfo() {
      security().then(res => {
        this.accountInfo = res.data.result
      })
    },
    onSubmit() {
      bindcode({ google_2fa_token: this.form.code }).then(res => {
        if (res.code === 200) {
          this.$message('绑定成功')
          location.reload()
          return
        } else {
          this.$message('绑定失败！')
          return
        }
      })
    },
    onClose() {
      close2fa({ google_2fa_token: this.form.code }).then(res => {
        if (res.code === 200) {
          this.$message('关闭成功')
          location.reload()
          return
        } else {
          this.$message('关闭失败！')
          return
        }
      })
    },
    changeState() {
      this.state = !this.state
    }
  }
}
</script>
<style lang="scss">
.qr-image-goog-auth {
  background-image: url(../../assets/images/google-authenticator.png);
  background-size: contain;
  width: 200px;
  height: 200px;
  margin: 10px auto;
  display: block;
}
.qr-tip{
  font-size: 14px;
  line-height: 80px;
  color: #383838;
  text-align: center;
}
</style>
<style lang="scss" scoped>
.card .card-header .card-title {
    font-weight:600;
    margin: 0;
}
.card .qr-modal{
  color: #3DA8F5;
}
.card .card-header.with-border {
    padding-bottom:15px
}
.card .card-header .card-title.with-handler {
    overflow:hidden;
    text-overflow:ellipsis;
    white-space:nowrap
}
.card .card-header .handler {
    float:right;
    margin-left:10px;
    font-size:20px
}
.card .card-header .handler .icon, .card .card-help .icon {
    float:left;
    top:-1px;
    position:relative
}
.card .card-header>:last-child {
    margin-bottom:0
}
.card .card-footer {
    padding:0 15px;
    padding-top:15px!important;
    padding-bottom:15px!important
}
.card .card-help, .card .card-tip {
    color:grey
}
.card .card-help .icon {
    margin-right:10px;
    font-size:20px;
    color:#ababab
}
.card-tip {
    line-height:20px;
    color:grey
}
.card-tip p {
    margin-bottom:3px
}
.twofactor-manager .twofactor-bind:after, .twofactor-manager .twofactor-bind:before {
    content:" ";
    display:table
}
.twofactor-bind {
  overflow: hidden;
}
.twofactor-manager .twofactor-qrcode {
  float: left;
  width:140px;
  height:140px;
  img{
    width: 100%;
    height: 100%;
  }
}
.bind-validator-code{
  width: 200px;
  margin: 10px 10px 0 0;
}
.twofactor-manager .input-button {
    margin-bottom:15px;
}

.twofactor-manager h4 {
    line-height:25px
}
.twofactor-manager .card-tip {
    line-height:20px
}
.twofactor-manager .card-tip p {
    margin-bottom:3px
}
.twofactor-manager .form-control[readonly] {
    cursor:auto;
    background:#fff;
    text-overflow:ellipsis
}
.twofactor-manager .emergency-code {
    font-size:14px
}
.twofactor-manager .emergency-code ul {
    margin:0;
    line-height:24px;
    padding-left:20px
}
.twofactor-manager .red {
    color:#FF4F3E
}

.is-bfc {
    overflow:hidden
}
.form-account-info {
    margin: 20px;
    width: 450px;
}
.margin-resp-bottom {
    margin-bottom:15px
}
</style>
