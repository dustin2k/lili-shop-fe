<template>
  <div class="form">
    <u-form :model="codeForm" ref="validateCodeForm">
      <u-form-item class="cell" label-width="120" label="手机号" prop="mobile">
        <u-input maxlength="11" v-model="codeForm.mobile" placeholder="请输入您的手机号" />
      </u-form-item>

      <u-form-item class="cell code" label-width="120" prop="code" label="验证码">
        <div style="display:flex; with:100%;">
          <u-input v-model="codeForm.code" placeholder="请输入验证码" />
          <u-verification-code keep-running unique-key="page-login" :seconds="seconds" @end="end" @start="start" ref="uCode" @change="codeChange"></u-verification-code>
          <view @tap="getCode" class="text-tips">{{ tips }}</view>
        </div>
      </u-form-item>

      <view class="submit" @click="submit">登录</view>
      <view class="text-tips cell" @click="clickLogin">一键登录</view>
      <myVerification v-if="codeFlag" @send="verification" class="verification" ref="verification" business="LOGIN" />
    </u-form>
  </div>
</template>

<script>
import { sendMobile, smsLogin } from "@/api/login";
import { getUserInfo } from "@/api/members";
import storage from "@/utils/storage.js"; //缓存
import { whetherNavigate } from "@/utils/Foundation"; //登录跳转
import myVerification from "@/components/verification/verification.vue"; //验证码模块
import uuid from "@/utils/uuid.modified.js"; // uuid
export default {
  components: {
    myVerification,
  },
  props: ["status"], //是否勾选 《用户隐私》和《隐私政策》
  data() {
    return {
      uuid,
      flage: false, //是否验证码验证
      codeFlag: true, //验证开关，用于是否展示验证码
      codeForm: {
        mobile: "", //手机号
        code: "", //验证码
      },
      tips: "", //提示，点击发送验证码和重新发送时赋值
      clientType: "", // 客户端类型
      seconds: 60, //默认验证码等待时间
      // 二维码登录验证规则
      codeRules: {
        // 手机号验证
        mobile: [
          {
            validator: (rule, value, callback) => {
              return this.$u.test.mobile(value);
            },
            message: "手机号码不正确",
            trigger: ["blur"],
          },
        ],
        // 验证码验证
        code: [
          {
            min: 4,
            max: 6,
            required: true,
            message: "请输入验证码",
            trigger: ["blur"],
          },
        ],
      },
    };
  },
  // 必须要在onReady生命周期setRules，因为onLoad生命周期组件可能尚未创建完毕
  mounted() {
    // whetherNavigate();
    this.$refs.validateCodeForm.setRules(this.codeRules);
    /**
     * 条件编译判断当前客户端类型
     */
    //#ifdef H5
    this.clientType = "H5";
    //#endif
    //#ifdef APP-PLUS
    this.clientType = "APP";
    //#endif
  },
  watch: {
    async flage(val) {
      if (val) {
        if (this.$refs.uCode.canGetCode) {
          // 向后端请求验证码
          uni.showLoading({
            title: "正在获取验证码",
          });

          let res = await sendMobile(this.codeForm.mobile);

          uni.hideLoading();
          // 这里此提示会被this.start()方法中的提示覆盖
          if (res.data.success) {
            this.$refs.uCode.start();
          } else {
            uni.showToast({
              title: res.data.message,
              duration: 2000,
              icon: "none",
            });
            this.flage = false;
          }
        } else {
          this.$u.toast("请倒计时结束后再发送");
        }
      } else {
        this.$refs.verification.hide();
      }
    },
  },

  methods: {
    // 验证码验证
    verification(val) {
      this.flage = val == this.$store.state.verificationKey ? true : false;
    },
    //   登录
    submit() {
      if (!this.status) {
        /**
         * 用户必须了解
         * 用户协议以及隐私政策
         */
        uni.showToast({
          title: "请您阅读并同意用户协议以及隐私政策",
          duration: 2000,
          icon: "none",
        });
        return false;
      }
      let _this = this;
      this.$refs.validateCodeForm.validate((valid) => {
        if (valid) {
          /**
           * 清空当前账号信息
           */
          storage.setHasLogin(false);
          storage.setAccessToken("");
          storage.setRefreshToken("");
          storage.setUuid(this.uuid.v1());
          storage.setUserInfo({});
          /**
           * 执行登录
           */
          smsLogin(this.codeForm, _this.clientType).then((res) => {
            if (res.data.success) {
              storage.setAccessToken(res.data.result.accessToken);
              storage.setRefreshToken(res.data.result.refreshToken);

              /**
               * 登录成功后获取个人信息
               */
              getUserInfo().then((user) => {
                if (user.data.success) {
                  /**
                   * 个人信息存储到缓存userInfo中
                   */
                  storage.setUserInfo(user.data.result);
                  storage.setHasLogin(true);
                  // 登录成功
                  uni.showToast({
                    title: "登录成功!",
                    icon: "none",
                  });

                  /**
                   * 计算出当前router路径
                   * 1.如果跳转的链接为登录页面或跳转的链接为空页面。则会重新跳转到首页
                   * 2.都不满足返回跳转页面
                   */
                  whetherNavigate();
                } else {
                  uni.switchTab({
                    url: "/pages/tabbar/home/index",
                  });
                }
              });
            }
          });
        }
      });
    },
    // 跳转到一键登录
    clickLogin() {
      this.$emit("open", "click");
    },

    /**点击验证码*/
    codeChange(text) {
      this.tips = text;
    },
    /** 结束验证码后执行 */
    end() {},
    /**获取验证码 */
    getCode() {
      if (this.tips == "重新获取") {
        this.codeFlag = true;

        uni.showLoading({
          title: "加载中",
        });
        setTimeout(() => {
          this.$refs.verification.hide();
          uni.hideLoading();
        }, 2000);
      }

      if (!this.$u.test.mobile(this.codeForm.mobile)) {
        uni.showToast({
          title: "请输入正确手机号",
          icon: "none",
        });

        return false;
      }
      if (!this.flage) {
        this.$refs.verification.error();
        return false;
      }
    },

    // 点击获取验证码
    start() {
      this.$u.toast("验证码已发送");
      this.flage = false;

      this.codeFlag = false;
      this.$refs.verification.hide();
    },
  },
};
</script>
<style lang="scss" scoped>
@import url("./login.scss");

// #ifdef MP-WEIXIN
@import url("./mp-codeLogin.scss");

// #endif
</style>