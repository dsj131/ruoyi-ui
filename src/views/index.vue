<template>
  <div>
    <el-row style="margin-top: 10px;" :gutter="20">
      <el-col :offset="0" :span="8">
        <div class="dashboard-container">
          <div class="dashboard-text">欢迎回来: {{ name }}</div>
        </div>
      </el-col>
      <el-col :offset="1" :span="4">
        <el-button type="success" @click="handleWeather">当前城市天气</el-button>
      </el-col>
    </el-row>
    <el-row :gutter="20" v-if="city">
      <el-col :offset="2" :span="20" style="margin-top: 1%;">
        <el-card>
          <el-descriptions title="当前实时天气" bordered>
            <el-descriptions-item label="当前城市">{{ city }}</el-descriptions-item>
            <el-descriptions-item label="温度">{{ weather.realtime.daytemp_float }}℃</el-descriptions-item>
            <el-descriptions-item label="天气情况">{{ weather.realtime.dayweather }}</el-descriptions-item>
            <el-descriptions-item label="数据时间">{{ reporttime }}</el-descriptions-item>
            <el-descriptions-item label="风向">{{ weather.realtime.daywind }}</el-descriptions-item>
            <el-descriptions-item label="风力">{{ weather.realtime.daypower }}</el-descriptions-item>
          </el-descriptions>
        </el-card>
      </el-col>
    </el-row>
    <el-row v-for="item in weather.future" :key="item.date" style="margin-top: 30px;" :gutter="20">
      <el-col :offset="2" :span="20">
        <el-card>
          <el-descriptions :title="item.date" :column="4" bordered>
            <el-descriptions-item label="温度">{{ item.daytemp_float }}</el-descriptions-item>
            <el-descriptions-item label="天气情况">{{ item.dayweather }}</el-descriptions-item>
            <el-descriptions-item label="风向">{{ item.daywind }}</el-descriptions-item>
            <el-descriptions-item label="风力">{{ item.daypower }}</el-descriptions-item>
          </el-descriptions>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import { getWeather } from "@/api/weather/weather";

export default {
  computed: {
    ...mapGetters([
      'name'
    ])
  },
  name: "weather",
  data() {
    return {
      city: "",
      weather: {
        realtime: {},
        future: []
      },
      reporttime: ""
    }
  },
  methods: {
    handleWeather() {
      getWeather().then(res => {
        const weatherInfo = JSON.parse(res.msg);
        if (weatherInfo.status === '1' && weatherInfo.count > 0 && weatherInfo.forecasts.length > 0) {
          const forecasts = weatherInfo.forecasts[0];
          this.reporttime = forecasts.reporttime;
          this.city = forecasts.city;
          if (forecasts.casts.length > 0) {
            this.weather.realtime = {
              daytemp_float: forecasts.casts[0].daytemp_float,
              daywind: forecasts.casts[0].daywind,
              daypower: forecasts.casts[0].daypower,
              dayhumidity: forecasts.casts[0].dayhumidity,
              dayweather: forecasts.casts[0].dayweather,
            };
            this.weather.future = forecasts.casts.slice(1);
          } else {
            console.error('Failed to get weather data');
          }
        } else {
          console.error('Failed to get weather data');
        }
      }).catch(err => {
        console.error(err);
      });
    }
  },
  created() {
    this.handleWeather();
  }
}
</script>

<style scoped>
.dashboard-text {
  color: gray;
  font-size: 30px;
  line-height: 46px;
}

.el-card {
  box-shadow: 0 2px 4px rgba(0, 0, 0, .12), 0 0 6px rgba(0, 0, 0, .04);
}
</style>
