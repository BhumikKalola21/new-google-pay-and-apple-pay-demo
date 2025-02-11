<template>
  <div>
    <h2>Express Checkout</h2>
    <div id="express-checkout-element"></div>
    <p id="error-message" class="text-red-500"></p>
  </div>
</template>

<script>
import { loadStripe } from "@stripe/stripe-js";

export default {
  async mounted() {
    console.log('VUE_APP_STRIPE_PUBLIC_KEY',process.env.VUE_APP_STRIPE_PUBLIC_KEY);
    console.log('VUE_APP_STRIPE_PUBLIC_KEY',process.env.VUE_APP_STRIPE_SECRET_KEY);
    const stripe = await loadStripe(process.env.VUE_APP_STRIPE_PUBLIC_KEY);

    const response = await fetch("https://rf8f5d67-3000.inc1.devtunnels.ms/create-intent", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ amount: 1099 }),
    });

    const { clientSecret } = await response.json();

    const elements = stripe.elements({
      mode: "payment",
      amount: 1099,
      currency: "usd",
    });

    const expressCheckoutElement = elements.create("expressCheckout");
    expressCheckoutElement.mount("#express-checkout-element");

    expressCheckoutElement.on("ready", ({ availablePaymentMethods }) => {
      if (availablePaymentMethods.length > 0) {
        alert('availablePaymentMethods' + availablePaymentMethods)
        document.getElementById("express-checkout-element").style.visibility = "visible";
      }
    });

    expressCheckoutElement.on("click", (event) => {
      event.resolve({
        emailRequired: true,
        phoneNumberRequired: true,
      });
    });

    expressCheckoutElement.on("confirm", async () => {
      const { error } = await stripe.confirmPayment({
        elements,
        clientSecret,
      });

      if (error) {
        document.getElementById("error-message").textContent = error.message;
      }
    });
  },
};
</script>

<style>
#error-message {
  color: red;
}
</style>
