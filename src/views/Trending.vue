<template>
  <BlockChart>
    <div class="flex justify-center items-center mb-4" :style="{ height }">
      <!-- <div class="flex"> -->
      <div
        class="overflow-hidden whitespace-nowrap text-ellipsis text-white font-bold text-[20px]"
      >
        <slot name="title"> {{ title }} </slot>
      </div>
    </div>
    <div
      class="overflow-hidden whitespace-nowrap text-ellipsis text-white text-center text-[48px] font-bold align-top leading-[50px] h-full"
    >
      <div>{{ unitPrice }}{{ formatTotal }}</div>
    </div>
    <div
      class="overflow-hidden whitespace-nowrap text-ellipsis text-gray-400 text-center text-[16px] font-light align-top leading-[50px] h-full mb-3"
    >
      <span :style="style">{{ growth }}% </span>was {{ unitPrice
      }}{{ formatTotalLastMonth }} {{ unit }} last month
    </div>
  </BlockChart>
</template>
<script>
import BlockChart from "@/components/BlockChart.vue";
import { roundValue, handlerPrice } from "@/ultis/number";
export default {
  components: {
    BlockChart,
  },
  name: "trending-component",
  props: {
    title: { type: String },
    total: { type: Number },
    growth: { type: Number },
    totalLastMonth: { type: Number },
    unit: { type: String },
    unitPrice: { type: String },
    height: {
      type: String,
      default: "30px",
    },
  },
  data: (vm) => ({
    formatTotal: 0,
    formatTotalLastMonth: 0,
  }),
  watch: {
    total: {
      handler(val) {
        this.formatTotal = handlerPrice(val, 2);
      },
    },
    totalLastMonth: {
      handler(val) {
        this.formatTotalLastMonth = handlerPrice(val, 2);
      },
    },
  },
  created() {},
  computed: {
    style() {
      //#e85b5a
      let color = "#8dc950";
      if (this.growth && parseFloat(this.growth) < 0) {
        color = "#e85b5a";
      }
      return { color };
    },
  },
  methods: {},
};
</script>
