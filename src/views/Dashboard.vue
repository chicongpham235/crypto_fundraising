<template>
  <div class="bg-[#16151a] w-full">
    <div class="grid grid-cols-12 gap-4 mb-4">
      <div class="xl:col-span-3 lg:col-span-3 md:col-span-12">
        <Trending
          class="gap-4 mb-4"
          :total="funding_rounds.total"
          :totalLastMonth="funding_rounds.totalLastMonth"
          :unit="funding_rounds.unit"
          :unitPrice="funding_rounds.unitPrice"
          :growth="funding_rounds.growth"
          title="Total Funding Rounds"
        />
        <Trending
          class="gap-4 mb-4"
          :total="funding_amount.total"
          :totalLastMonth="funding_amount.totalLastMonth"
          :unit="funding_amount.unit"
          :unitPrice="funding_amount.unitPrice"
          :growth="funding_amount.growth"
          title="Total Funding Amount"
        />
        <TopChain class="gap-4" :api_res_data="res_data" title="Top Chain" />
      </div>
      <div class="xl:col-span-9 lg:col-span-9 md:col-span-12">
        <CryptoFundraisingTrend
          :api_res_data="res_data"
          @funding_rounds="(v) => (funding_rounds = v)"
          @funding_amount="(v) => (funding_amount = v)"
        />
      </div>
    </div>
    <div class="grid grid-cols-12 gap-4 mb-4">
      <div class="xl:col-span-6 lg:col-span-6 md:col-span-12">
        <FundraisingByCategory :api_res_data="res_data" />
      </div>
      <div class="xl:col-span-6 lg:col-span-6 md:col-span-12">
        <FundRaisingRoundsByStage :api_res_data="res_data" />
      </div>
    </div>
    <div class="grid grid-cols-12 gap-4 mb-4">
      <div class="xl:col-span-12 lg:col-span-12 md:col-span-12">
        <TopFundraisingEvents :api_res_data="res_data" />
      </div>
    </div>
  </div>
</template>
<script>
import CryptoFundraisingTrend from "./CryptoFundraisingTrend.vue";
import Trending from "./Trending.vue";
import TopChain from "./TopChain.vue";
import FundraisingByCategory from "./FundraisingByCategory.vue";
import { FUNDING_API } from "@/services/api";
import FundRaisingRoundsByStage from "./FundRaisingRoundsByStage.vue";
import TopFundraisingEvents from "./TopFundraisingEvents.vue";
export default {
  components: {
    CryptoFundraisingTrend,
    Trending,
    TopChain,
    FundraisingByCategory,
    FundRaisingRoundsByStage,
    TopFundraisingEvents,
  },
  name: "dash-board",
  data: () => ({
    funding_rounds: {
      total: 0,
      growth: 0,
      totalLastMonth: 0,
      unit: "rounds",
      unitPrice: "",
    },
    funding_amount: {
      total: 0,
      growth: 0,
      totalLastMonth: 0,
      unit: "",
      unitPrice: "$",
    },
    res_data: [],
  }),
  created() {
    this.fetchData();
  },
  methods: {
    async fetchData() {
      try {
        const res = await FUNDING_API();
        this.res_data =
          res.data.map((v) => {
            v.date = v.date * 1000;
            return v;
          }) ?? [];
      } catch (error) {
        console.error(error);
      }
    },
  },
};
</script>
