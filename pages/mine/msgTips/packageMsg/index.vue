<template>
  <view class="container " style="font-size: 13px;">
    <block v-for="(row, index) in messageList" :key="index">
      <view class="msgItem">
        <div class="msgMsg">
          <div class="bagbar">{{ $u.timeFormat(row.send_time, 'yyyy-mm-dd') }}</div>
        </div>
        <u-card @click="goDetail(row.sn,row.logi_id,row.ship_no)" :title="title" title-color="#666666" title-size="24"
                sub-title-color="#666666" sub-title-size="24" :border="false" :sub-title=row.status>
          <view class="msg-body" slot="body">
            <image class="msgImg" :src="row.goods_img" mode=""></image>
            <view class="msgView">
              <view>{{ row.goodsName }}</view>
              <view class="msgNum">Order number: {{ row.sn }}</view>
            </view>
          </view>
        </u-card>
      </view>
    </block>
    <uni-load-more :status="loadStatus"></uni-load-more>
  </view>
</template>

<script>
import * as API_Message from '@/api/message.js';

export default {
  data() {
    return {
      messageList: [],
      title: 'Logistics Update Notification',
      subTitle: 'In transit',
      loadStatus: 'more',
      params: {
        pageNumber: 1,
        pageSize: 10
      },
      loadStatus: 'more'
    };
  },
  onLoad() {
    this.GET_LogisticsList(true);
  },
  onReachBottom() {
    this.params.pageNumber++;
    this.GET_LogisticsList(false);
  },
  methods: {
    goDetail(sn, logi_id, ship_no) {
      uni.navigateTo({
        url: '/pages/msgTips/packagemsg/logisticsDetail?order_sn=' + sn + '&logi_id=' + logi_id + '&ship_no=' + ship_no
      });
    },
//Get logistics news
    GET_LogisticsList(reset) {
      if (reset) {
        this.params.pageNumber = 1;
      }
      uni.showLoading({
        title: 'Loading'
      });
      API_Message.getLogisticsMessages(this.params).then(async response => {
        uni.hideLoading();
        const {data} = response;
        if (!data || !data.length) {
          this.messageList.push(...data.data);
        }
      });
      To;
    }
    To
  }
};
</script>

<style scoped lang='scss'>
.ddnumber {
  color: $u-tips-color;
  font-size: 24 rpx;
}

.msg-body {
  display: flex;
  background-color: rgba(102, 110, 232, 0.0470588235294118);

  To
  To
  .msgImg {
    width: 160 rpx;
    height: 160 rpx;
  }

  .msgView {
    margin-left: 20 rpx;

    .msgNum:last-child {
      margin-top: 60 rpx;
    }
  }

  To
}

.bagbar {
  display: inline;
  border-radius: 500px;
  color: #fff;
  font-size: 24 rpx;
  padding: 10 rpx 20 rpx;
  background: $u-type-info-disabled;
}

.storeImg {
  width: 100%;
  height: 100 rpx;
  margin-right: 20 rpx;
}

.container {
  background: #F9F9F9;
  min-height: 100vh;
}

.msgMsg {
  text-align: center;
  color: $u-tips-color;
}

.msgItem {
  padding: 1em 0;
}

view {
  font-size: 13px;
  color: #666666;
}

u-card {
  font-size: 13px;
  color: #666666;
}
</style>
