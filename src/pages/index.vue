<template>
  <view>
    <view class="header">
      <view class="item">
        <view class="title">实时收益</view>
        <view class="amount">100.00</view>
      </view>
    </view>
    <view class="operation">
      <view class="item">添加持仓</view>
      <view class="item">批量添加</view>
      <view class="item">删除持仓</view>
      <view class="item">修改持仓</view>
    </view>
    <view class="content">
      <view class="column">
        <view class="name">基金名称</view>
        <view class="rate">
          <view class="title">涨跌</view>
          <view class="icon">
            <u-icon name="arrow-upward"  size="28"></u-icon>
            <u-icon name="arrow-downward" color="green"  size="28"></u-icon>
          </view>
        </view>
        <view class="amount">
          <view class="title">收益</view>
          <view class="icon">
            <u-icon name="arrow-upward"  size="28"></u-icon>
            <u-icon name="arrow-downward" color="green"  size="28"></u-icon>
          </view>
        </view>
      </view>
      <view class="funds">
        <view class="item">
          <view class="name">基金名称</view>
          <view class="rate">1%</view>
          <view class="amount">20.00</view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    name: "index",
    data() {
      return {
        codes: [],
        lists:[]
      }
    },
    methods: {
      loadData() {

      }
    }
  }
</script>

<style lang="scss" scoped>
.header {
  display: flex;
  align-items: center;
  height: 160rpx;
  border-bottom: 1rpx solid #F8F8F8;
  .item {
    flex: 1;
    text-align: center;
    .title {
      font-size: 36rpx;
      margin-bottom: 10rpx;
    }
    .amount {
      font-size: 36rpx;
      font-weight: 500;
    }
  }
}
.operation {
  display: flex;
  height: 80rpx;
  align-items: center;
  border-bottom: 1rpx solid #F8F8F8;
  .item {
    flex: 1;
    text-align: center;
  }
}
.content {
  .column {
    display: flex;
    height: 80rpx;
    align-items: center;
    border-bottom: 1rpx solid #F8F8F8;
    .name {
      flex: 1;
    }
    .rate,.amount {
      width: 180rpx;
      text-align: center;
    }
    .rate,.amount {
      display: flex;
      align-items: center;
      justify-content: center;
      .title {
        margin-right: 5rpx;
      }
    }
    .icon {
      display: flex;
    }
  }
  .funds {
    .item {
      display: flex;
      height: 80rpx;
      align-items: center;
      border-bottom: 1rpx solid #F8F8F8;
      .name {
        flex: 1;
      }
      .rate,.amount {
        width: 180rpx;
        text-align: center;
      }
    }
  }
}
</style>