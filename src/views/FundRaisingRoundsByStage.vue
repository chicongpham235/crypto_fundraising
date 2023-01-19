<template>
  <BlockChart :loading="loading">
    <template #title>
      <div class="w-full">
        <h2 class="text-white font-bold text-[20px]">
          Number of Fundraising Rounds by Stage
          <TimeFrame
            class="absolute top-[42px] right-0 z-[1]"
            :time_frame="time_frame"
            @set_time_frame="(v) => (time = v)"
          />
        </h2>
      </div>
    </template>
    <HighChart
      id="fundraising-rounds-by-stage"
      :options="options"
      height="600px"
      ref="highchart"
    />
  </BlockChart>
</template>

<script>
import { roundValue } from "@/ultis";
import moment from "moment";
import BlockChart from "@/components/BlockChart.vue";
import HighChart from "@/components/HighChart.vue";
import TimeFrame from "@/components/TimeFrame.vue";
let highcharts = require("highcharts");
require("highcharts/highcharts-more")(highcharts);
require("highcharts/modules/accessibility")(highcharts);
require("highcharts/modules/exporting")(highcharts);
var pieColors = [
  "#7c7cf4",
  "#7F00FF",
  "#DA70D6",
  "#8ab4f8",
  "#6495ED",
  "#01bbbb",
  "#3dca3d",
  "#aecbfa",
  "rgb(234 179 8)",
  "orange ",
  "rgb(209 213 219)",
];
export default {
  components: {
    BlockChart,
    HighChart,
    TimeFrame,
  },
  name: "fundraising-rounds-by-stage",
  props: {
    api_res_data: { type: Array },
  },
  data: () => ({
    data: [],
    time_frame: ["1y", "2y", "YTD", "All"],
    time: "1y",
    seriesOptions: [],
    loading: true,
    legendX: 85,
    legendY: 30,
    verticalAlign: "middle",
    total: 0,
    titleText: "",
    highchartTemp: null,
  }),
  watch: {
    api_res_data: {
      handler() {
        this.init();
      },
    },
    total: {
      handler(val) {
        this.highchartTemp?.highChart?.setTitle({
          text: `
          <div class="text-center col-span-12 font-bold text-6xl">${val}</div>
          <div class="text-center font-medium text-3xl">Total</div>
          `,
        });
      },
    },
    time: {
      handler() {
        this.fetchData();
      },
    },
  },
  computed: {
    options(vm) {
      vm.highchartTemp = vm.$refs.highchart;
      return {
        chart: {
          backgroundColor: "#2e2e33",
          events: {
            load: function () {
              const legend = this.legend,
                boxWrapper = legend.chart.renderer.boxWrapper;
              const points = this.series[0]?.points;
              points?.forEach((point) => {
                ["label", "symbol"].forEach((prop) => {
                  let isPoint = point instanceof highcharts.Point,
                    activeClass =
                      "highcharts-legend-" +
                      (isPoint ? "point" : "series") +
                      "-active";
                  point.legendItem[prop]
                    .on("mouseover", function () {
                      this.style.fontWeight = "bold";
                      if (point.visible) {
                        points.forEach(function (inactiveItem) {
                          if (point !== inactiveItem) {
                            inactiveItem.setState("inactive", !isPoint);
                          }
                        });
                        boxWrapper.addClass(activeClass);
                        let text = `
                    <div class="text-center col-span-12 font-bold text-6xl">${point.y}</div>
                    <div class="text-center font-medium text-3xl" style="color: ${point.color}">${point.name}</div>
                    `;
                        vm.highchartTemp?.highChart?.setTitle({
                          text: text,
                        });
                      }
                      if (!point.visible) {
                        let text = `
                    <div class="text-center col-span-12 font-bold text-6xl">0</div>
                    <div class="text-center font-medium text-3xl">${point.name}</div>
                    `;
                        vm.highchartTemp?.highChart?.setTitle({
                          text: text,
                        });
                      }
                      point.setState("hover");
                    })
                    .on("mouseout", function () {
                      this.style.fontWeight = "normal";
                      points.forEach(function (inactiveItem) {
                        if (point !== inactiveItem) {
                          inactiveItem.setState("", !isPoint);
                        }
                      });
                      let text = `
                    <div class="text-center col-span-12 font-bold text-6xl">${point.total}</div>
                    <div class="text-center font-medium text-3xl">Total</div>
                    `;
                      vm.highchartTemp?.highChart?.setTitle({
                        text: text,
                      });
                    });
                });
              });
            },
          },
        },
        title: {
          useHTML: true,
          style: {
            color: "#ffffff",
          },
          text: this.titleText,
          align: "center",
          verticalAlign: "middle",
          x: this.legendX,
          y: this.legendY,
        },
        credits: {
          enabled: false,
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
          shared: true,
          formatter: function () {
            let result = `<div class="border border-white rounded p-2">
                <div style="color: ${this.color}">${
              this.key
            }: <span class="text-white font-semibold mr-2">${
              this.y
            }</span><span class="text-gray-500 mr-2">
              (${roundValue(this.percentage, 2)}%)
            </span></div>
                </div>`;
            return result;
          },
        },
        legend: {
          enabled: true,
          useHTML: true,
          borderWidth: 0,
          verticalAlign: this.verticalAlign,
          layout: "vertical",
          align: "left",
          width: 150,
          itemStyle: {
            fontSize: "12px",
            color: "rgb(148 163 184)",
            fontWeight: "normal",
          },
          labelFormatter: function () {
            vm.total = this.total;
            return (
              '<div class=" "> ' +
              '<div class="row" style="display: flex; width: 130px; height:25px; justify-content: space-between;"> ' +
              '<div class="col">' +
              this.name +
              "</div>" +
              '<div class="col">' +
              roundValue(this.percentage, 2) +
              "%" +
              "</div>" +
              "</div>" +
              "</div>"
            );
          },
        },
        plotOptions: {
          pie: {
            allowPointSelect: true,
            cursor: "pointer",
            innerSize: "60%",
            colors: pieColors,
            dataLabels: {
              enabled: false,
            },
            showInLegend: true,
            point: {
              events: {
                mouseOver: function () {
                  let text = `
                    <div class="text-center col-span-12 font-bold text-6xl">${this.y}</div>
                    <div class="text-center font-medium text-3xl" style="color: ${this.color}">${this.name}</div>
                    `;
                  vm.highchartTemp?.highChart?.setTitle({
                    text: text,
                  });
                },
                mouseOut: function () {
                  let text = `
                    <div class="text-center col-span-12 font-bold text-6xl">${this.total}</div>
                    <div class="text-center font-medium text-3xl"">Total</div>
                    `;
                  vm.highchartTemp?.highChart?.setTitle({
                    text: text,
                  });
                },
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
      this.seriesOptions = [];
      try {
        this.loading = true;
        await this.fetchData();
      } catch (error) {
        console.log(error);
        this.seriesOptions = [];
      } finally {
        this.loading = false;
      }
    },
    async fetchData() {
      const res_data = this.api_res_data;
      let now = moment(new Date().getTime());
      let time = null;
      if (this.time == "1y")
        time = new Date(
          Date.UTC(now.year() - 1, now.month(), now.date(), 0, 0, 0)
        ).getTime();
      if (this.time == "2y")
        time = new Date(
          Date.UTC(now.year() - 2, now.month(), now.date(), 0, 0, 0)
        ).getTime();
      if (this.time == "ytd")
        time = new Date(Date.UTC(now.year(), 0, 1, 0, 0, 0)).getTime();
      if (this.time == "all") time = new Date(time).getTime();
      let data = res_data.filter((v) => !(v.date < time));
      let roundArr = [...data.map((item) => item.round)];
      this.total = roundArr.length;
      let round = [
        {
          name: "Seed",
          count: roundArr.filter(
            (x) =>
              x?.toLowerCase().replace(/\s/g, "").replace("-", "") == "seed"
          ).length,
        },
        {
          name: "Strategic",
          count: roundArr.filter(
            (x) =>
              x?.toLowerCase().replace(/\s/g, "").replace("-", "") ==
              "strategic"
          ).length,
        },
        {
          name: "Pre-Seed",
          count: roundArr.filter(
            (x) =>
              x?.toLowerCase().replace(/\s/g, "").replace("-", "") == "preseed"
          ).length,
        },
        {
          name: "Series A",
          count: roundArr.filter((x) => {
            let str =
              x?.toLowerCase().replace(/\s/g, "").replace("-", "") ?? "";
            if (str != "seriesa&b") return str.includes("seriesa");
          }).length,
        },
        {
          name: "Series B",
          count: roundArr.filter((x) => {
            let str =
              x?.toLowerCase().replace(/\s/g, "").replace("-", "") ?? "";
            return str.includes("seriesb");
          }).length,
        },
        {
          name: "Series C",
          count: roundArr.filter((x) => {
            let str =
              x?.toLowerCase().replace(/\s/g, "").replace("-", "") ?? "";
            return str.includes("seriesc");
          }).length,
        },
        {
          name: "Series D",
          count: roundArr.filter((x) => {
            let str =
              x?.toLowerCase().replace(/\s/g, "").replace("-", "") ?? "";
            return str.includes("seriesd");
          }).length,
        },
        {
          name: "Series E",
          count: roundArr.filter((x) => {
            let str =
              x?.toLowerCase().replace(/\s/g, "").replace("-", "") ?? "";
            return str.includes("seriese");
          }).length,
        },
        {
          name: "Series F",
          count: roundArr.filter((x) => {
            let str =
              x?.toLowerCase().replace(/\s/g, "").replace("-", "") ?? "";
            return str.includes("seriesf");
          }).length,
        },
        {
          name: "Private Sale",
          count: roundArr.filter((x) =>
            x
              ?.toLowerCase()
              .replace(/\s/g, "")
              .replace("-", "")
              .includes("private")
          ).length,
        },
      ];
      let othersRoundCount = () => {
        let sum = 0;
        for (let i = 0; i < round.length; i++) {
          sum += round[i].count;
        }
        return sum;
      };
      let dataSeries = [];
      let i = 0;
      round.forEach((element) => {
        dataSeries.push({
          name: element.name,
          y: element.count,
          color: pieColors[i],
        });
        i++;
      });
      dataSeries = dataSeries.sort((a, b) => b.y - a.y);
      dataSeries.push({
        name: "Others",
        y: roundArr.length - othersRoundCount(),
        color: pieColors[10],
      });
      this.seriesOptions = [
        {
          type: "pie",
          name: "Percent",
          data: dataSeries,
        },
      ];
      this.titleText = `
            <div class="text-center col-span-12 font-bold text-6xl">${this.total}</div>
            <div class="text-center font-medium text-3xl">Total</div>
            `;
    },
  },
};
</script>
