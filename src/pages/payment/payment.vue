<template>
    <div class="payment-wrap">
        <div class="payment-header">
            <i class="iconfont icon-left" @click="goBack"></i>
            <span>订单支付</span>
        </div>
        <div class="payment-content">
            <p v-text="`订单提交成功，请您尽快支付！订单号：${orderNo}`"></p>
            <p>请用沙箱支付宝扫描如下二维码进行支付：</p>
            <img :src="paymentImg"/>
        </div>
    </div>
</template>

<script>

  function sleep (time) {
    return new Promise((resolve) => setTimeout(resolve, time));
  }
  import {getUrlKey} from "../../common/js/util";

    export default {
        data() {
            return {
                paymentImg: '',
                orderNo: '',
                paystatus: 'false'
            }
        },
        created(){
            let $orderNo = getUrlKey('orderNo')
            this.orderNo = $orderNo
            this.getPaymentImg()
        },
        methods: {
            getPaymentImg(){
                this.$http('/api/order/pay.do',{
                    orderNo: this.orderNo
                },'POST').then((res)=>{
                    this.paymentImg = res.data.qrPath
                })
              this.payStatus();
            },
            goBack(){
                this.$router.push('./order-list?orderType=1')
            },
            payStatus(){
              console.log("正在查询订单状态")
              this.$http('/api/order/query_order_pay_status.do',{
                orderNo: this.orderNo
              },'POST').then((res)=>{
                console.log( "res.data   "+res.data)
                this.paystatus = res.data
                console.log("this.paystatus      "+this.paystatus)
              })
              console.log(this.paystatus)
              if(this.paystatus == true){
                alert("支付成功，即将跳转到订单列表")
                this.$router.push('./order-list?orderType=3')
              }else {
                /**每隔一秒查询一次*/
                sleep(3000).then(() => {
                  // 这里写sleep之后需要去做的事情
                  this.payStatus()
                })
                console.log("每隔3000ms")
              }
            }
        }
    }
</script>

<style lang="scss" type="text/scss" scoped>
    .payment-wrap{
        .payment-header{
            position: relative;
            width: 100%;
            height: 80px;
            line-height: 80px;
            text-align: center;
            font-size: 34px;
            background: #eee;
            .iconfont{
                position: absolute;
                left: 20px;
                top: 0;
                font-size: 46px;
            }
        }
        .payment-content{
            position: absolute;
            width: 600px;
            height: 600px;
            left: 50%;
            top: 50%;
            margin-left: -300px;
            margin-top: -300px;
            text-align: center;
            font-size: 40px;
            p:nth-child(1){
                font-size: 40px;
                color: #666666;
            }
            p:nth-child(2){
                padding: 20px 0;
                font-size: 44px;
                font-weight: bold;
                color: red;
            }
            img{
                width: 400px;
                height: 400px;
            }
        }
    }

</style>
