<template>
  <div class="chart">
    <li v-for="province in provinces" :key="province.prefCode">
      <input
        type="checkbox"
        v-bind:id="province.prefCode"
        v-bind:value="province.prefName"
      />
      <label v-bind:for="province.prefCode">{{ province.prefName }}</label
      ><br />
    </li>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      provinces: null,
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
          console.log(this.provinces);
        });
    }
  },
  created() {
    this.loadingAllProvince();
  },
};
</script>
<style scoped>
  li {
    list-style-type: none;
  }
</style>
