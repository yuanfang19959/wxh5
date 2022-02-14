微信支付
``` 关键代码
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
 * 微信支付步骤2 这一步可以把微信支付弹窗召唤起来，然后再加一个支付确认结果弹窗判断一下就差不多了
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
```