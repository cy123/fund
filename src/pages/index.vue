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
      <view class="column">
        <view class="name">基金名称</view>
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
      </view>
      <view class="funds">
        <view class="item" v-for="(item, idx) of lists" :key="idx" @click="handleCheck(item)">
          <view class="name" :class="{'name--checked': codes_checked.indexOf(item.code) !== -1}">{{item.title}}</view>
          <view class="rate" :style="{color: item.amount > 0 ? 'red' : 'green'}">{{item.gszzl}}%</view>
          <view class="amount" :style="{color: item.amount > 0 ? 'red' : 'green'}">{{item.amount}}</view>
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
  export default {
    onLoad() {
      this.loadData()
    },
    name: "index",
    data() {
      return {
        codes: [{
          code: '003634',
          price: '1.8642',
          total: '2413.92'
        },{
          code: '004857',
          price: '1.3737',
          total: '8219.46'
        },{
          code: '004433',
          price: '0.8070',
          total: '7435.04'
        }],
        lists:[],
        host: 'https://bird.ioliu.cn/v1/?url=http://fund.52sar.cn/getData?codes=000717',
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
        batch: ''
      }
    },
    methods: {
      loadData() {
        if (this.codes.length === 0) {
          return
        }
        uni.request({
          url: 'https://bird.ioliu.cn/v1/?url=http://fund.52sar.cn/getData?codes=' + this.code_str,
          success:(res) => {
            //console.log(res)
            let temp = res.data.Data.KFS.map(v => {
              let code = this.codes.find(v2 => v2.code === v.FCODE)
              return {
                code: v.FCODE,
                title: v.SHORTNAME,
                gszzl: v.gszzl,
                amount: NP.round(NP.times(NP.times(v.DWJZ, v.gszzl)/100, code.total), 2),
              }
            })
            if (this.current_column && this.sort[this.current_column]) {

              if (this.sort[this.current_column] === 'up') {
                temp.sort((a, b) => b[current_column] - a[current_column])
              } else if (this.sort[this.current_column] === 'down') {
                temp.sort((a, b) => a[current_column] - b[current_column])
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
          url: 'https://bird.ioliu.cn/v1/?url=http://fund.52sar.cn/getData?codes=' + code,
          success: (res) => {
            this.form.title = res.data.Data.KFS[0].SHORTNAME
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
        let idx = this.codes.indexOf(this.form.code)
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
        this.loadData()
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
        this.batch.split('\n').forEach(v => {
          let code = ''
          code = v.split(':')
        })
      }
    },
    computed: {
      code_str() {
        if (this.codes.length > 0) {
          return this.codes.map(v => v.code).join('_')
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
    border-bottom: 1rpx solid #F8F8F8;
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
  .funds {
    .item {
      display: flex;
      padding: 30rpx 0;
      align-items: center;
      border-bottom: 1rpx solid #F8F8F8;
      .name {
        flex: 1;
        padding-left: 32rpx;
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
      .rate,.amount {
        width: 180rpx;
        text-align: center;
      }
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
</style>