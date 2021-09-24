<template>
  <div class="chart">
    <li v-for="province in provinces" :key="province.prefCode">
      <input
        type="checkbox"
        v-bind:id="province.prefCode"
        v-bind:value="province.prefName"
        v-on:change="provinceSelected($event)"
      />
      <label v-bind:for="province.prefCode">{{ province.prefName }}</label
      ><br />
    </li>
    <div>
      <highcharts
        v-if="chartShow"
        class="hc"
        :options="chartOptions"
        ref="chart"
      ></highcharts>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      provinces: null,
      chartShow: false,
      arrayData: [],
      chartOptions: {
        series: [],
        title: {
          text: "人口チャート",
        },
        chart: {
          height: 500,
          type: "line",
        },
        yAxis: {
          title: {
            text: "人口数",
          },
        },
        xAxis: {
          title: {
            text: "年度",
          },
        },
      },
    };
  },
  methods: {
    loadingAllProvince: function () {
      axios
        .get("https://opendata.resas-portal.go.jp/api/v1/prefectures", {
          headers: {
            "X-API-KEY": "mWoSgpkUSvaG4xrQy3VGp9o3sh8QjfIBHDLDtyGe",
          },
        })
        .then((response) => {
          this.provinces = response.data.result;
        });
    },
    provinceSelected: function (event) {
      axios
        .get(
          "https://opendata.resas-portal.go.jp/api/v1/population/composition/perYear?cityCode=-&prefCode=" +
            event.target.id,
          {
            headers: {
              "X-API-KEY": "mWoSgpkUSvaG4xrQy3VGp9o3sh8QjfIBHDLDtyGe",
            },
          }
        )
        .then((response) => {
          this.arrayData = [];
          for (var i = 0; i < response.data.result.data[0].data.length; i++) {
            this.arrayData.push([
              response.data.result.data[0].data[i].year,
              response.data.result.data[0].data[i].value,
            ]);
          }
          this.chartOptions.series.push({
            name: event.target.value,
            data: this.arrayData,
            id: event.target.id,
          });
          if (this.chartOptions.series.length > 0) this.chartShow = true;
        });
    },
  },
  created() {
    this.loadingAllProvince();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
</style>
