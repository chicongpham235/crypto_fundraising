<template>
  <BlockChart :loading="loading">
    <template #title>
      <div class="w-full">
        <h2 class="text-white font-bold text-[20px]">
          Number of Fundraising by Category
          <TimeFrame
            class="absolute top-[2%] right-0 z-1"
            :time_frame="time_frame"
            @set_time_frame="(v) => (time = v)"
          />
        </h2>
      </div>
    </template>
    <HighChart
      id="fundraising-by-category"
      :options="options"
      height="600px"
      ref="highchart"
      isStockChart
    />
  </BlockChart>
</template>

<script>
import moment from "moment";
import { getQuarterFromMonth, handlerPrice, roundValue } from "@/ultis";
import BlockChart from "@/components/BlockChart.vue";
import HighChart from "@/components/HighChart.vue";
import TimeFrame from "@/components/TimeFrame.vue";
let highcharts = require("highcharts");
require("highcharts/highcharts-more")(highcharts);
require("highcharts/modules/accessibility")(highcharts);
require("highcharts/modules/exporting")(highcharts);
var groupingUnits = [["month", [1]]];
export default {
  components: {
    BlockChart,
    HighChart,
    TimeFrame,
  },
  name: "fundraising-by-category",
  props: {
    api_res_data: { type: Array },
  },
  data: () => ({
    data: [],
    time_frame: ["Monthly", "Quarterly"],
    time: "monthly",
    web3: {
      name: "Web3",
      data: [],
      color: "#7c7cf4",
      dataGrouping: {
        units: groupingUnits,
      },
    },
    nfts: {
      name: "NFTs",
      data: [],
      color: "#7F00FF",
      dataGrouping: {
        units: groupingUnits,
      },
    },
    gamefi: {
      name: "GameFi",
      data: [],
      color: "#DA70D6",
      dataGrouping: {
        units: groupingUnits,
      },
    },
    defi: {
      name: "DeFi",
      data: [],
      color: "#8ab4f8",
      dataGrouping: {
        units: groupingUnits,
      },
    },
    cefi: {
      name: "CeFi",
      data: [],
      color: "#01bbbb",
      dataGrouping: {
        units: groupingUnits,
      },
    },
    blockchain: {
      name: "Blockchain Service",
      data: [],
      color: "#6495ED",
      dataGrouping: {
        units: groupingUnits,
      },
    },
    infrastructure: {
      name: "Infrastructure",
      data: [],
      color: "#3dca3d",
      dataGrouping: {
        units: groupingUnits,
      },
    },
    currency: {
      name: "Currency",
      data: [],
      color: "#aecbfa",
      dataGrouping: {
        units: groupingUnits,
      },
    },
    social: {
      name: "Social",
      data: [],
      color: "rgb(234 179 8)",
      dataGrouping: {
        units: groupingUnits,
      },
    },
    result: [],
    seriesOptions: [],
    tickInterval: 1 * 30 * 24 * 3600 * 1000,
    loading: true,
    legendX: 0,
    legendY: 0,
    verticalAlign: "top",
    labelFormat: "{value:%b, %y}",
    roundValue,
  }),
  watch: {
    api_res_data: {
      handler() {
        this.init();
      },
    },
    time: {
      async handler() {
        this.loading = true;
        await this.setDataChart();
        this.loading = false;
      },
    },
  },
  computed: {
    options(vm) {
      return {
        chart: {
          backgroundColor: "#2e2e33",
          type: "column",
        },
        rangeSelector: {
          enabled: false,
        },
        title: {
          text: null,
          // style: {
          //   color: "transparent",
          //   fontSize: "20px",
          //   fontWeight: "bold",
          // },
          // align: "left",
        },
        xAxis: {
          title: null,
          type: "datetime",
          labels: {
            format: this.labelFormat,
            style: {
              color: "rgb(156 163 175)",
            },
          },
          crosshair: {
            color: "#ffffff1f",
            snap: false,
            dashStyle: "longdash",
          },
          tickInterval: this.tickInterval,
        },
        yAxis: [
          {
            gridLineColor: "#ffffff1f",
            gridLineDashStyle: "longdash",
            labels: {
              formatter: function () {
                return handlerPrice(this.value) + "%";
              },
              style: {
                color: "rgb(156 163 175)",
              },
            },
            title: null,
            opposite: false,
            tickAmount: 5,
            showLastLabel: true,
          },
          {
            // Secondary yAxis
            gridLineColor: "#ffffff1f",
            gridLineDashStyle: "longdash",
            title: null,
            labels: {
              style: {
                color: "rgb(156 163 175)",
              },
            },
            opposite: true,
            crosshair: {
              color: "#ffffff1f",
              snap: false,
              dashStyle: "longdash",
            },
            showLastLabel: true,
          },
        ],
        credits: {
          enabled: false,
        },
        legend: {
          enabled: true,
          align: "left",
          verticalAlign: this.verticalAlign,
          x: this.legendX,
          y: this.legendY,
          itemMarginBottom: 4,
          itemStyle: {
            color: "rgb(148 163 184)",
            fontWeight: "normal",
          },
          itemHoverStyle: {
            color: "rgb(148 163 184)",
            fontWeight: "bold",
          },
        },
        tooltip: {
          useHTML: true,
          backgroundColor: "#1f2937",
          followPointer: true,
          split: false,
          followTouchMove: false,
          animation: false,
          borderWidth: 0,
          shadow: false,
          padding: 0,
          hideDelay: 0,
          shared: true,
          formatter: function () {
            let points = this.points;
            let title = "";
            let content = [];
            if (vm.time == "monthly")
              title = `<div class="text-gray-500 pb-1">Month: <b class="text-white font-semibold">${moment(
                this.x
              ).format("MMM, YYYY")}</b></div>`;
            if (vm.time == "quarterly")
              title = `<div class="text-gray-500 pb-1">Quarter: <b class="text-white font-semibold">Quarter ${
                getQuarterFromMonth(moment(this.x).month()) + 1
              }, ${moment(this.x).year()}</b></div>`;
            points.forEach((point) => {
              content.push(`<div style="color:${point.color}">${
                point.series.name
              }: <b class="text-gray-400 font-semibold"> ${roundValue(
                point.percentage,
                2
              )}%
               </b> </div>`);
            });
            content = content.join("");
            let result =
              `<div class="border border-white rounded p-2">` +
              title +
              content +
              `</div>`;
            return result;
          },
        },
        plotOptions: {
          column: {
            maxPointWidth: 20,
            stacking: "percent",
            states: {
              inactive: {
                opacity: 1,
              },
            },
          },
        },
        series: this.seriesOptions,
        navigator: {
          series: {
            type: "line",
          },
        },
      };
    },
  },
  methods: {
    async init() {
      try {
        this.loading = true;
        await this.fetchData();
      } catch (error) {
        console.log(error);
      } finally {
        this.loading = false;
      }
    },
    async fetchData() {
      this.setDataChart();
    },

    async setDataChart() {
      this.result = [];
      const res_data = this.api_res_data;
      let data = res_data;
      let vm = this;
      this.result = data.reduce(function (accumulator, currentValue) {
        let timestamp = moment(currentValue.date);
        let time = null;
        if (vm.time == "monthly") {
          time = new Date(
            Date.UTC(timestamp.year(), timestamp.month(), 1, 0, 0, 0)
          ).getTime();
        }
        if (vm.time == "quarterly") {
          let quarter = Math.floor(timestamp.month() / 3) * 3;
          time = new Date(
            Date.UTC(timestamp.year(), quarter, 1, 0, 0, 0)
          ).getTime();
        }
        if (!accumulator[time]) {
          accumulator[time] = {
            time: time,
            categoryArr: [],
            sectorArr: [],
            category: [
              { name: "Infrastructure", count: 0 },
              { name: "Blockchain Service", count: 0 },
              { name: "CeFi", count: 0 },
              { name: "DeFi", count: 0 },
              { name: "GameFi", count: 0 },
              { name: "NFTs", count: 0 },
              { name: "Web3", count: 0 },
              { name: "Currency", count: 0 },
              { name: "Social", count: 0 },
            ],
          };
          vm.result.push([accumulator[time]]);
        }
        accumulator[time].categoryArr = accumulator[time].categoryArr.concat(
          currentValue.category
        );
        accumulator[time].sectorArr = accumulator[time].sectorArr.concat(
          currentValue.sector
        );
        return accumulator;
      }, {});
      this.result = Object.values(this.result).sort((a, b) => a.time - b.time);
      this.mapCategory(this.result);

      this.nfts.data = this.result.map((v) => {
        let index = v.category.findIndex((x) => x.name == "NFTs");
        if (index != -1) return [v.time, v.category[index].count];
      });
      this.defi.data = this.result.map((v) => {
        let index = v.category.findIndex((x) => x.name == "DeFi");
        if (index != -1) return [v.time, v.category[index].count];
      });
      this.cefi.data = this.result.map((v) => {
        let index = v.category.findIndex((x) => x.name == "CeFi");
        if (index != -1) return [v.time, v.category[index].count];
      });
      this.infrastructure.data = this.result.map((v) => {
        let index = v.category.findIndex((x) => x.name == "Infrastructure");
        if (index != -1) return [v.time, v.category[index].count];
      });
      this.web3.data = this.result.map((v) => {
        let index = v.category.findIndex((x) => x.name == "Web3");
        if (index != -1) return [v.time, v.category[index].count];
      });
      this.blockchain.data = this.result.map((v) => {
        let index = v.category.findIndex((x) => x.name == "Blockchain Service");
        if (index != -1) return [v.time, v.category[index].count];
      });
      this.currency.data = this.result.map((v) => {
        let index = v.category.findIndex((x) => x.name == "Currency");
        if (index != -1) return [v.time, v.category[index].count];
      });
      this.social.data = this.result.map((v) => {
        let index = v.category.findIndex((x) => x.name == "Social");
        if (index != -1) return [v.time, v.category[index].count];
      });
      this.gamefi.data = this.result.map((v) => {
        let index = v.category.findIndex((x) => x.name == "GameFi");
        if (index != -1) return [v.time, v.category[index].count];
      });
      this.seriesOptions = [
        this.infrastructure,
        this.blockchain,
        this.cefi,
        this.defi,
        this.gamefi,
        this.nfts,
        this.web3,
        this.currency,
        this.social,
      ];
      if (this.time == "monthly") {
        this.labelFormat = "{value:%b, %y}";
        this.tickInterval = 1 * 30 * 24 * 3600 * 1000;
      }
      if (this.time == "quarterly") {
        this.labelFormat = "{value:Q%q, %y}";
        this.tickInterval = 3 * 30 * 24 * 3600 * 1000;
      }
    },

    mapCategory(e) {
      e = e.map((v) => {
        if (
          v.categoryArr.includes("L1") ||
          v.categoryArr.includes("L2") ||
          v.categoryArr.includes("Infrastructure") ||
          v.categoryArr.includes("Smart Contract Platform") ||
          v.categoryArr.includes("Zero Knowledge Industry")
        ) {
          v.category[0].count++;
        }
        if (
          v.categoryArr.includes("Blockchain-as-a-Service") ||
          v.categoryArr.includes("Cloud Intelligence") ||
          v.categoryArr.includes("Custody") ||
          v.categoryArr.includes("Cybersecurity") ||
          v.categoryArr.includes("Information security") ||
          v.categoryArr.includes("Information services digital assets") ||
          v.categoryArr.includes("IoT") ||
          v.categoryArr.includes("MEV") ||
          v.categoryArr.includes("Mining") ||
          v.categoryArr.includes("Scaling Solution") ||
          v.categoryArr.includes("Smart contract audits") ||
          v.categoryArr.includes("Smart contract security") ||
          v.categoryArr.includes("Storage")
        ) {
          v.category[1].count++;
        }
        if (
          v.categoryArr.includes("CEX") ||
          v.categoryArr.includes("CeFi") ||
          v.categoryArr.includes("CeFi Yield") ||
          v.categoryArr.includes("Centralized Exchange")
        ) {
          v.category[2].count++;
        }
        if (
          v.categoryArr.includes("DEX") ||
          v.categoryArr.includes("DeFi") ||
          v.categoryArr.includes("DAO Infrastructure")
        ) {
          v.category[3].count++;
        }
        if (
          v.categoryArr.includes("Game") ||
          v.categoryArr.includes("Gaming") ||
          v.categoryArr.includes("Metaverse")
        ) {
          v.category[4].count++;
        }
        if (v.categoryArr.includes("NFT")) v.category[5].count++;
        if (v.categoryArr.includes("Web3")) v.category[6].count++;
        if (v.categoryArr.includes("Payments")) v.category[7].count++;
        if (v.categoryArr.includes("Social Platform")) v.category[8].count++;
        if (v.categoryArr.includes(null)) {
          v.sectorArr.forEach((e) => {
            if (
              ["layer 1", "layer1", "dao"].some((x) =>
                e?.toLowerCase().includes(x)
              )
            )
              v.category[0].count++;
            else if (
              [
                "sass",
                "data",
                "management",
                "analytics",
                "solution",
                "solutions",
              ].some((x) => e?.toLowerCase().includes(x))
            )
              v.category[1].count++;
            else if (
              ["market maker", "exchange"].some((x) => {
                if (!e?.toLowerCase().includes("decentralized"))
                  return e?.toLowerCase().includes(x);
              })
            )
              v.category[2].count++;
            else if (
              [
                "decentralized",
                "wallet",
                "amm",
                "liquidity",
                "swap",
                "leveraged",
                "lending",
                "borrowing",
                "money",
              ].some((x) => e?.toLowerCase().includes(x))
            )
              v.category[3].count++;
            else if (["play"].some((x) => e?.toLowerCase().includes(x)))
              v.category[4].count++;
            else if (["web3"].some((x) => e?.toLowerCase().includes(x)))
              v.category[5].count++;
            else if (["nft"].some((x) => e?.toLowerCase().includes(x)))
              v.category[5].count++;
            else if (["fiat"].some((x) => e?.toLowerCase().includes(x)))
              v.category[7].count++;
            else if (
              ["social", "network"].some((x) => e?.toLowerCase().includes(x))
            )
              v.category[8].count++;
          });
        }
        return v;
      });
    },
  },
};
</script>
