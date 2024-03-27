<template>
  <span style="display: inline-block; vertical-align: middle; height: 24px">
    <span class="icon" v-if="iconSvg">
      <img :src="iconSvg" :alt="name.toUpperCase()" />
    </span>
    <span v-else>
      {{ name.toUpperCase() }}
    </span>
  </span>
</template>

<script>
export default {
  name: 'CryptoIcon',
  props: {
    name: {
      required: true,
      type: String,
    }
  },
  computed: {
    iconSvg() {
      return require(`cryptocurrency-icons/svg/icon/${this.name.toLowerCase()}.svg`)
    }
  }
}
</script>
