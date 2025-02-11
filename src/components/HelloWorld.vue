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
    console.log(
      "VUE_APP_STRIPE_PUBLIC_KEY",
      process.env.VUE_APP_STRIPE_PUBLIC_KEY
    );

    const stripe = await loadStripe(process.env.VUE_APP_STRIPE_PUBLIC_KEY);
    const elements = stripe.elements({
      mode: "payment",
      amount: 500,
      currency: "usd",
    });

    const expressCheckoutElement = elements.create("expressCheckout");
    expressCheckoutElement.mount("#express-checkout-element");

    expressCheckoutElement.on("ready", ({ availablePaymentMethods }) => {
      alert(JSON.stringify(availablePaymentMethods))
      console.log("Available Payment Methods:", availablePaymentMethods);
      if (availablePaymentMethods.length > 0) {
        document.getElementById("express-checkout-element").style.visibility =
          "visible";
      }
    });

    expressCheckoutElement.on("click", (event) => {
      event.resolve({
        emailRequired: true,
        phoneNumberRequired: true,
      });
    });

    expressCheckoutElement.on("confirm", async () => {
      // Step 1: Generate Confirmation Token
      const { error, confirmationToken } = await stripe.createConfirmationToken({
        elements, 
        payment_method_data: { billing_details: { email: "user@example.com" } } // Optional
      });

      if (error) {
        document.getElementById("error-message").textContent = error.message;
        return;
      }
      alert(confirmationToken.id)
      console.log("Generated Confirmation Token:", confirmationToken.id);

      // Step 2: Send Confirmation Token to Backend
      const response = await fetch(
        "https://8blkr6mj-3001.inc1.devtunnels.ms/api/v2/subscription/test-apple-pay/67a1d4ff2baca465ed21d0b7",
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ confirmationToken: confirmationToken.id }),
        }
      );

      const { clientSecret } = await response.json();

      if (!clientSecret) {
        alert("Failed to get client secret");
        return;
      }
      alert(clientSecret)
      console.log("Received Client Secret:", clientSecret);

      // Step 3: Confirm Payment with Stripe
      const { error: confirmError } = await stripe.confirmPayment({
        clientSecret,
        confirmParams: {
          return_url: "https://yourdomain.com/payment-success",
        },
      });

      if (confirmError) {
        document.getElementById("error-message").textContent = confirmError.message;
      } else {
        alert("Payment Successful!");
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
