<template>
  <div id="main-app">
    <div class="container">
      <div v-if="!intro">
        <div class="card select-list">
          <div class="province-search">
            <input
              type="text"
              v-model="provinceSearch"
              placeholder="サーチ . . ."
            />
          </div>
          <div class="grid-container">
            <button class="search-button" @click="toggleInputSearch()">
              Search
            </button>
            <button
              class="grid-item item-not-selected"
              v-for="province in filterItems(provinces)"
              :key="province.prefCode"
              v-on:click="provinceSelected($event)"
              v-bind:id="province.prefCode"
              v-bind:value="province.prefName"
            >
              {{ province.prefName }}
            </button>
          </div>
        </div>
        <button class="card show-select-list" @click="selectItem()">
          + / -
        </button>
        <div id="chartView" class="card" v-if="chartShow">
          <highcharts
            class="hc"
            :options="chartOptions"
            ref="chart"
          ></highcharts>
        </div>
      </div>
      <div @click="endIntro()" v-if="intro" class="start-button drop">+</div>
    </div>
  </div>
</template>
<script>
import axios from "axios";
import $ from "jquery";
export default {
  name: "MainApp",
  data() {
    return {
      intro: true,
      provinces: null,
      provinceSearch: "",
      chartShow: false,
      arrayData: [],
      chartOptions: {
        series: [],
        title: {
          text: "人口チャート",
        },
        chart: {
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
  filters: {},
  methods: {
    getAllProvince: function () {
      axios
        .get(process.env.VUE_APP_API_URL + "/prefectures", {
          headers: {
            "X-API-KEY": process.env.VUE_APP_API_KEY,
          },
        })
        .then((response) => {
          this.provinces = response.data.result;
        });
    },
    filterItems: function (provinces) {
      var app = this;
      if (this.provinceSearch == "") return provinces;
      else {
        return provinces.filter(function (province) {
          return province.prefName
            .toLowerCase()
            .includes(app.provinceSearch.toLowerCase());
        });
      }
    },
    provinceSelected: function (event) {
      this.bufferObj = this.chartOptions.series.find(
        (o) => o.id === event.target.id.toString()
      );
      if (this.bufferObj == undefined) {
        $(event.target).attr("disabled", true);
        $(event.target).addClass("spinning");
        axios
          .get(
            process.env.VUE_APP_API_URL +
              "/population/composition/perYear?cityCode=-&prefCode=" +
              event.target.id,
            {
              headers: {
                "X-API-KEY": process.env.VUE_APP_API_KEY,
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
            $(event.target).attr("disabled", false);
            $(event.target).removeClass("spinning");
            $(event.target).addClass("item-checked");
            $(".select-list").hide("slow");
            $(".show-select-list").show("slow");
            if ($(".province-search").css("display") != "none") {
              this.toggleInputSearch();
            }
            if (this.chartOptions.series.length > 0) this.chartShow = true;
          });
      } else {
        $(event.target).removeClass("item-checked");
        $(".select-list").hide("slow");
        this.chartOptions.series.splice(
          this.chartOptions.series.indexOf(this.bufferObj),
          1
        );
        if (this.chartOptions.series.length == 0) {
          this.chartShow = false;
          this.intro = true;
        } else {
          $(".show-select-list").show("slow");
        }
      }
    },
    selectItem: function () {
      $(".select-list").show("slow");
      $(".show-select-list").hide("slow");
    },
    endIntro() {
      this.intro = false;
    },
    toggleInputSearch() {
      this.provinceSearch = "";
      if ($(".province-search").css("display") == "none") {
        $(".province-search").css("display", "block");
        $(".province-search").removeClass("hide-input");
        $(".province-search").addClass("show-input");
      } else {
        $(".province-search").removeClass("show-input");
        $(".province-search").addClass("hide-input");
        setTimeout(function () {
          $(".province-search").css("display", "none");
        }, 1000);
      }
    },
  },
  created() {
    this.chartOptions.chart.height = $(window).height() * 0.7;
    this.getAllProvince();
  },
};
</script>
<style scoped src="./../assets/css/style.css"></style>