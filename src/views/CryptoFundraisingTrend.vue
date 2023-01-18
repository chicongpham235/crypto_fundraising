<template>
  <BlockChart :loading="loading">
    <template #title>
      <div class="w-full">
        <h2 class="text-white font-bold text-[20px]">
          Crypto Fundraising Trend
          <TimeFrame
            class="relative z-[1] top-[41px]"
            :time_frame="time_frame"
            @set_time_frame="(v) => (time = v)"
          />
        </h2>
      </div>
    </template>
    <HighChart
      id="crypto-fundraising-trend"
      :options="options"
      height="556px"
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
  name: "crypto-fundraising-trend",
  components: {
    BlockChart,
    HighChart,
    TimeFrame,
  },
  props: {
    api_res_data: { type: Array },
  },
  data: () => ({
    data: [],
    result: [],
    time_frame: ["Monthly", "Quarterly"],
    time: "monthly",
    labelFormat: "{value:%b, %y}",
    symbol: {},
    interval: null,
    loading: true,
    functionFireChart: null,
    yCategories: [],
    seriesOptions: [],
    legendX: 0,
    legendY: -30,
    verticalAlign: "top",
    tickInterval: 1 * 30 * 24 * 3600 * 1000,
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
          inputEnabled: true,
          inputDateFormat: "%e.%m.%Y",
          inputPosition: {
            align: "center",
          },
          enabled: true,
          selected: 5,
          height: 45,
          buttons: [
            {
              type: "month",
              count: 6,
              text: "6m",
            },
            {
              type: "year",
              count: 1,
              text: "1y",
            },
            {
              type: "year",
              count: 2,
              text: "2y",
            },
            {
              type: "ytd",
              text: "YTD",
            },
            {
              type: "all",
              text: "All",
            },
          ],
          buttonTheme: {
            fill: "none",
            stroke: "none",
            "stroke-width": 0,
            r: 8,
            style: {
              color: "#fff",
              fontWeight: "normal",
            },
            states: {
              hover: {
                fill: "#404044",
              },
              select: {
                fill: "#404044",
                style: {
                  color: "white",
                  fontWeight: "normal",
                },
              },
              disabled: { fontWeight: "normal" },
            },
          },
          inputStyle: {
            color: "#fff",
            fontWeight: "normal",
            fontFamily: "'Rubik' sans-serif",
          },
          labelStyle: {
            color: "#fff",
            fontWeight: "normal",
          },
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
            // gridLineDashStyle: "longdash",
            labels: {
              formatter: function () {
                return "$" + handlerPrice(this.value);
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
            // gridLineDashStyle: "longdash",
            gridLineWidth: 0,
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
          align: "center",
          verticalAlign: this.verticalAlign,
          x: this.legendX,
          y: this.legendY,
          itemStyle: {
            color: "#fff",
            fontWeight: "normal",
          },
          itemHoverStyle: {
            color: "#fff",
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
          borderRadius: 5,
          borderWidth: 0,
          shadow: false,
          padding: 0,
          hideDelay: 0,
          formatter: function () {
            let list = this.point.custom;
            if (vm.time == "monthly") {
              let content = `<div class="border border-white rounded p-2">
                <div class="text-gray-500">Month: <b class="text-white font-semibold">${moment(
                  list.time
                ).format("MMM, YYYY")}</b></div>
                <div class="text-gray-500">Raised: <b class="text-white font-semibold">$ ${handlerPrice(
                  list.amount,
                  2
                )}</b></div>
                <div class="text-gray-500">Number of fundraising rounds: <b class="text-white font-semibold">${
                  list.countRound
                }</b></div>
                </div>`;
              return content;
            }
            if (vm.time == "quarterly") {
              let content = `<div class="border border-white rounded p-2">
                <div class="text-gray-500">Quarter: <b class="text-white font-semibold">Quarter ${
                  getQuarterFromMonth(moment(list.time).month()) + 1
                }, ${moment(list.time).year()}</b></div>
                <div class="text-gray-500">Raised: <b class="text-white font-semibold">$ ${handlerPrice(
                  list.amount,
                  2
                )}</b></div>
                <div class="text-gray-500">Number of fundraising rounds: <b class="text-white font-semibold">${
                  list.countRound
                }</b></div>
                </div>`;
              return content;
            }
          },
        },
        plotOptions: {
          column: {
            maxPointWidth: 20,
            stacking: "normal",
            states: {
              inactive: {
                opacity: 1,
              },
            },
          },
        },
        series: this.seriesOptions,
      };
    },
  },
  methods: {
    async init() {
      let amount = {
        name: "Funding Amount",
        data: [],
        yAxis: 0,
        color: "rgb(97, 100, 255)",
        enableMouseTracking: false,
        borderWidth: 0,
        dataGrouping: {
          units: groupingUnits,
        },
      };
      let rounds = {
        name: "Number of Funding Rounds",
        data: [],
        type: "spline",
        color: "#f2ba2a",
        yAxis: 1,
        marker: {
          enabled: false,
        },
        dataGrouping: {
          units: groupingUnits,
        },
      };
      this.seriesOptions = [amount, rounds];
      try {
        this.loading = true;
        await this.fetchData();
      } catch (error) {
        console.log(error);
        this.seriesOptions = [amount, rounds];
      } finally {
        this.loading = false;
      }
    },
    async fetchData() {
      const res_data = this.api_res_data;
      this.data = res_data;
      await this.setDataChart();
      let fundingRounds = {
        total: 0,
        growth: 0,
        totalLastMonth: 0,
        unit: "rounds",
        unitPrice: "",
      };
      let fundingAmount = {
        total: 0,
        growth: 0,
        totalLastMonth: 0,
        unit: "",
        unitPrice: "$",
      };
      let lastMonth = this.result.length - 2;
      let presentMonth = this.result.length - 1;
      fundingRounds.total = this.result[presentMonth].countRound;
      fundingRounds.totalLastMonth = this.result[lastMonth].countRound;
      let growth = (a, b) => {
        let temp = a / b;
        return temp - 1;
      };
      fundingRounds.growth = roundValue(
        growth(fundingRounds.total, fundingRounds.totalLastMonth) * 100,
        0
      );
      fundingAmount.total = this.result[presentMonth].amount * 1000000;
      fundingAmount.totalLastMonth = this.result[lastMonth].amount * 1000000;
      fundingAmount.growth = roundValue(
        growth(fundingAmount.total, fundingAmount.totalLastMonth) * 100,
        0
      );
      this.$emit("funding_rounds", fundingRounds);
      this.$emit("funding_amount", fundingAmount);
    },

    async setDataChart() {
      let vm = this;
      this.result = [];
      this.result = this.data.reduce(function (accumulator, currentValue) {
        let timestamp = moment(currentValue.date);
        let time = null;
        if (vm.time == "monthly") {
          time = new Date(
            Date.UTC(timestamp.year(), timestamp.month(), 1, 7, 0, 0)
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
            amount: 0,
            countRound: 0,
          };
          vm.result.push([accumulator[time]]);
        }
        accumulator[time].amount += currentValue.amount;
        if (currentValue.round) accumulator[time].countRound += 1;
        return accumulator;
      }, {});
      this.result = Object.values(this.result).sort((a, b) => a.time - b.time);

      //Set the latest month
      this.result[this.result.length - 1].time = this.data[0].date;

      let amount = {
        name: "Funding Amount",
        data: [],
        maxPointWidth: 50,
        yAxis: 0,
        color: "rgb(97, 100, 255)",
        enableMouseTracking: false,
        borderWidth: 0,
        dataGrouping: {
          units: groupingUnits,
        },
      };
      let rounds = {
        name: "Number of Funding Rounds",
        data: [],
        type: "spline",
        // lineWidth: 1,
        color: "#f2ba2a",
        yAxis: 1,
        marker: {
          enabled: false,
        },
        dataGrouping: {
          units: groupingUnits,
        },
      };

      amount.data = this.result.map((v) => {
        return [v.time, v.amount * 1000000];
      });
      rounds.data = this.result.map((v) => {
        return {
          x: v.time,
          y: v.countRound,
          custom: {
            countRound: v.countRound,
            amount: v.amount * 1000000,
            time: v.time,
          },
        };
      });
      this.seriesOptions = [amount, rounds];
      if (this.time == "monthly") {
        this.labelFormat = "{value:%b, %y}";
        this.tickInterval = 1 * 30 * 24 * 3600 * 1000;
      }
      if (this.time == "quarterly") {
        this.labelFormat = "{value:Q%q, %y}";
        this.tickInterval = 3 * 30 * 24 * 3600 * 1000;
      }
    },
  },
};
</script>
