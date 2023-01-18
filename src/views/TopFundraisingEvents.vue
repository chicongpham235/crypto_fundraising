<template>
  <BlockChart class="h-full" :loading="loading">
    <template #title>
      <div class="w-full">
        <h2 class="text-white font-bold text-[20px] pt-[20px] pb-[20px]">
          Top 10 Fundraising Events
          <TimeFrame
            :time_frame="time_frame"
            :disabled="disabled"
            @set_time_frame="(v) => (time = v)"
          />
        </h2>
      </div>
    </template>

    <table
      class="w-full text-center table-fixed border-collapse"
      :style="{ minHeight: 500 + 'px', maxHeight: 500 + 'px' }"
    >
      <thead class="align-baseline text-white text-[15px]">
        <tr>
          <th class="w-[17%]">Project</th>
          <th class="w-[17%]">Value (USD)</th>
          <th class="w-[20%]">Round</th>
          <th class="w-[25%]">Investors</th>
          <th>Chain</th>
          <th>Category</th>
        </tr>
      </thead>
      <tbody
        class="align-baseline text-white text-[14px] overflow-hidden h-full"
      >
        <tr v-for="(item, index) in top_events" :key="index">
          <td
            :class="[index != top_events.length - 1 ? `border-b-2` : '']"
            class="overflow-hidden whitespace-nowrap text-ellipsis font-bold text-left border-[#535355ba] dark:border-slate-400/10 h-[10%] pt-[11.775px]"
          >
            {{ item.name }}
          </td>
          <td
            :class="[index != top_events.length - 1 ? `border-b-2` : '']"
            class="text-yellow-500 font-bold text-left border-[#535355ba] dark:border-slate-400/10 h-[10%] pt-[11.775px]"
          >
            <div class="grid grid-cols-12">
              <div class="col-span-3 ml-auto">
                ${{ handlerPrice(item.amount * 1000000) }}
              </div>
              <div class="col-span-1"></div>
              <div
                class="w-full h-2 bg-gray-500 rounded-full col-span-8 mt-[6px]"
              >
                <div
                  class="h-2 leading-none rounded-full bg-purple-blue"
                  :style="{ width: (item.amount / max_amount) * 100 + '%' }"
                ></div>
              </div>
            </div>
          </td>
          <td
            :class="[index != top_events.length - 1 ? `border-b-2` : '']"
            class="overflow-hidden whitespace-nowrap text-ellipsis text-gray-300 border-[#535355ba] dark:border-slate-400/10 h-[10%] pt-[11.775px]"
          >
            {{ item.round }}
          </td>
          <td
            :class="[index != top_events.length - 1 ? `border-b-2` : '']"
            class="overflow-hidden whitespace-nowrap text-ellipsis border-[#535355ba] dark:border-slate-400/10 h-[10%] pt-[11.775px]"
          >
            <a class="tooltip"
              >{{ item.investors }}
              <div class="tooltipInvestors">
                {{ item.investors }}
              </div></a
            >
          </td>
          <td
            :class="[index != top_events.length - 1 ? `border-b-2` : '']"
            class="overflow-hidden whitespace-nowrap border-[#535355ba] dark:border-slate-400/10 h-[10%] pt-[11.775px]"
          >
            <div
              class="w-[24px] h-[24px] inline-block"
              v-for="chain in chainsOne(item.chains)"
              :key="chain"
            >
              <a class="w-[24px] h-[24px] tooltip">
                <img
                  class="inline rounded-full"
                  :src="getSrcChain(chain)"
                  alt=""
                />
                <div class="tooltipContent">
                  {{ chain }}
                </div>
              </a>
            </div>
            <div
              v-if="lengthChainsTwo(item.chains) > 0"
              class="w-[24px] h-[24px] inline-block"
            >
              <button
                class="w-[24px] h-[24px] text-[10px] bg-gray-500 rounded-full tooltip"
              >
                +{{ lengthChainsTwo(item.chains) }}
                <div class="tooltipContent">
                  <span
                    v-for="chain in chainsTwo(item.chains)"
                    :key="chain"
                    class="tooltip"
                  >
                    <img
                      :src="getSrcChain(chain)"
                      class="rounded-full"
                      alt=""
                    />
                    <div class="tooltipContent">{{ chain }}</div>
                  </span>
                </div>
              </button>
            </div>
          </td>
          <td
            :class="[
              index != top_events.length - 1 ? `border-b-2` : 'h-[70px]',
            ]"
            class="overflow-hidden whitespace-nowrap text-ellipsis border-[#535355ba] dark:border-slate-400/10 h-[10%] pt-[11.775px]"
          >
            <a class="tooltip"
              >{{ item.category }}
              <div class="tooltipContent right-[2%]">
                {{ item.category }}
              </div>
            </a>
          </td>
        </tr>
      </tbody>
    </table>
  </BlockChart>
</template>
<script>
import BlockChart from "@/components/BlockChart.vue";
import TimeFrame from "@/components/TimeFrame.vue";
import moment from "moment";
import { handlerPrice } from "@/ultis/number";
export default {
  components: {
    BlockChart,
    TimeFrame,
  },
  name: "top-events",
  props: {
    api_res_data: { type: Array },
  },
  data: () => ({
    top_events: [],
    loading: true,
    max_amount: 0,
    handlerPrice,
    time_frame: ["1m", "3m", "1y", "YTD", "All"],
    time: "1m",
    disabled: true,
    chainsOne: (x) => {
      return x.slice(0, 3);
    },
    chainsTwo: (x) => {
      return x.slice(3);
    },
    lengthChainsTwo: (x) => {
      return x.length - x.slice(0, 3).length;
    },
  }),
  watch: {
    api_res_data: {
      handler() {
        this.fetchData();
      },
    },
    time: {
      handler() {
        this.setTimeFetch();
      },
    },
  },
  methods: {
    async fetchData() {
      try {
        this.disabled = true;
        this.loading = true;
        await this.setTimeFetch();
        this.disabled = false;
        this.loading = false;
      } catch (error) {
        console.error(error);
      }
    },
    getSrcChain(src) {
      return `https://icons.llamao.fi/icons/chains/rsz_${src.toLowerCase()}?w=24&h=24`;
    },
    async setTimeFetch() {
      const res_data = this.api_res_data;
      let data = res_data;
      let now = moment(new Date().getTime());
      let time = null;
      if (this.time == "1m")
        time = new Date(
          Date.UTC(now.year(), now.month(), 1, 0, 0, 0)
        ).getTime();
      if (this.time == "3m")
        time = new Date(
          Date.UTC(now.year(), now.month() - 3, now.date(), 0, 0, 0)
        ).getTime();
      if (this.time == "1y")
        time = new Date(
          Date.UTC(now.year() - 1, now.month(), now.date(), 0, 0, 0)
        ).getTime();
      if (this.time == "ytd")
        time = new Date(Date.UTC(now.year(), 0, 1, 0, 0, 0)).getTime();
      if (this.time == "all") time = new Date(time).getTime();
      data = data.filter((v) => !(v.date < time));
      data = data.sort((a, b) => b.amount - a.amount);
      data.map((v) => ({ ...v, investors: "", investorsArr: [] }));
      data.map((v) => {
        let investors = [];
        investors = investors.concat(v.leadInvestors);
        investors = investors.concat(v.otherInvestors);
        v.investorsArr = investors;
        v.investors = investors.join(", ");
        return v;
      });
      this.top_events = data.slice(0, 10);
      this.max_amount = this.top_events[0].amount;
    },
  },
};
</script>

<style scoped>
.tooltip .tooltipInvestors,
.tooltipContent {
  display: none;
}
.tooltip:hover .tooltipInvestors {
  position: absolute;
  display: flex;
  background-color: #1f2937;
  font-size: 14px;
  right: 25%;
  border-radius: 5px;
  border: solid;
  padding: 4px;
  color: rgb(156 163 175);
  white-space: initial;
  text-align: left;
}
.tooltip:hover .tooltipContent {
  position: absolute;
  display: flex;
  background-color: #1f2937;
  font-size: 14px;
  border-radius: 5px;
  border: solid;
  padding: 4px;
  color: rgb(156 163 175);
}
button:hover span.tooltip .tooltipContent {
  display: none;
}
button:hover span.tooltip:hover .tooltipContent {
  position: absolute;
  display: flex;
  background-color: #1f2937;
  font-size: 14px;
  border-radius: 5px;
  border: solid;
  padding: 4px;
  color: rgb(156 163 175);
}
</style>
