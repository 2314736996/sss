<template>
  <div class="login-container">
    <el-form ref="loginForm" :model="loginForm" class="login-form" autocomplete="on" label-position="left">
      <div class="title-container">
        <h3 class="title">系统登录</h3>
      </div>
      <el-form-item prop="username">
        <span class="svg-container">
          <svg-icon icon-class="user" />
        </span>
        <el-input
          ref="username"
          v-model="loginForm.petname"
          placeholder="用户名"
          name="username"
          type="text"
          tabindex="1"
          autocomplete="on"
        />
      </el-form-item>
      <el-tooltip v-model="capsTooltip" content="Caps lock is On" placement="right" manual>
        <el-form-item prop="password">
          <span class="svg-container">
            <svg-icon icon-class="password" />
          </span>
          <el-input
            :key="passwordType"
            ref="password"
            v-model="loginForm.uspws"
            :type="passwordType"
            placeholder="密码"
            name="password"
            tabindex="2"
            autocomplete="on"
            @blur="capsTooltip = false"
            @keyup.enter.native="handleLogin"
          />
        </el-form-item>
      </el-tooltip>
      <el-button type="primary" style="width:100%;margin-bottom:30px;" @click="handleLogin">登录</el-button>
    </el-form>
  </div>
</template>

<script>
  import { validUsername } from '@/utils/validate'
  import SocialSign from './components/SocialSignin'
  import store from '@/store/index'

  export default {
    name: 'Login',
    components: { SocialSign },
    store,
    data() {
      const validateUsername = (rule, value, callback) => {
        console.log('用户名：'+value)
        if (value==="") {
          callback(new Error('请输入用户名'))
        } else {
          callback()
        }
      }
      const validatePassword = (rule, value, callback) => {
        console.log('密码：'+value)
        if (value.length < 3) {
          callback(new Error('密码不能少于3位'))
        } else {
          callback()
        }
      }
      return {
        loginForm: {
          petname: '',
          uspws: ''
        },
        passwordType: 'password',
        capsTooltip: false,
        loading: false,
        showDialog: false,
        redirect: undefined,
        otherQuery: {}
      }
    },
    watch: {
      $route: {
        handler: function(route) {
          const query = route.query
          if (query) {
            this.redirect = query.redirect
            this.otherQuery = this.getOtherQuery(query)
          }
        },
        immediate: true
      }
    },
    mounted() {
      if (this.loginForm.petname === '') {
        this.$refs.petname.focus()
      } else if (this.loginForm.uspws === '') {
        this.$refs.uspws.focus()
      }
    },
    methods: {
      showPwd() {
        if (this.passwordType === 'password') {
          this.passwordType = ''
        } else {
          this.passwordType = 'password'
        }
        this.$nextTick(() => {
          this.$refs.uspws.focus()
        })
      },
      async handleLogin() {
        const hand_router = await this.$store.dispatch('permission/generateRoutes')
        let numReg = new RegExp(/^[0-9]*$/);
        let engReg = new RegExp(/^[a-zA-Z]+$/);
        if (!engReg.test(this.loginForm.petname)) {
          alert("输入不符合格式！")
          return false;
        }
        if (this.loginForm.petname.trim() === "") {
          alert("用户名不能为空！");
          return false;
        }
        if (this.loginForm.uspws === "") {
          alert("密码不能为空！");
          return false;
        }
        if (!numReg.test(this.loginForm.uspws)) {
          alert("请输入数字！");
          return false;
        }
        this.axios({url: "http://localhost:10086/system_login",method: 'post', data: this.loginForm, withCredentials: true}).then(result => {
          if (result.data.length !== 0) {
            if (result.data[0].type === 3 || result.data[0].type === 2) {
              alert("登录成功！")
              this.$store.commit('setName', result.data[0].petname);
              this.$store.commit('setPrint', result.data[0].type);
              this.$router.replace("/");
              this.$router.addRoutes(hand_router);
            }else{
              alert("权限不足！")
              return false;
            }
          }else{
            alert("用户名或密码错误！")
            return false;
          }
        });
      },
      getOtherQuery(query) {
        return Object.keys(query).reduce((acc, cur) => {
          if (cur !== 'redirect') {
            acc[cur] = query[cur]
          }
          return acc
        }, {})
      }
    }
  }
</script>

<style lang="scss">
  $bg:#283443;
  $light_gray:#fff;
  $cursor: #fff;

  @supports (-webkit-mask: none) and (not (cater-color: $cursor)) {
    .login-container .el-input input {
      color: $cursor;
    }
  }

  /* reset element-ui css */
  .login-container {
    .el-input {
      display: inline-block;
      height: 47px;
      width: 85%;

      input {
        background: transparent;
        border: 0px;
        -webkit-appearance: none;
        border-radius: 0px;
        padding: 12px 5px 12px 15px;
        color: $light_gray;
        height: 47px;
        caret-color: $cursor;

        &:-webkit-autofill {
          box-shadow: 0 0 0px 1000px $bg inset !important;
          -webkit-text-fill-color: $cursor !important;
        }
      }
    }

    .el-form-item {
      border: 1px solid rgba(255, 255, 255, 0.1);
      background: rgba(0, 0, 0, 0.1);
      border-radius: 5px;
      color: #454545;
    }
  }
</style>

<style lang="scss" scoped>
  $bg:#2d3a4b;
  $dark_gray:#889aa4;
  $light_gray:#eee;

  .login-container {
    min-height: 100%;
    width: 100%;
    background-color: $bg;
    overflow: hidden;

    .login-form {
      position: relative;
      width: 520px;
      max-width: 100%;
      padding: 160px 35px 0;
      margin: 0 auto;
      overflow: hidden;
    }

    .tips {
      font-size: 14px;
      color: #fff;
      margin-bottom: 10px;

      span {
        &:first-of-type {
          margin-right: 16px;
        }
      }
    }

    .svg-container {
      padding: 6px 5px 6px 15px;
      color: $dark_gray;
      vertical-align: middle;
      width: 30px;
      display: inline-block;
    }

    .title-container {
      position: relative;

      .title {
        font-size: 26px;
        color: $light_gray;
        margin: 0px auto 40px auto;
        text-align: center;
        font-weight: bold;
      }
    }

    .show-pwd {
      position: absolute;
      right: 10px;
      top: 7px;
      font-size: 16px;
      color: $dark_gray;
      cursor: pointer;
      user-select: none;
    }

    .thirdparty-button {
      position: absolute;
      right: 0;
      bottom: 6px;
    }

    @media only screen and (max-width: 470px) {
      .thirdparty-button {
        display: none;
      }
    }
  }
</style>
