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
      <div>{{ topChain.name }}</div>
    </div>
    <div
      class="overflow-hidden whitespace-nowrap text-ellipsis text-gray-400 text-center text-[16px] font-light align-top leading-[50px] h-full mb-3"
    >
      <div>~${{ topChain.total }} value of {{ topChain.deal }} deals</div>
    </div>
  </BlockChart>
</template>
<script>
import BlockChart from "@/components/BlockChart.vue";
import { handlerPrice } from "@/ultis/number";
import moment from "moment";
export default {
  components: {
    BlockChart,
  },
  name: "trending-component",
  props: {
    api_res_data: { type: Array },
    title: { type: String },
    height: {
      type: String,
      default: "30px",
    },
  },
  data: () => ({
    topChain: {
      name: "...",
      total: 0,
      deal: 0,
    },
  }),
  watch: {
    api_res_data: {
      handler() {
        this.fetchData();
      },
    },
  },
  // created() {
  //   this.fetchData();
  // },
  computed: {},
  methods: {
    fetchData() {
      let result_crypto_fundraising = [];
      const res_data = this.api_res_data;
      let crypto_fundraising_data = res_data;
      result_crypto_fundraising = crypto_fundraising_data.reduce(function (
        accumulator,
        currentValue
      ) {
        let timestamp = currentValue.date;
        let time = moment(timestamp);
        time = new Date(
          Date.UTC(time.year(), time.month(), 1, 0, 0, 0)
        ).getTime();
        if (!accumulator[time]) {
          accumulator[time] = {
            time: time,
            chains: [],
          };
          result_crypto_fundraising.push([accumulator[time]]);
        }
        if (currentValue.chains.length > 0) {
          currentValue.chains.forEach((element) => {
            let total = currentValue.amount * 1000000;
            let index = accumulator[time].chains.findIndex(
              (x) => x.name == element
            );
            if (index == -1) {
              accumulator[time].chains.push({
                name: element,
                total: total,
                deal: 1,
              });
            } else {
              accumulator[time].chains[index].total += total;
              accumulator[time].chains[index].deal++;
            }
          });
        }
        return accumulator;
      },
      {});

      result_crypto_fundraising = Object.values(result_crypto_fundraising).sort(
        (a, b) => a.time - b.time
      );

      let presentMonth = result_crypto_fundraising.length - 1;
      let maxChainTotal = Math.max(
        ...result_crypto_fundraising[presentMonth].chains.map((o) => o.total)
      );
      this.topChain = result_crypto_fundraising[presentMonth].chains.filter(
        (obj) => {
          return obj.total == maxChainTotal;
        }
      )[0];
      this.topChain.total = handlerPrice(this.topChain.total);
    },
  },
};
</script>
