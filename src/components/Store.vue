<template>
  <!-- TODO: store where items can be bought-->
  <div>

 <VueStripeCheckout
       ref="checkoutRef"
    ></VueStripeCheckout>
        <button @click="checkout">Checkout</button>
  </div>
</template>

<script>
import axios from "axios";

export default {
  methods: {
    async checkout() {
      // token - is the token object
      // args - is an object containing the billing and shipping address if enabled
      const { token } = await this.$refs.checkoutRef.open();
      console.log(token);
      console.log(process.env.VUE_APP_PAYMENT_SERVER_ADDRESS);
      await axios.post(
        process.env.VUE_APP_PAYMENT_SERVER_ADDRESS,
        { hello: "world" },
        {
          headers: {
            "content-type": "application/json"
          }
        }
      );
    }
  }
};
</script>
