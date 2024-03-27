<template>
  <div class="modal" id="internal-transfer-modal">
    <div class="modal-background"></div>
    <div class="modal-card">
      <header class="modal-card-head">
        <p class="modal-card-title">Internal Transfer</p>
        <button class="delete" aria-label="close"></button>
      </header>
      <form method="POST" action="/u/api/internalTransfer">
        <input type="hidden" name="csrfToken" :value="this.csrfToken">
        <section class="modal-card-body">
          <p>
            Please choose the transfer direction, the asset to transfer and the amount.
          </p>
          <div class="mt-4">
            <div class="field is-horizontal">
              <div class="field-body">
                <div class="field">
                  <label class="label is-sr-only" for="destinationSel">Direction</label>
                  <div class="control">
                    <span class="select is-fullwidth">
                        <select id="destinationSel" name="destination" v-model="destination">
                          <option value="SPOT" v-if="balances['FUTURES']">Futures ➜ Spot</option>
                          <option value="FUTURES" v-if="balances['SPOT']">Spot ➜ Futures</option>
                        </select>
                    </span>
                  </div>
                </div>
                <div class="field has-addons">
                  <p class="control">
                    <label for="assetSel" class="is-sr-only">Asset</label>
                    <span class="select is-fullwidth">
                      <select id="assetSel" name="asset" v-model="assetStr">
                        <option v-for="b in balances[source]" :value="b.asset">{{ b.asset }}</option>
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
          </div>
        </section>
        <footer class="modal-card-foot">
          <button type="submit" class="button is-primary" :disabled="!amountValid">Transfer</button>
        </footer>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'internal_transfer_modal',
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
      return this.balances[this.source].find(a => a.asset === this.assetStr)
    },
    source() {
      return this.destination === 'SPOT' ? 'FUTURES' : 'SPOT'
    },
    amountValid() {
      if (!this.amountStr) return false
      const amount = this.parseBalance(this.amountStr);
      return this.amountStr && amount > 0 && this.asset.balance >= amount
    }
  },
  data() {
    return {
      destination: null,
      assetStr: null,
      amountStr: null
    }
  },
  created() {
    this.destination = Object.keys(this.balances)[0] === 'SPOT' ? 'FUTURES' : 'SPOT'
    this.assetStr = this.balances[this.source][0].asset
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
    destination() {
      this.assetStr = this.balances[this.source][0].asset
      this.amountStr = null
    },
    assetStr() {
      this.amountStr = null
    }
  }
}
</script>
