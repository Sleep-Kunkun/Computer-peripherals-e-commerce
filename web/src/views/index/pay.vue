<template>
  <div>
    <Header />
    <div class="pay-content">
      <div class="title">订单提交成功</div>
      <div class="text time-margin">
        <span>请在 </span>
        <span class="time">{{ ddlTime }}</span>
        <span> 之前付款，超时订单将自动取消</span>
      </div>
      <div class="text">支付金额</div>
      <div class="price">
        <span class="num">{{ amount }}</span>
        <span>元</span>
      </div>
      <div class="pay-choose-view" style="">
        <div class="pay-choose-box flex-view">
          <div @click="chooseBoxActive(1)" ref="weixin"
            :class="['choose-box', { 'choose-box-active': boxActive === 1 }]">
            <img :src="WxPayIcon" />
            <span>微信支付</span>
          </div>
          <div @click="chooseBoxActive(2)" ref="zhifubao"
            :class="['choose-box', { 'choose-box-active': boxActive === 2 }]">
            <img :src="AliPayIcon" />
            <span>支付宝</span>
          </div>
        </div>
        <div class="tips">请选择任意一种支付方式</div>
        <button class="pay-btn pay-btn-active" @click="handlePay()">确认支付</button>
      </div>
      <div class="pay-qr-view" style="display: none">
        <div class="loading-tip" style="">正在生成安全支付二维码</div>
        <div class="qr-box" style="display: none">
          <div id="qrCode" class="qr"> </div>
          <div class="tips">请打开微信扫一扫进行支付</div>
          <button class="pay-finish-btn">支付完成</button>
          <button class="back-pay-btn">选择其他支付方式</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import Header from '/@/views/index/components/header.vue';
import { message } from 'ant-design-vue';
import WxPayIcon from '/@/assets/images/wx-pay-icon.svg';
import AliPayIcon from '/@/assets/images/ali-pay-icon.svg';
import { cancelApi } from '/@/api/admin/order';
import { useRoute, useRouter } from 'vue-router';
const route = useRoute();
const router = useRouter();
let ddlTime = ref();
let amount = ref();
let zhifubao = ref();
let weixin = ref();
let boxActive = ref(0);
let orderId = ref(route.query.orderId);
//定义定时器
var timer = null
onMounted(() => {
  amount.value = route.query.amount;
  ddlTime.value = formatDate(new Date().getTime(), 'YY-MM-DD hh:mm:ss');
});
onUnmounted(() => {
  if (timer) {
    clearTimeout(timer)
  }
})
//支付
async function handlePay() {
  console.log(orderId);

  cancelApi({ id: orderId.value, status: 2 })
    .then((res) => {
      console.log(res);
      message.success('支付成功');
      timer = setTimeout(() => {
        router.push({ name: "orderView" })
      }, 1000);
    })
    .catch((err) => {
      message.error(err.msg || '操作失败');
    });
}
//触发选择支付方式，修改对应的样式
const chooseBoxActive = (num) => {
  num === 1 ? (boxActive.value = 1) : 0;
  num === 2 ? (boxActive.value = 2) : 0;
};
const formatDate = (time, format = 'YY-MM-DD hh:mm:ss') => {
  const date = new Date(time);

  const year = date.getFullYear(),
    month = date.getMonth() + 1,
    day = date.getDate() + 1,
    hour = date.getHours(),
    min = date.getMinutes(),
    sec = date.getSeconds();
  const preArr = Array.apply(null, Array(10)).map(function (elem, index) {
    return '0' + index;
  });

  const newTime = format
    .replace(/YY/g, year)
    .replace(/MM/g, preArr[month] || month)
    .replace(/DD/g, preArr[day] || day)
    .replace(/hh/g, preArr[hour] || hour)
    .replace(/mm/g, preArr[min] || min)
    .replace(/ss/g, preArr[sec] || sec);

  return newTime;
};
</script>

<style scoped lang="less">
.flex-view {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
}

.pay-content {
  position: relative;
  margin: 120px auto 0;
  width: 500px;
  background: #fff;
  overflow: hidden;

  .title {
    color: #152844;
    font-weight: 500;
    font-size: 24px;
    line-height: 22px;
    height: 22px;
    text-align: center;
    margin-bottom: 11px;
  }

  .time-margin {
    margin: 11px 0 24px;
  }

  .text {
    height: 22px;
    line-height: 22px;
    font-size: 14px;
    text-align: center;
    color: #152844;
  }

  .time {
    color: #f62a2a;
  }

  .text {
    height: 22px;
    line-height: 22px;
    font-size: 14px;
    text-align: center;
    color: #152844;
  }

  .price {
    color: #ff8a00;
    font-weight: 500;
    font-size: 16px;
    height: 36px;
    line-height: 36px;
    text-align: center;

    .num {
      font-size: 28px;
    }
  }

  .pay-choose-view {
    margin-top: 24px;

    .choose-box {
      width: 140px;
      height: 126px;
      border: 1px solid #cedce4;
      border-radius: 4px;
      text-align: center;
      cursor: pointer;
    }

    .pay-choose-box {
      -webkit-box-pack: justify;
      -ms-flex-pack: justify;
      justify-content: space-between;
      max-width: 300px;
      margin: 0 auto;

      img {
        height: 40px;
        margin: 24px auto 16px;
        display: block;
      }
    }

    .tips {
      color: #6f6f6f;
      font-size: 14px;
      line-height: 22px;
      height: 22px;
      text-align: center;
      margin: 16px 0 24px;
    }

    .choose-box-active {
      border: 1px solid #f67280;
    }

    .tips {
      color: #6f6f6f;
      font-size: 14px;
      line-height: 22px;
      height: 22px;
      text-align: center;
      margin: 16px 0 24px;
    }

    .pay-btn {
      cursor: pointer;
      background: #c3c9d5;
      border-radius: 32px;
      width: 104px;
      height: 32px;
      line-height: 32px;
      border: none;
      outline: none;
      font-size: 14px;
      color: #fff;
      text-align: center;
      display: block;
      margin: 0 auto;
    }

    .pay-btn-active {
      background: #f67280;
    }
  }
}
</style>
