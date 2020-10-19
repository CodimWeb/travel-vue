<template>
  <vue-slider :max="max" :tooltip="'focus'" :tooltip-placement="'bottom'" v-model="price" />
</template>
 
<script>
import VueSlider from 'vue-slider-component'
import 'vue-slider-component/theme/antd.css'
 
export default {
  components: {
    VueSlider
  },
  props: ['max', 'value', 'getPrice'],
  data() {
    return {
      price: this.value,
    }
  },
  updated: function () {
    this.getPrice(this.price)
  }
}
</script>