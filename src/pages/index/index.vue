<template>
  <view class="content">
    <image class="logo" src="/static/logo.png"></image>
    <view>
      <text class="title">{{title}}</text>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        title: 'Hello'
      }
    },
    onLoad() {

    },
    methods: {}
  }
</script>

<style>
  .content {
    text-align: center;
    height: 400 upx;
  }

  .logo {
    height: 200 upx;
    width: 200 upx;
    margin-top: 200 upx;
  }

  .title {
    font-size: 36 upx;
    color: #8f8f94;
  }
</style>
