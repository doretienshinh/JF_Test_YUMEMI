<template>
  <div id="app">
    <div class="container">
      <div class="card select-list" v-if="!intro">
        <div class="grid-container">
          <button
            class="grid-item"
            v-for="province in provinces"
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
        <highcharts class="hc" :options="chartOptions" ref="chart"></highcharts>
      </div>
      <div @click="show()" v-if="intro" class="start-button drop loading">
        +
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";
import $ from "jquery";
export default {
  name: "App",
  data() {
    return {
      apiKey: "mWoSgpkUSvaG4xrQy3VGp9o3sh8QjfIBHDLDtyGe",
      intro: false,
      provinces: null,
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
    loadingAllProvince: function () {
      axios
        .get("https://opendata.resas-portal.go.jp/api/v1/prefectures", {
          headers: {
            "X-API-KEY": this.apiKey,
          },
        })
        .then((response) => {
          this.provinces = response.data.result;
        });
    },
    provinceSelected: function (event) {
      $(event.target).toggleClass("item-checked");
      this.bufferObj = this.chartOptions.series.find(
        (o) => o.id === event.target.id.toString()
      );
      if (this.bufferObj == undefined) {
        $(event.target).attr("disabled", true);
        $(event.target).addClass("spinning");
        axios
          .get(
            "https://opendata.resas-portal.go.jp/api/v1/population/composition/perYear?cityCode=-&prefCode=" +
              event.target.id,
            {
              headers: {
                "X-API-KEY": this.apiKey,
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
            $(".select-list").hide("slow");
            $(".show-select-list").show("slow");
            if (this.chartOptions.series.length > 0) this.chartShow = true;
          });
      } else {
        $(".select-list").hide("slow");
        $(".show-select-list").show("slow");
        this.chartOptions.series.splice(
          this.chartOptions.series.indexOf(this.bufferObj),
          1
        );
        if (this.chartOptions.series.length == 0) this.chartShow = false;
      }
    },
    selectItem: function () {
      $(".select-list").show("slow");
      $(".show-select-list").hide("slow");
    },
    show() {
      this.intro = true;
      //document.querySelector("body").style.overflowY = 'scroll';
    },
  },
  created() {
    console.log($(window).height());
    this.chartOptions.chart.height = $(window).height() * 0.7;
    this.loadingAllProvince();
    document.querySelector("body").style.overflowX = "hidden";
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.container {
  padding: 20px 30px;
}
.card {
  border-radius: 10px;
  cursor: pointer;
  box-shadow: 0 0 20px 8px #d0d0d0;
  padding: 20px 30px;
  margin: 20px 0 0 0;
}
.grid-container {
  display: inline-grid;
  width: 100%;
  grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr;
}
.grid-item {
  text-align: center;
  margin: 0.5rem 0.5rem;
  background-color: rgba(255, 255, 255, 0.9);
  border: 2px solid rgba(139, 139, 139, 0.3);
  color: #adadad;
  border-radius: 25px;
  position: relative;
  white-space: nowrap;
  user-select: none;
  transition: all 0.2s;
  padding: 8px 12px;
  cursor: pointer;
}
.item-checked {
  border: 2px solid #1bdbf8;
  background-color: #12bbd4;
  color: #fff;
  transition: all 0.2s;
}
.start-button {
  margin: auto;
  margin-top: 10%;
  height: 20rem;
  line-height: 20rem;
  width: 20rem;
  background-color: rgb(49, 133, 182);
  border-radius: 50%;
  cursor: pointer;
  text-align: center;
  color: white;
  font-size: 15rem;
}
.drop {
  animation: drop 1s normal;
}
@keyframes drop {
  0% {
    transform-origin: center;
    opacity: 1;
  }
  /* 50% {
    opacity: 0.5;
  } */
  to {
    opacity: 0;
  }
}
.spinning {
  padding-right: 15px;
}
.spinning:before {
  content: "";
  width: 0px;
  height: 0px;
  border-radius: 50%;
  right: 5px;
  top: 50%;
  border: 2px solid white;
  border-right: 3px solid #2a0a42;
  position: absolute;
  animation: rotate360 0.5s infinite linear, exist 0.1s forwards ease;
}
@keyframes rotate360 {
  100% {
    transform: rotate(360deg);
  }
}

@keyframes exist {
  100% {
    width: 15px;
    height: 15px;
    margin: -8px 5px 0 0;
  }
}
.show-select-list {
  background-color: #3dd871;
  border: 1px solid #3dd871;
  border-radius: 15px;
  color: white;
  font-size: 20px;
  width: 100%;
  display: none;
}
@media screen and (max-width: 1024px) {
  .grid-container {
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr;
  }
}
@media screen and (max-width: 768px) {
  .grid-container {
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr;
  }
}
@media screen and (max-width: 644px) {
  .grid-container {
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr;
  }
}
@media screen and (max-width: 567px) {
  .grid-container {
    grid-template-columns: 1fr 1fr 1fr 1fr;
  }
}
@media screen and (max-width: 447px) {
  .grid-container {
    grid-template-columns: 1fr 1fr 1fr;
  }
}
@media screen and (max-width: 375px) {
  .grid-container {
    grid-template-columns: 1fr 1fr;
  }
}
@media screen and (max-width: 285px) {
  .grid-container {
    grid-template-columns: 1fr;
  }
}
</style>

