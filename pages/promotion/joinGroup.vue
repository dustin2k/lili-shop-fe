<template>
  <view class="wrapper">

    <u-navbar :border-bottom="false" :background="background" class="unavbar" :title="title">
      <!-- middle -->
      <view class="slot-wrap container-wrap">

        <view v-if="search">
          <u-search @search="searchFun()" @custom="searchFun()" v-model="params.goodsName"></u-search>
        </view>

      </view>
      <!-- right side -->
      <view slot="right">
        <view style="margin-right: 24rpx;" @click="searchFlag()">
          <view v-if="search">Cancel</view>
          <u-icon v-if="!search" size="44rpx" name="search"></u-icon>
        </view>
      </view>
    </u-navbar>
    <!-- Top bar -->

    <!-- Commodity column -->
    <div class="swiper">

      <div v-if="groupBuy.length !=0">
        <view class="view-item" v-for="(groupItem, groupIndex) in groupBuy" :key="groupIndex">
          <view class="view-left">
            <u-image border-radius="10" shape="square" :src="groupItem.goodsImage" width="186rpx" height="186rpx">

              <view slot="error" style="font-size: 24rpx;">Failed to load</view>
            </u-image>
          </view>
          <view class="view-content">
            <view class="view-content-name">
              {{ groupItem.goodsName }}
            </view>
            <view class="view-content-bottom">
              <view>
                <view class="view-content-price">
                  <!-- ￥{{groupItem.sales_price | unitPrice }} <span v-if="groupItem.point">+{{groupItem.point}}Points</span> -->
                  ￥{{groupItem.price | unitPrice }}
                </view>
                <view class="view-content-original_price">
                  ￥{{ groupItem.originPrice | unitPrice }}
                </view>
              </view>

              <view>
                <view class="btn-group" @click="toHref(groupItem)"> To join the group </view>
                <view class="buy-content">Sold {{groupItem.num || 0}} pieces</view>
              </view>
            </view>
          </view>
        </view>
        <u-loadmore bg-color='#f8f8f8' :status="status" />
      </div>
      <u-empty v-else style="margin-top:20%" text="No group activities at the moment" mode="data"></u-empty>

    </div>

  </view>
</template>

<script>
import * as API_Promotions from "@/api/promotions";
import * as API_Goods from "@/api/goods";
export default {
  components: {},
  data() {
    return {
      status: "loadmore",
      is_empty: false,
      search: false,
      title: "Group Joining Activities",
      background: {
        backgroundColor: "#fff",
      },
      empty: false,
      params: {
        pageNumber: 1,
        pageSize: 10,
        categoryPath: "",
        goodsName: "",
      },
      groupBuy: [],
    };
  },
  mounted() {},
  watch: {
    search(val) {
      val? (this.title = ""): (this.title = "Join group activities");
    },
  },
  onReachBottom() {
    this.loadMore();
  },
  // Click the search button
  onNavigationBarButtonTap(e) {
    this.popupFlag = !this.popupFlag;
  },
  async onLoad() {
    this.GET_AssembleGoods();
  },

  methods: {
    loadMore() {
      this.params.pageNumber++;
      this.GET_AssembleGoods();
    },
    searchFlag() {
      this.search = !this.search;
    },

    toHref(goods) {
      uni.navigateTo({
        url: `/pages/product/goods?id=${goods.skuId}&goodsId=${goods.goodsId}`,
      });
    },
    searchFun() {
      this.groupBuy = [];
      this.GET_AssembleGoods();
    },
    // Request join group data
    GET_AssembleGoods() {
      this.status = "loading";
      const params = JSON.parse(JSON.stringify(this.params));
      if (params.category_id === 0) delete params.category_id;

      API_Promotions.getAssembleList(params)
        .then((response) => {
          const data = response.data.result.records;

          if (!data || !data.length) {
            this.is_empty = true;
            this.status = "nomore";
          } else {
            if (data.length <= this.params.pageSize) {
              this.status = "nomore";
            } else {
              this.status = "loadmore";
            }
            this.is_empty = false;
            this.groupBuy.push(...(data || []));
          }
        })
        .catch(() => {});
    },
  },
};
</script>

<style lang="scss" scoped>
@import "./style.scss";
</style>
