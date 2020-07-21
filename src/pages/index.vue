<template>
  <view>
    <view class="header">
      <view class="item">
        <view class="title">实时收益</view>
        <view class="amount" :style="{color: amount > 0 ? 'red' : 'green'}">{{amount ? amount : '--'}}</view>
      </view>
    </view>
    <view class="operation">
      <view class="item" @click="add">添加持仓</view>
      <view class="item" @click="addBatch">批量添加</view>
      <view class="item" @click="update">修改持仓</view>
      <view class="item" @click="handleDel">删除持仓</view>
    </view>
    <view class="content">
      <view class="left">
        <view class="name">基金名称</view>
        <view class="title" v-for="(item, idx) of lists" :key="idx" @click="handleCheck(item)" :class="{'name--checked': codes_checked.indexOf(item.code) !== -1}">{{item.title}}</view>
      </view>
      <view class="right">
        <view class="column">
          <view class="rate" @click="handleSort('gszzl')">
            <view class="title">涨跌</view>
            <view class="icon">
              <u-icon name="arrow-upward" :color="sort['gszzl']==='up' ? 'red' : ''"  size="28"></u-icon>
              <u-icon name="arrow-downward" :color="sort['gszzl']==='down' ? 'red' : ''"  size="28"></u-icon>
            </view>
          </view>
          <view class="amount" @click="handleSort('amount')">
            <view class="title">收益</view>
            <view class="icon">
              <u-icon name="arrow-upward" :color="sort['amount']==='up' ? 'red' : ''"  size="28"></u-icon>
              <u-icon name="arrow-downward" :color="sort['amount']==='down' ? 'red' : ''"  size="28"></u-icon>
            </view>
          </view>
          <view class="amount" @click="handleSort('total_rate')">
            <view class="title">共涨</view>
            <view class="icon">
              <u-icon name="arrow-upward" :color="sort['total_rate']==='up' ? 'red' : ''"  size="28"></u-icon>
              <u-icon name="arrow-downward" :color="sort['total_rate']==='down' ? 'red' : ''"  size="28"></u-icon>
            </view>
          </view>
          <view class="amount" @click="handleSort('total_amount')">
          <view class="title">共计</view>
          <view class="icon">
            <u-icon name="arrow-upward" :color="sort['total_amount']==='up' ? 'red' : ''"  size="28"></u-icon>
            <u-icon name="arrow-downward" :color="sort['total_amount']==='down' ? 'red' : ''"  size="28"></u-icon>
          </view>
        </view>
          <view class="amount" @click="handleSort('principal')">
            <view class="title">本金</view>
            <view class="icon">
              <u-icon name="arrow-upward" :color="sort['principal']==='up' ? 'red' : ''"  size="28"></u-icon>
              <u-icon name="arrow-downward" :color="sort['principal']==='down' ? 'red' : ''"  size="28"></u-icon>
            </view>
          </view>
        </view>
        <view class="item-wrapper" v-for="(item, idx) of lists" :key="idx" @click="handleCheck(item)">
          <view class="item" :style="{color: item.amount > 0 ? 'red' : 'green'}">{{item.gszzl}}%</view>
          <view class="item" :style="{color: item.amount > 0 ? 'red' : 'green'}">{{item.amount}}</view>
          <view class="item" :style="{color: item.total_rate > 0 ? 'red' : 'green'}">{{item.total_rate}}%</view>
          <view class="item" :style="{color: item.total_amount > 0 ? 'red' : 'green'}">{{item.total_amount}}</view>
          <view class="item">{{item.principal}}</view>
        </view>
      </view>
    </view>
    <!--弹窗-->
    <u-popup v-model="show" mode="center" length="90%" border-radius="6">
      <view class="add-popup">
        <view class="title">{{is_add ? '添加' : '修改'}}持仓信息</view>
        <view class="from-wrapper">
          <u-form :model="form" ref="uForm">
            <u-form-item label="基金代码" label-width="160"><u-input v-model="form.code" type="number" @input="onInput" maxlength="6" /></u-form-item>
            <u-form-item label="基金名称" label-width="160"><u-input v-model="form.title" :disabled="true" placeholder="自动获取" /></u-form-item>
            <u-form-item label="持仓成本" label-width="160"><u-input v-model="form.price" type="digit" /></u-form-item>
            <u-form-item label="持仓份额" label-width="160"><u-input v-model="form.total" type="digit" /></u-form-item>
          </u-form>
        </view>
        <view class="button">
          <view class="cancel" @click="addCancel">取消</view>
          <view class="sure" @click="addSure">确定</view>
        </view>
      </view>
    </u-popup>

    <u-modal v-model="show_del" title="确定删除下列鸡" :mask-close-able="true" @confirm="confirmDel">
      <view class="del-wrapper">
        <view class="del-item" v-for="(item,idx) of lists_checked" :key="idx">
          {{item.title}}
        </view>
      </view>
    </u-modal>

    <u-modal v-model="show_batch" :show-cancel-button="true" title="批量添加鸡" :mask-close-able="true" @confirm="confirmBatch">
      <view class="batch">
        <u-input v-model="batch" height="400" placeholder="输入格式：代码:持仓成本:份额   一鸡一行" type="textarea" maxlength="10000" :border="true" />
      </view>
    </u-modal>
  </view>
</template>

<script>
  import NP from 'number-precision'
  NP.enableBoundaryChecking(false)

  import $ from 'jquery'
  export default {
    onLoad(options) {
      if (uni.getStorageSync('codes')) {
        this.codes = uni.getStorageSync('codes')
      }
      this.loadData()
      if (this.timer) {
        clearInterval(this.timer)
      }
      this.timer = setInterval(() => {
        this.loadData()
      },10000)
    },
    name: "index",
    data() {
      return {
        codes: [],
        lists:[],
        host: 'https://bird.ioliu.cn/v1/?url=http://fund.52sar.cn/getData?codes=000717',
        url: 'https://fundmobapi.eastmoney.com/FundMApi/FundFavorInformation.ashx?fundtype=0&Sort=desc&pageIndex=1&pagesize=30&deviceid=Wap&plat=Wap&product=EFund&version=2.0.0'+'&Fcodes=006408,320007',
        show: false,
        show_batch: false,
        form: {
          title: ''
        },
        codes_checked:[],
        show_del: false,
        lists_checked: [],
        sort: {
          amount: '',
          gszzl: ''
        },
        current_column: '',
        is_add: false,
        batch: '',
        timer: ''
      }
    },
    methods: {
      loadData() {
        if (this.codes.length === 0) {
          return
        }
        uni.request({
          url: 'https://bird.ioliu.cn/v1/?url=https://fundmobapi.eastmoney.com/FundMNewApi/FundMNFInfo',
          data: {
            Version: '6.2.8',
            deviceid: '58ff22b3b502d20bbef5507dbdfa8125||140748540302169',
            appType: 'ttjj',
            pageIndex: 1,
            Fcodes: this.code_str,
            plat: 'Android',
            pageSize: this.codes.length,
            product: 'EFund'
          },
          method: 'POST',
          success:(res) => {
            let temp = res.data.Datas.map(v => {
              let code = this.codes.find(v2 => v2.code === v.FCODE)
              let total_rate =  NP.round((v.NAV - code.price) / code.price * 100,2)
              let total_amount = NP.round((v.NAV - code.price) * code.total, 2)
              return {
                code: v.FCODE,
                title: v.SHORTNAME,
                gszzl: v.GSZZL,
                amount: NP.round(NP.times(NP.times(v.NAV, v.GSZZL)/100, code.total), 2),
                total_rate: total_rate,
                total_amount: total_amount,
                principal: NP.round(NP.times(code.price, code.total),0)
              }
            })
            if (this.current_column && this.sort[this.current_column]) {

              if (this.sort[this.current_column] === 'up') {
                temp.sort((a, b) => b[this.current_column] - a[this.current_column])
              } else if (this.sort[this.current_column] === 'down') {
                temp.sort((a, b) => a[this.current_column] - b[this.current_column])
              }
            }
            this.lists = temp
          },
          fail:(e) => {
            console.log(e)
          }
        })
      },
      loadFund(code) {
        uni.request({
          url: 'https://bird.ioliu.cn/v1/?url=https://fundmobapi.eastmoney.com/FundMApi/FundFavorInformation.ashx?fundtype=0&Sort=desc&pageIndex=1&pagesize=30&deviceid=Wap&plat=Wap&product=EFund&version=2.0.0&Fcodes=' + code,
          success: (res) => {
            console.log(res)
            this.form.title = res.data.Datas[0].SHORTNAME
          },
          fail: (e) => {
           this.form.title='获取数据失败'
           console.log(e)
          }
        })
      },
      add(){
        this.is_add = true
        this.show = true
      },
      onInput(code){
        if (code.length === 6) {
          this.loadFund(code)
        }
      },
      addSure() {
        console.log(this.form)
        let idx = this.codes.findIndex(v => v.code === this.form.code)
        console.log(idx)
        if (idx === -1) {
          this.codes.push({
            code: this.form.code,
            price: this.form.price,
            total: this.form.total
          })
        } else {
          this.codes[idx].price = this.form.price
          this.codes[idx].total = this.form.total
        }

        uni.setStorageSync('codes', this.codes)
        this.loadData()
        this.form = {}
        this.show = false
      },
      addCancel() {
        this.show = false
        this.form.title = ''
        this.form.code = ''
        this.form.price = ''
        this.form.total = ''
      },
      handleCheck(item) {
        let idx = this.codes_checked.indexOf(item.code)
        if (idx === -1) {
          this.codes_checked.push(item.code)
        } else {
          this.codes_checked.splice(idx, 1)
        }
      },
      handleDel() {
        if (this.codes_checked.length === 0) {
          uni.showToast({
            title: '请选择要删除的鸡！',
            icon: 'none'
          })
        } else {
          this.lists_checked = this.lists.filter(v=>this.codes_checked.indexOf(v.code) !== -1)
          this.show_del = true
        }
      },
      confirmDel() {
        this.codes = this.codes.filter(v=>this.codes_checked.indexOf(v.code) === -1)
        this.lists = this.lists.filter(v => this.codes_checked.indexOf(v.code) === -1)
        this.loadData()
        this.codes_checked = []
        uni.setStorageSync('codes', this.codes)
      },
      handleSort(column) {
        if (this.sort[column]==='up') {
          this.sort[column] = 'down'
        } else if(this.sort[column]==='down'){
          this.sort[column] = ''
        } else {
          this.sort[column] = 'up'
        }

        // 重置
        Object.keys(this.sort).forEach(v => {
          if (v !== column) {
            this.sort[v] = ''
          }
        })

        this.current_column = column
        this.sortColumn(column, this.sort[column])
      },
      sortColumn(column = 'gszzl',type = 'up') {

        if (type === 'up') {
          this.lists.sort((a, b) => b[column] - a[column])
        } else if (type === 'down') {
          this.lists.sort((a, b) => a[column] - b[column])
        } else {
          this.resetSort()
        }
      },
      resetSort() {
        this.lists = this.codes.map(v => {
          return this.lists.find(v2 => v2.code === v.code)
        })
      },
      addBatch() {
        this.show_batch = true
      },
      update() {
        if (this.codes_checked.length === 0) {
          uni.showToast({
            title: '请选择要修改的鸡！',
            icon: 'none'
          })
          return
        }
        if (this.codes_checked.length > 1) {
          uni.showToast({
            title: '一次只能修改一只鸡！',
            icon: 'none'
          })
          return
        }
        this.is_add = false
        let item = this.lists.find(v=>v.code === this.codes_checked[0])
        let code = this.codes.find(v=>v.code === this.codes_checked[0])
        this.form.title = item.title
        this.form.price = code.price
        this.form.total = code.total
        this.form.code = code.code
        this.show = true
      },
      confirmBatch() {
        let codes = []
        let codes2 = []
        this.batch.split('\n').forEach(v => {
          let temp = []
          temp = v.split(':')
          if (this.codes.indexOf(temp[0] === -1)) {
            codes2.push(temp[0])
            codes.push({
              code: temp[0],
              price: temp[1],
              total: temp[2]
            })
          }
        })
        let code_str = codes2.join(',')
        uni.request({
          url: 'https://bird.ioliu.cn/v1/?url=https://fundmobapi.eastmoney.com/FundMApi/FundFavorInformation.ashx?fundtype=0&Sort=desc&pageIndex=1&pagesize=30&deviceid=Wap&plat=Wap&product=EFund&version=2.0.0&Fcodes=' + code_str,
          success: (res) => {
            console.log(res)
            let codes3 = res.data.Datas.map(v => v.FCODE)
            codes = codes.filter(v=>codes3.indexOf(v.code) !== -1)
            this.codes.push(...codes)
            uni.setStorageSync('codes', this.codes)
            this.loadData()
          },
          fail: (e) => {
            console.log(e)
          }
        })
      }
    },
    computed: {
      code_str() {
        if (this.codes.length > 0) {
          return this.codes.map(v => v.code).join(',')
        }
        return ''
      },
      amount() {
        if (this.lists.length > 0) {
          return NP.round(this.lists.map(v=>v.amount).reduce((a,v)=>a+v), 2)
        } else {
          return ''
        }
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
    .name {
      flex: 1;
      margin-left: 32rpx;
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
}
.add-popup {
  padding-top: 20rpx;
  padding-left: 20rpx;
  padding-right: 20rpx;
  .title {
    font-size: 36rpx;
    text-align: center;
    margin-bottom: 20rpx;
  }
  .button {
    display: flex;
    height: 100rpx;
    align-items: center;
    font-size: 34rpx;
    .cancel,.sure {
      flex: 1;
      text-align: center;
    }
  }
}
.del-wrapper {
  padding: 20rpx;
  padding-bottom: 0;
  .del-item {
    height: 80rpx;
    line-height: 80rpx;
    border-bottom: 1rpx solid #F8F8F8;
  }
  .del-item:last-child {
    border-bottom: 0;
  }
}
.batch {
  padding: 20rpx;
}
.content {
  display: flex;
  .left {
    flex: 1;
    .name {
      height: 80rpx;
      display: flex;
      align-items: center;
      padding-left: 32rpx;
      border-bottom: 1rpx solid #F8F8F8;
    }
    .title {
      height: 100rpx;
      display: flex;
      align-items: center;
      padding-left: 32rpx;
      border-bottom: 1rpx solid #F8F8F8;
      position: relative;
    }
    .name--checked::after {
      content: '';
      position: absolute;
      left: 12rpx;
      top: 50%;
      width: 10rpx;
      height: 10rpx;
      border-radius: 5rpx;
      background: #515151;
    }
  }
  .right {
    width: 360rpx;
    overflow: scroll;
    .column {
      display: flex;
      height: 80rpx;
      align-items: center;
      .rate,.amount {
        width: 180rpx;
        height: 100%;
        flex-shrink: 0;
        border-bottom: 1rpx solid #F8F8F8;
      }
    }
    .item-wrapper {
      display: flex;
      .item {
        flex-shrink: 0;
        width: 180rpx;
        text-align: center;
        height: 100rpx;
        display: flex;
        align-items: center;
        padding-left: 32rpx;
        border-bottom: 1rpx solid #F8F8F8;
      }
    }
  }
}
</style>