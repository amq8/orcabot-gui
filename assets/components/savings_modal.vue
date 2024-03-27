<template>
  <div class="modal" id="savings-modal" ref="modal">
    <div class="modal-background"></div>
    <div class="modal-card modal-card-large">
      <header class="modal-card-head">
        <p class="modal-card-title">Savings</p>
        <button class="delete" aria-label="close"></button>
      </header>
      <section class="modal-card-body py-0">
        <div class="notification is-danger my-2" v-if="error">
          {{ error }}
        </div>
        <template v-if="!loading">
          <template v-if="accountData">
            <div>
              <strong>Total Balance ≈</strong>
              <span class="is-inline-block mx-1">
                <code>{{accountData.totalAmountInBTC}}BTC <small class="is-muted">≈ {{accountData.totalAmountInUSDT}}USDT</small></code>
              </span>
            </div>
            <div class="mt-3 table-container">
              <table class="table has-background-black-ter" style="width: 100%">
                <thead>
                <tr>
                  <th>Position</th>
                  <th>Total Interest</th>
                  <th>Avg. Annual Interest Rate</th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="balance in savingsAssets" :key="balance.asset">
                  <td style="vertical-align: middle">
                    <crypto-icon :name="balance.asset" style="margin-top: -3px" />
                    <code class="is-inline-block ml-1">
                      {{ balance.amount }}{{ balance.asset }}
                      <small class="is-muted">≈ {{balance.amountInUSDT}}USDT</small>
                    </code>
                    <div class="tag is-inline-block mt-1 is-dark" v-if="getRedeemingAmount(balance.asset) > 0">
                      <code class="is-small" style="font-size: .9rem">
                        {{ getRedeemingAmount(balance.asset) }}{{balance.asset}}
                      </code> pending redeem
                    </div>
                  </td>
                  <template v-if="positionDetails[balance.asset]">
                    <td style="vertical-align: middle">
                      <code>{{ positionDetails[balance.asset].totalInterest }}{{ balance.asset }}</code>
                    </td>
                    <td style="vertical-align: middle">
                      <code>{{ positionDetails[balance.asset].avgAnnualInterestRate }}%</code>
                    </td>
                  </template>
                  <template v-else>
                    <td colspan="2" style="vertical-align: middle">
                      <div class="loading-spinner"></div>
                    </td>
                  </template>
                </tr>
                </tbody>
              </table>
            </div>

            <div class="mt-5 mb-3">
              <strong class="is-block is-size-6 mb-1">Purchase / Redeem</strong>
              <form method="POST" action="/u/api/savings/transfer">
                <input type="hidden" name="csrfToken" :value="this.csrfToken">
                <div class="field is-horizontal">
                  <div class="field-body">
                    <div class="field is-flex-grow-0">
                      <div class="control">
                        <span class="select">
                          <select id="savings-mode-select" name="transferMode" v-model="transferMode">
                            <option value="purchase">Purchase (Spot ➜ Savings)</option>
                            <option value="redeem">Redeem (Savings ➜ Spot)</option>
                          </select>
                        </span>
                      </div>
                    </div>
                    <div class="field" v-if="transferMode === 'redeem'">
                      <div class="control">
                        <span class="select">
                          <select id="savings-redemption-type-select" name="redemptionType">
                            <option value="FAST" selected>Fast</option>
                            <option value="NORMAL">Normal</option>
                          </select>
                        </span>
                      </div>
                    </div>

                  </div>
                </div>
                <div v-if="transferMode === 'redeem'" class="mb-3">
                  <small>
                    <strong>Normal: </strong> 1 working day, gives interest for the current day<br>
                    <strong>Fast: </strong> same day (usually instant), no interest for current day, subject to a daily quota
                  </small>
                </div>
                <div class="field is-horizontal">
                  <div class="field-body">
                    <div class="field has-addons">
                      <p class="control">
                        <span class="select is-fullwidth">
                          <select id="savings-asset-select" name="transferAsset" v-model="transferAsset">
                            <option v-for="asset in transferableAssets" :value="asset">{{ asset }}</option>
                          </select>
                        </span>
                      </p>
                      <p class="control is-expanded">
                        <input class="input" type="text" pattern="\d+(\.\d{1,8})?"
                               :class="{'is-danger': transferAmount && !amountValid}"
                               id="savings-amount-input" name="transferAmount" placeholder="Amount"
                               v-model="transferAmount" required>
                      </p>
                    </div>
                    <div class="field">
                      <button type="submit" class="button is-block is-primary" :disabled="!valid">
                        {{ transferMode === 'purchase' ? 'Purchase' : 'Redeem' }}
                      </button>
                    </div>
                  </div>
                </div>
                <div v-if="transferAsset" class="is-flex is-align-items-center">
                  <span>Available: <code>{{ maxAmountForSelectionStr }}{{ transferAsset }}</code></span>

                  <button type="button" class="button is-dark is-small ml-2" :disabled="!transferAsset"
                          @click="transferAmount = maxAmountForSelectionStr">
                    Max
                  </button>
                </div>
              </form>
            </div>
          </template>
        </template>
        <template v-else>
          Loading...
        </template>
      </section>
      <footer class="modal-card-foot">

      </footer>
    </div>
  </div>
</template>

<script>
import CryptoIcon from "./crypto_icon";

export default {
  name: 'SavingsModal',
  components: {CryptoIcon},
  props: {
    autoOpen: {
      type: Boolean,
      default: false,
    },
    spotBalances: {
      type: Array,
      required: true
    },
    csrfToken: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      loading: true,
      error: null,
      accountData: {},
      positionDetails: {},

      transferMode: 'purchase',
      transferAsset: null,
      transferAmount: null,
    }
  },
  computed: {
    savingsAssets() {
      return this.accountData?.positionAmountVos?.filter(p => Number(p.amount) > 0) ?? []
    },
    transferableAssets() {
      if (this.transferMode === 'purchase') {
        return this.spotBalances.map(b => b.asset)
      } else {
        return this.savingsAssets.map(b => b.asset)
      }
    },
    maxAmountForSelectionStr() {
      if (this.transferMode === 'purchase') {
        return this.formatBalance(this.spotBalances?.find(b => b.asset === this.transferAsset)?.balance)
      } else {
        return this.savingsAssets.find(p => p.asset === this.transferAsset)?.amount
      }
    },
    amountValid() {
      if (!this.transferAmount) return false
      const amount = this.parseBalance(this.transferAmount);
      return this.transferAmount && amount > 0 && this.parseBalance(this.maxAmountForSelectionStr) >= amount
    },
    valid () {
      return this.transferMode && this.transferAsset && this.amountValid
    }
  },
  created() {
    void this.loadAccount()
  },
  mounted() {
    if (this.autoOpen) {
      this.$refs.modal.classList.add('is-active')
    }
  },
  methods: {
    async loadAccount() {
      try {
        this.loading = true
        this.error = null
        const data = await (await fetch('/u/api/savings/account')).json()
        if (data.error) {
          this.error = data.error
          return
        }
        this.accountData = data
        Promise.all(this.savingsAssets.map(p => this.loadDetails(p.asset))).then(() => {})
      } catch (err) {
        console.dir(err)
        this.error = 'An error occurred, please try again later'
      } finally {
        this.loading = false
      }
    },
    async loadDetails(asset) {
      try {
        this.error = null
        const data = await (await fetch('/u/api/savings/position/' + asset)).json()
        if (data.error) {
          this.error = data.error
          return
        }
        this.$set(this.positionDetails, asset, data)
      } catch (err) {
        console.dir(err)
        this.error = 'An error occurred, please try again later'
      }
    },
    getRedeemingAmount(asset) {
      if (!this.positionDetails[asset]?.redeemingAmount) {
        return null
      }
      return parseFloat(this.positionDetails[asset]?.redeemingAmount)
    },
    formatBalance(balance) {
      return (Number(balance) / Math.pow(10, 8)).toFixed(8)
    },
    parseBalance(balanceStr) {
      const split = balanceStr.split(/,\./)
      return Number(split[0]) * Math.pow(10, 8) + (split[1] ? Number(split[1]) : 0)
    }
  },
  watch: {
    transferMode() {
      this.transferAsset = null
      this.transferAmount = null
    }
  }
}
</script>

<style scoped>

</style>