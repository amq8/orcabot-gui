<template>
  <div>
    <div v-show="step === 0">
      <p>
        Hi, <strong>{{ username }}</strong>!<br>
        Please choose your subscription type.
      </p>
      <div class="mt-2">
        <div class="card card-radio my-3" :class="{'selected': subType === 'profit'}"
             role="radio" @click="subType = 'profit'">
          <div class="card-content">
            <div class="content">
              <div class="title is-size-4">Profit sharing</div>
              30% profit share for master trader on every successful trade
            </div>
          </div>
        </div>
        <div class="card card-radio my-3" :class="{'selected': subType === 'subscription'}"
             role="radio" @click="subType = 'subscription'">
          <div class="card-content">
            <div class="content">
              <div class="title is-size-4">Subscription based</div>
              $29 per month
            </div>
          </div>
        </div>
      </div>

      <div class="field">
        <p class="control">
          <button class="button is-primary" :class="{'is-loading': loadingProfitSharing}" @click="submitType">
            Confirm
          </button>
        </p>
      </div>
    </div>

    <div v-show="step === 1 && subType === 'subscription'">
      <slot name="checkout"></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: "Subscription",
  props: {
    username: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      step: 0,

      subType: 'subscription',
      binanceKey: null,
      binanceSecret: null,

      loadingProfitSharing: false
    }
  },
  methods: {
    async submitType() {
      if (this.subType === 'profit') {
        await this.submitProfitSharing();
      } else {
        this.step++;
      }
    },
    async submitProfitSharing() {
      if (!this.loadingProfitSharing) {
        this.loadingProfitSharing = true;

        const reqData = new FormData()
        reqData.append('subType', 'profit')
        const result = await (await fetch('/u/subscription/submit', {
          method: 'POST',
          body: reqData
        })).json()

        if (result.error) {
          console.log(result.error)
        }

        location.href = '/u/dashboard'
      }
    }
  }
}
</script>
