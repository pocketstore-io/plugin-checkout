<template>
  <div v-if="!paymentDetails.id">
    <div id="paypal-button-container" class="mt-3"/>
  </div>
  <div v-else>
    <section class="alert alert-success text-white mb-3 block text-center">
      {{$t('alert.paypal.success')}}
    </section>
  </div>
</template>

<script lang="ts" setup>
import {useLocalStorage} from "@vueuse/core";
import {loadScript} from "@paypal/paypal-js";
import {cartToPurchaseUnits} from "~/utils/paypal";
import {faTrash} from "@fortawesome/free-solid-svg-icons";
import config from '~/pocketstore.json'

const {locked} = defineProps({
  locked: {
    type: Boolean,
    required: false,
    default: () => {
      return false;
    },
  },
});

const paymentMethod = useLocalStorage("paymentMethod", "vorkasse", {});
const checkoutStep = useLocalStorage("checkoutStep", "cart", {});
const paymentDetails = ref({});

const resetLock = () => {
};

const initPaypal = () => {
  loadScript({
    "client-id": config.payment.paypal.id,
    currency: config.payment.paypal.currency,
  })
      .then((paypal) => {
        paypal
            .Buttons({
              createOrder: (data, actions) => {
                // Directly create an order with the amount, no server needed
                return actions.order.create({
                  purchase_units: cartToPurchaseUnits(),
                });
              },
              onApprove: (data, actions) =>
                  actions.order.capture().then((details) => {
                    paymentDetails.value = details;
                  }),
              onError: (err) => {
                console.error(
                    "An error occurred during the transaction",
                    err,
                );
              },
            })
            .render("#paypal-button-container");
      })
      .catch((err) => {
        console.error("failed to load the PayPal JS SDK script", err);
      });
};

onMounted(() => {
  if (paymentMethod.value == "paypal" && checkoutStep.value == "payment") {
    initPaypal();
  }
});

watch(paymentMethod, (value) => {
  if (value === "paypal") {
    initPaypal();
  }
});

watch(checkoutStep, (value) => {
  if (paymentMethod.value == "paypal" && value == "payment") {
    initPaypal();
  }
});
</script>