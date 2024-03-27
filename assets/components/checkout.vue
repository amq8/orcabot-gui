<template>
  <div class="py-2 has-text-centered">
    <div class="notification is-danger my-2" v-if="error">
      {{ error }}
    </div>
    <div class="pb-5">
      <div class="title is-size-5 mb-3">Select a plan</div>
      <div class="columns is-multiline is-variable is-1">
        <div class="column" v-for="plan in plans" :key="plan.id" @click="selectedPlan = plan">
          <div class="card card-radio" :class="{'selected': selectedPlan.id === plan.id}">
            <div class="card-content p-3">
              <div class="content">
                <span class="is-size-5 is-block">{{ plan.description }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="py-5">
      <div class="title is-size-5 mb-3">Select a payment method</div>
      <div class="columns is-multiline is-variable is-1">
        <div class="column" v-for="(method, index) in paymentMethods"
             v-if="selectedPlan.supportedMethods.includes(index)"
             :key="index" @click="selectedPaymentMethod = index">
          <div class="card card-radio" :class="{'selected': selectedPaymentMethod === index}">
            <div class="card-content p-3">
              <div class="content">
                <span class="is-size-5 is-block">{{ method.name }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="py-3" v-if="selectedPaymentMethod !== null && selectedCurrency !== null && paymentAmount">
      <span class="is-size-5">
        Total: {{ paymentAmount }} {{ selectedCurrency }}
      </span>
    </div>

    <div class="py-5">
      <button class="button is-primary is-large" :class="{'is-loading': loadingCheckout}"
              :disabled="this.selectedPaymentMethod === null" @click="startCheckout">
        Checkout
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: "Checkout",
  props: {
    plans: {
      type: Array,
      required: true,
      validator: x => x.length > 0
    },
    paymentMethods: {
      type: Object,
      required: true,
      validator: x => Object.entries(x).length > 0
    }
  },
  computed: {
    selectedCurrency() {
      return this.paymentMethods[this.selectedPaymentMethod]?.currency
    },
    paymentAmount() {
      if (!this.selectedCurrency) return null;
      return this.selectedPlan.amounts[this.selectedCurrency]
    }
  },
  data() {
    return {
      selectedPlan: this.plans[0],
      selectedPaymentMethod: null,

      loadingCheckout: false,
      error: null
    }
  },
  methods: {
    async startCheckout() {
      if (!this.loadingCheckout) {
        this.loadingCheckout = true

        const reqData = new FormData()
        reqData.append('subType', 'subscription')
        reqData.append('plan', this.selectedPlan.id)
        reqData.append('method', this.selectedPaymentMethod)
        const result = await (await fetch('/u/subscription/submit', {
          method: 'POST',
          body: reqData
        })).json()

        if (result.error) {
          this.error = result.error
          this.loadingCheckout = false
          return
        }

        window.paymentHandlers[this.selectedPaymentMethod](result);
      }
    }
  },
  watch: {
    selectedPlan() {
      this.selectedPaymentMethod = null;
    }
  }
}
</script>

<style scoped>
.plan-card {
  cursor: pointer;
}

.plan-card.selected {
  border: 1px solid #0d6129;
}
</style>
