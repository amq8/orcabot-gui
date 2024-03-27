<template>
  <div class="modal" id="withdraw-modal">
    <div class="modal-background"></div>
    <div class="modal-card">
      <header class="modal-card-head">
        <p class="modal-card-title">Withdraw</p>
        <button class="delete" aria-label="close"></button>
      </header>
      <form method="POST" action="/u/api/withdraw">
        <input type="hidden" name="csrfToken" :value="this.csrfToken">
        <section class="modal-card-body">
          <p>
            Please choose the asset to withdraw, the amount and the destination address.
            <strong>
              <a href="https://www.binance.com/en/fee/deposit" target="_blank" rel="noopener noreferrer">
                Binance withdrawal fees
              </a> apply.
            </strong>
          </p>
          <div class="mt-4">
            <div class="field is-horizontal">
              <div class="field-body">
                <div class="field">
                  <label class="label is-sr-only" for="contractSel">Account</label>
                  <div class="control">
                    <span class="select is-fullwidth">
                        <select id="contractSel" name="contract" v-model="contract">
                          <option value="SPOT" v-if="balances['SPOT']">Spot</option>
                          <option value="FUTURES" v-if="balances['FUTURES']">Futures</option>
                        </select>
                    </span>
                  </div>
                </div>
                <div class="field has-addons">
                  <p class="control">
                    <label for="assetSel" class="is-sr-only">Asset</label>
                    <span class="select is-fullwidth">
                      <select id="assetSel" name="asset" v-model="assetStr">
                        <option v-for="b in balances[contract]" :value="b.asset">{{ b.asset }}</option>
                      </select>
                    </span>
                  </p>
                  <p class="control is-expanded">
                    <label for="amountInput" class="is-sr-only">Amount</label>
                    <input class="input" :class="{'is-danger': amountStr && !amountValid}" type="text" pattern="\d+(\.\d{1,8})?"
                           id="amountInput" name="amount" placeholder="Amount" v-model="amountStr" required>
                  </p>
                </div>
              </div>
            </div>
            <div class="is-flex is-align-items-center is-justify-content-flex-end">
              <strong>Total available:</strong>
              <code id="totalAvailable" class="px-1 py-0">
                {{ formatBalance(asset.balance) + assetStr }}
              </code>
              <button type="button" class="button is-dark is-small ml-2" @click="amountStr = formatBalance(asset.balance)">
                Max
              </button>
            </div>
            <div class="field is-grouped mt-5">
              <template v-if="assetStr === 'USDT'">
                <label class="label is-sr-only" for="networkSel">Network</label>
                <div class="control">
                <span class="select">
                    <select id="networkSel" name="network" v-model="networkStr">
                      <option value="ETH">ERC20</option>
                      <option value="TRX">TRC20</option>
                    </select>
                </span>
                </div>
              </template>

              <label class="label is-sr-only" for="addressInput">Address</label>
              <div class="control is-expanded">
                <input id="addressInput" name="address" class="input" type="text"
                       placeholder="Address" required>
              </div>
            </div>
          </div>
        </section>
        <footer class="modal-card-foot">
          <button type="submit" class="button is-primary" :disabled="!amountValid">Withdraw</button>
        </footer>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'withdraw_modal',
  props: {
    balances: {
      type: Object,
      required: true
    },
    csrfToken: {
      type: String,
      required: true
    }
  },
  computed: {
    asset() {
      return this.balances[this.contract].find(a => a.asset === this.assetStr)
    },
    amountValid() {
      if (!this.amountStr) return false
      const amount = this.parseBalance(this.amountStr);
      return this.amountStr && amount > 0 && this.asset.balance >= amount
    }
  },
  data() {
    return {
      contract: null,
      assetStr: null,
      amountStr: null,
      networkStr: null
    }
  },
  created() {
    this.contract = Object.keys(this.balances)[0]
    this.assetStr = this.balances[this.contract][0].asset
  },
  methods: {
    formatBalance(balance) {
      return (Number(balance) / Math.pow(10, 8)).toFixed(8)
    },
    parseBalance(balanceStr) {
      const split = balanceStr.split(/,\./)
      return Number(split[0]) * Math.pow(10, 8) + (split[1] ? Number(split[1]) : 0)
    }
  },
  watch: {
    contract(val) {
      this.assetStr = this.balances[val][0].asset
      this.amountStr = null
    },
    assetStr(val) {
      this.amountStr = null

      if (val === 'USDT') {
        this.networkStr = 'TRX'
      } else {
        this.networkStr = null
      }
    }
  }
}
</script>

<style scoped>

</style>