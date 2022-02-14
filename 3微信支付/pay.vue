<script setup>
import { onMounted, reactive, toRefs, watch, watchEffect, onBeforeUpdate } from "vue";
import { Col, Row, Field, Dialog, Notify } from "vant";
import request from "@/api/request";
import { MARKORDER, PAY, CHECKORDERSTATUS } from "@/api/ApiConfig";
const props = defineProps({
  arrearmoney:"",
  obj:{}
});
const state = reactive({
  money: 10,
  nowIndex: 0,
  list: [10, 15, 20, 25, 50, 100],
  orderId: "",
});
let { money, list, nowIndex } = toRefs(state);
watch(money, (newVal, oldVal) => {
  if (!state.list.includes(Number(newVal))) {
    nowIndex.value = 7;
  }
});
onBeforeUpdate(() => {
  state.money = props.arrearmoney || 10;
  state.obj = props.obj
})
/**
 * 支付后弹窗验证
 */
const payResult = (orderId) => {
  Dialog.confirm({
    title: "提示",
    message: "您是否已经完成支付？",
  }).then(() => {
    request({
      method: "POST",
      url: CHECKORDERSTATUS,
      params: {
        command: "30021.208",
        orderId,
        type: 2,
        buyType: 2,
      },
    }).then((res) => {
      let { status, message } = res;
      Notify({ type: !status ? "success" : "danger", message });
    });
  });
};
/**
 * 支付步骤一。下单
 */
const markorder = () => {
  let { money, obj } = state;
  if(money <= 0) {
    Notify({type:'danger', message:'请输入正确的金额'})
    return;
  }
  request({
    method: "POST",
    url: MARKORDER,
    params: {
      meterNum: obj.meterNumber,
      openInvoice: 0,
      areaId: 0,
      money,
      mobile: obj.meterMobile,
      feeids: "",
    },
  }).then((res) => {
    if (!res.status) {
      paynow(res.data.orderId);
    }
  });
};
/**
 * 支付步骤二。唤起微信支付
 */
const paynow = (orderId) => {
  request({
    method: "POST",
    url: PAY,
    params: {
      orderId,
      type: 2,
      payMethod: 2,
      openId: localStorage.getItem("openId3") || localStorage.getItem("openId"),
    },
  }).then((res) => {
    if (!res.status) {
      weixinPay(res.data);
    }
  });
};
/**
 * 微信支付步骤1
 */
const weixinPay = (payData) => {
  if (typeof WeixinJSBridge == "undefined") {
    if (document.addEventListener) {
      document.addEventListener("WeixinJSBridgeReady", onBridgeReady, false);
    } else if (document.attachEvent) {
      document.attachEvent("WeixinJSBridgeReady", onBridgeReady);
      document.attachEvent("onWeixinJSBridgeReady", onBridgeReady);
    }
  } else {
    onBridgeReady(payData);
  }
};
/**
 * 微信支付步骤2
 */
const onBridgeReady = (payData) => {
  WeixinJSBridge.invoke(
    "getBrandWCPayRequest",
    {
      appId: payData.appid,
      timeStamp: payData.timestamp,
      nonceStr: payData.noncestr,
      package: "prepay_id=" + payData.prepayid,
      signType: "MD5",
      paySign: payData.sign,
    },
    function (res) {
      if (res.err_msg == "get_brand_wcpay_request:ok") {
      }
      if (res.err_msg == "get_brand_wcpay_request:cancel") {
      }
      if (res.err_msg == "get_brand_wcpay_request:fail") {
      }
      payResult();
    }
  );
};
</script>

<template>
  <div class="money-input">
    <div>选择缴费金额</div>
    <Row>
      <Col span="24">
        <Field
          v-model="money"
          name="money"
          type="number"
          placeholder="输入缴费金额"
          :rules="[{ required: true, message: '请填写用户名' }]"
        >
          <template #button>
            <div class="btn" @click="markorder">立即缴费</div>
          </template>
        </Field>
      </Col>
    </Row>
  </div>

  <div class="money-box">
    <Row gutter="10">
      <Col
        span="8"
        v-for="(r, idx) in list"
        :key="idx"
        @click="
          nowIndex = idx;
          money = r;
        "
      >
        <div :class="[nowIndex === idx ? 'money-select' : 'money-select active']">
          {{ r }}元
        </div>
      </Col>
    </Row>
  </div>
</template>

<style lang="scss" scoped>
.money-input {
  border-top: 20px solid #f3f3f3;
  :deep(.van-field) {
    padding-left: 0 !important;
    input {
      font-size: 28px;
      color: #ecb23b;
    }
  }
  padding: 20px;
  .btn {
    color: rgb(25, 137, 250);
  }
}
.money-box {
  padding: 0 20px;
  .money-select {
    transition: all ease;
    margin-bottom: 20px;
    padding: 49px 0;
    text-align: center;
    font-size: 44px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #ffffff;
    background: linear-gradient(138deg, rgb(58 26 201 / 45%) 0%, #23709d 100%);
    border-radius: 8px;
  }
  .active {
    background: #d8d8d8;
    color: #333333;
  }
}
</style>