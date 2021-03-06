<template>
  <Panel :padding="30" class="container">
    <div slot="title" class="center">忘记密码</div>
    <template v-if="!successApply">
      <Form :rules="ruleResetPassword" :model=formResetPassword ref="formResetPassword">
        <Form-item prop="email">
          <Input v-model="formResetPassword.email" placeholder="电子邮箱地址" size="large">
          <Icon type="ios-email-outline" slot="prepend"></Icon>
          </Input>
        </Form-item>
        <Form-item prop="captcha" style="margin-bottom:10px">
          <div class="oj-captcha">
            <div class="oj-captcha-code">
              <Input v-model="formResetPassword.captcha" placeholder="验证码" size="large">
              <Icon type="ios-lightbulb-outline" slot="prepend"></Icon>
              </Input>
            </div>
            <div class="oj-captcha-img">
              <Tooltip content="Click to refresh" placement="top">
                <img :src="captchaSrc" @click="getCaptchaSrc"/>
              </Tooltip>
            </div>
          </div>
        </Form-item>
      </Form>
      <Button type="primary"
              @click="sendEmail"
              class="btn" long
              :loading="btnLoading">找回密码
      </Button>
    </template>
    <template v-else>
      <Alert type="success" show-icon>
        Success
        <span slot="desc">密码重置已经发送到您的邮箱.</span>
      </Alert>
    </template>
  </Panel>
</template>
<script>
  import api from '@oj/api'
  import {FormMixin} from '@oj/components/mixins'

  export default {
    mixins: [FormMixin],
    data () {
      const CheckEmailExist = (rule, value, callback) => {
        if (value !== '') {
          api.checkUsernameOrEmail(undefined, value).then(res => {
            if (res.data.data.email === false) {
              callback(new Error('The email doesn\'t exist'))
            } else {
              callback()
            }
          }, _ => callback())
        } else {
          callback()
        }
      }
      return {
        captchaSrc: '',
        successApply: false,
        btnLoading: false,
        formResetPassword: {
          email: '',
          captcha: ''
        },
        ruleResetPassword: {
          email: [
            {required: true, type: 'email', trigger: 'blur'},
            {validator: CheckEmailExist, trigger: 'blur'}
          ],
          captcha: [
            {required: true, trigger: 'blur', min: 1, max: 10}
          ]
        }
      }
    },
    mounted () {
      this.getCaptchaSrc()
    },
    methods: {
      sendEmail () {
        this.validateForm('formResetPassword').then(() => {
          this.btnLoading = true
          api.applyResetPassword(this.formResetPassword).then(res => {
            // 伪加载
            setTimeout(() => {
              this.btnLoading = false
              this.successApply = true
            }, 2000)
          }, _ => {
            this.btnLoading = false
            this.formResetPassword.captcha = ''
            this.getCaptchaSrc()
          })
        })
      }
    }
  }
</script>

<style scoped lang="less">
  .container {
    width: 450px;
    margin: auto;
    .center {
      text-align: center;
    }
    .btn {
      margin-top: 18px;
      text-align: center;
    }
  }
</style>
