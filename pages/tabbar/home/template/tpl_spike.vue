<template>
  <div class="layout">
    <div class="join-list">
      <div class="join-title">
        <div>{{ res.list[0].title }}</div>
        <div>More</div>
      </div>
      <div class="join-box">
        <div class="join-item" v-for="item in 4" :key="item">
          <div class="item-img-box">
            <img
                class="item-img"
                src="https://picsum.photos/id/268/200/200"
                alt
            />
          </div>
          <div class="item-price">
            <span>￥120.00</span>
          </div>
          <div class="item-line-through">
            <span>￥190.00</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
// This feature will be developed in the follow-up version of TODO, please look forward to it
export default {
  props: ['res']
};
</script>
<style lang="scss" scoped>
@import "./tpl.scss";
@import './advertising.scss';

</style>
