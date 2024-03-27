<template>
  <div class="modal" id="deposit-modal">
    <div class="modal-background"></div>
    <div class="modal-card">
      <header class="modal-card-head">
        <p class="modal-card-title">Deposit</p>
        <button class="delete" aria-label="close"></button>
      </header>
      <section class="modal-card-body">
        <p>
          Please choose the asset to deposit (and the network if required) to get your deposit address.
        </p>
        <div class="mt-4">
          <div class="field is-grouped mt-5">
            <div class="control is-expanded">
              <label for="assetSel" class="is-sr-only">Asset</label>
              <span class="select is-fullwidth">
                <select id="assetSel" name="asset" v-model="assetStr">
                  <option v-for="a in assets" :value="a">{{ a }}</option>
                </select>
              </span>
            </div>

            <template v-if="assetStr === 'USDT'">
              <div class="control">
                <label class="label is-sr-only" for="networkSel">Network</label>
                <span class="select">
                    <select id="networkSel" name="network" v-model="networkStr">
                      <option value="ETH">ERC20</option>
                      <option value="TRX">TRC20</option>
                    </select>
                </span>
              </div>
            </template>
          </div>
        </div>

        <div class="has-text-centered is-justify-content-center mt-5">
          <template v-if="address">
            <div>
              <div class="is-inline-block has-background-white is-rounded p-2 mt-2" style="border-radius: 4px; max-width: 170px">
                <qrcode-vue :value="address" level="H" :size="150" class-name="is-flex is-align-items-center is-justify-content-center"></qrcode-vue>
                <span class="is-size-5-desktop is-small is-block has-text-black-ter mt-2" style="font-size: .8rem; word-break: break-all; width: 100%">{{ this.address }}</span>
              </div>
              <div class="mt-4 is-small">
                Send only <strong>{{ this.shownAsset }}</strong>
                to this address.<br>
                Sending any other coins may result in permanent loss.
              </div>
            </div>
          </template>
          <button type="submit" class="button is-primary is-center" v-else @click="loadAddress">Get Deposit Address</button>
        </div>
      </section>
    </div>
  </div>
</template>

<script>
import QrcodeVue from 'qrcode.vue'

export default {
  name: 'withdraw_modal',
  components: { QrcodeVue },
  props: {
    assets: {
      type: Array,
      default: () => ['BTC', 'USDT']
    },
    email: {
      type: String,
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
    shownAsset() {
      return this.assetStr + (this.networkStr ? (' (' + {'ETH': 'ERC20', 'TRX': 'TRC20'}[this.networkStr] + ')') : '')
    }
  },
  data() {
    return {
      assetStr: null,
      networkStr: null,

      address: null
    }
  },
  created() {
    this.assetStr = this.assets[0]
  },
  methods: {
    async loadAddress() {
      const query = {
        email: this.email,
        asset: this.assetStr,
        csrfToken: this.csrfToken
      };

      if (this.networkStr != null) {
        query['network'] = this.networkStr
      }

      this.address = await (await fetch('/u/api/depositAddr?' + new URLSearchParams(query))).text()
    }
  },
  watch: {
    networkStr() {
      this.address = null
    },
    assetStr(val) {
      this.address = null

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