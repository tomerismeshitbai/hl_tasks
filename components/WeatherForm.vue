<template>
  <div class="max-w-md mx-auto p-6 bg-white shadow-lg rounded-lg mt-10">
    <form @submit.prevent="fetchWeather">
      <div class="mb-6">
        <label class="block text-gray-800 text-xl font-semibold mb-2" for="city">
          Поиск погоды:
        </label>
        <input
          v-model="city"
          class="shadow appearance-none border rounded w-full py-3 px-4 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          type="text"
          id="city"
          placeholder="Введите название города или почтовый индекс"
          required
        />
      </div>
      <button
        class="bg-gradient-to-r from-blue-500 to-purple-600 hover:from-purple-600 hover:to-blue-500 text-white font-bold py-3 px-6 rounded focus:outline-none focus:shadow-outline transition duration-200 ease-in-out"
        type="submit"
      >
        Показать погоду
      </button>
    </form>

    <div v-if="loading" class="flex justify-center items-center mt-6">
      <div class="loader"></div>
    </div>

    <div v-if="weather" class="mt-8 bg-gray-100 p-6 rounded-lg">
      <div class="flex items-center mb-4">
        <img :src="iconUrl" alt="Weather Icon" class="w-16 h-16" />
        <div class="ml-4">
          <h2 class="text-2xl font-bold text-gray-800">
            {{ weather.name }}, {{ weather.sys.country }}
          </h2>
          <p class="text-5xl font-bold text-gray-800">
            {{ weather.main.temp }}°C
          </p>
        </div>
      </div>

      <div class="grid grid-cols-2 gap-4">
        <div>
          <p class="text-gray-600">Влажность: {{ weather.main.humidity }}%</p>
          <p class="text-gray-600">Давление: {{ weather.main.pressure }} hPa</p>
          <p class="text-gray-600">Ветер: {{ weather.wind.speed }} м/с</p>
        </div>
        <div>
          <p class="text-gray-600">Восход: {{ formatTime(weather.sys.sunrise) }}</p>
          <p class="text-gray-600">Закат: {{ formatTime(weather.sys.sunset) }}</p>
        </div>
      </div>

      <h3 class="text-xl font-bold text-gray-800 mt-8">Прогноз на 3 дня</h3>
      <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4 mt-4">
        <div
          v-for="(day, index) in forecast"
          :key="index"
          class="text-center bg-white p-4 rounded-lg shadow"
        >
          <p class="text-gray-800 font-semibold text-sm">
            {{ formatDate(day.dt) }}
          </p>
          <img
            :src="getForecastIconUrl(day.weather[0].icon)"
            alt="Forecast Icon"
            class="w-12 h-12 mx-auto"
          />
          <p class="text-gray-600 text-xs truncate">
            {{ day.weather[0].description }}
          </p>
          <p class="text-gray-800 font-bold">{{ day.main.temp }}°C</p>
        </div>
      </div>
    </div>

    <div v-if="error" class="mt-6 text-red-500">
      <p>{{ error }}</p>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";
import "moment/locale/ru"; 

export default {
  data() {
    return {
      city: "",
      weather: null,
      forecast: [],
      loading: false,
      error: null,
    };
  },
  computed: {
    iconUrl() {
      if (this.weather) {
        const iconCode = this.weather.weather[0].icon;
        return `http://openweathermap.org/img/wn/${iconCode}@4x.png`;
      }
      return "";
    },
  },
  methods: {
    async fetchWeather() {
      this.loading = true;
      this.error = null;
      const apiKey = "ac5b375cb504c69378505a85186bcd8f";
      try {
        const response = await axios.get(
          `https://api.openweathermap.org/data/2.5/weather?q=${this.city}&units=metric&lang=ru&appid=${apiKey}`
        );
        this.weather = response.data;
        await this.fetchForecast();
      } catch (error) {
        console.error(error);
        this.error =
          "Не удалось получить данные о погоде. Проверьте название города или почтовый индекс и попробуйте еще раз.";
      } finally {
        this.loading = false;
      }
    },
    async fetchForecast() {
      const apiKey = "ac5b375cb504c69378505a85186bcd8f";
      try {
        const response = await axios.get(
          `https://api.openweathermap.org/data/2.5/forecast?q=${this.city}&units=metric&lang=ru&appid=${apiKey}`
        );
        this.forecast = response.data.list
          .filter((reading) => reading.dt_txt.includes("12:00:00"))
          .slice(0, 3);
      } catch (error) {
        console.error(error);
        this.error = "Не удалось получить данные о прогнозе погоды.";
      }
    },
    getForecastIconUrl(iconCode) {
      return `http://openweathermap.org/img/wn/${iconCode}@2x.png`;
    },
    formatTime(unixTimestamp) {
      return moment.unix(unixTimestamp).format("HH:mm");
    },
    formatDate(unixTimestamp) {
      return moment.unix(unixTimestamp).format("D MMMM");
    },
  },
  mounted() {
    moment.locale("ru"); 
  },
  watch: {
    city(newCity) {
      if (!newCity) {
        this.weather = null;
        this.forecast = [];
      }
    },
  },
};
</script>

<style>
body {
  font-family: "Inter", sans-serif;
  background: linear-gradient(to right, #e0f7fa, #ffccbc);
}

.loader {
  border: 4px solid #f3f3f3;
  border-radius: 50%;
  border-top: 4px solid #3498db;
  width: 40px;
  height: 40px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}


@media (max-width: 768px) {
  .loader {
    width: 30px;
    height: 30px;
  }

  .text-sm {
    font-size: 0.875rem; 
  }

  .text-xs {
    font-size: 0.75rem; 
  }

  .grid-cols-3 {
    grid-template-columns: repeat(1, minmax(0, 1fr));
  }
}

@media (min-width: 769px) {
  .grid-cols-3 {
    grid-template-columns: repeat(3, minmax(0, 1fr));
  }
}
</style>
