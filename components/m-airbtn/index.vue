<template>
  <view>
   <!-- #ifdef H5 -->
    <view class="wrapper" v-if="!weChat" @click="openApp()">
      <!-- Icon on the left -->
      <image class="img" :src="logo"></image>
      <view class="open">
        Open {{config.name}}
      </view>
    </view>

    <!-- #endif -->
  </view>
</template>

<script>
import config from "@/config/config";
export default {
  data() {
    return {
      config, // Set the tool class
      weChat: false, // Whether it is a WeChat browser, when this item is true, the entire page will not be displayed
      logo: require("@/icon.png"), //The circular logo displayed
    };
  },
  mounted() {
    // #ifdef H5
    // Determine whether it is a WeChat browser
    var ua = navigator.userAgent.toLowerCase();
    var isWeixin = ua.indexOf("micromessenger") != -1;
    if (isWeixin) {
      this.weChat = true;
    } else {
      this.weChat = false;
    }
    // #endif
  },
  methods: {

    /**
     * Jump to the download app page
     */
    downloadApp() {
      setTimeout(function () {
        window.location.href = config.downloadLink;
      }, 2000);
    },

    /**
     * Open the app only takes effect in h5 Use ifream to wake up the app
     */
    openApp() {
      let src;
      if (location.href) {
        src = location.href.split("/pages")[1];
      }
      let t = `${config.schemeLink}pages${src}`;

      try {
        var e = navigator.userAgent.toLowerCase(),
          n = e.match(/cpu iphone os (.*?) like mac os/);
        if (
          ((n = null !== n ? n[1].replace(/_/g, ".") : 0), parseInt(n) >= 9)
        ) {
          window.location.href = t;

          this.downloadApp();
        } else {
          var r = document.createElement("iframe");
          (r.src = t), (r.style.display = "none"), document.body.appendChild(r);

          this.downloadApp();
        }
      } catch (e) {
        window.location.href = t;
        this.downloadApp();
      }
    },
  },
};
</script>

<style scoped lang="scss">
.img {
  margin: 0 4rpx;
  width: 50rpx;
  height: 50rpx;
  border-radius: 50%;
  border: 5rpx solid #fff;
}

.open {
  margin: 0 10rpx;
  text-align: center;
  font-size: 26rpx;
}
.wrapper:hover {
  transform: translateX(0);
}
.wrapper {
  transform: translateX(160rpx);
  transition: 0.35s;
  align-items: center;
  justify-content: center;
  display: flex;
  border-top-left-radius: 20px;
  border-bottom-left-radius: 20px;
  color: #fff;
  // width: 220rpx;
  background: $light-color;
  position: fixed;
  top: 25%;
  right: 0;
  height: 60rpx;
  z-index: 9;
}
</style>
