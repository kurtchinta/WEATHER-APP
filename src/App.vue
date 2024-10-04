<template>
  <v-app :style="appStyle">
    <v-main>
      <v-container class="fill-height">
        <v-row justify="center" align="center">
          <v-col cols="12" sm="10" md="8" lg="6">
            <v-card class="mb-6 mt-12 weather-title-card" elevation="4" :style="cardStyle">
              <v-card-text class="text-center">
                <h1 class="weather-title">WeatherWise</h1>
                <p class="weather-subtitle">Know the Sky</p>
              </v-card-text>
            </v-card>

            <v-card :style="cardStyle" class="weather-card" elevation="10">
              <v-card-text>
                <v-text-field
                  v-model="city"
                  label="Enter city"
                  @keyup.enter="fetchWeather"
                  :color="staticColor"
                  outlined
                  rounded
                  dense
                  clearable
                  hide-details
                  class="mb-4"
                >
                  <template v-slot:prepend-inner>
                    <v-icon :color="staticColor">mdi-map-marker</v-icon>
                  </template>
                  <template v-slot:append>
                    <v-btn icon @click="fetchWeather" :color="staticColor">
                      <v-icon>mdi-magnify</v-icon>
                    </v-btn>
                  </template>
                </v-text-field>
                <v-alert v-if="error" type="error" dense outlined>{{ error }}</v-alert>
                <v-row v-if="!weather && !error" justify="center" class="mt-8">
                  <v-col cols="12" class="text-center">
                    <v-icon :color="iconColor" size="100">mdi-map-marker-question</v-icon>
                    <p class="mt-4 text-h6">Enter a city to discover its weather!</p>
                  </v-col>
                </v-row>
              </v-card-text>
            </v-card>

            <v-card v-if="weather" class="mt-4 weather-details-card" :style="weatherCardStyle">
              <v-card-title class="text-h5 font-weight-bold">
                <v-icon :color="iconColor" size="36" class="mr-2">{{ weatherIcon }}</v-icon>
                {{ weather.name }}
                <v-chip class="ml-2" :color="chipColor" text-color="white">
                  {{ weather.sys.country }}
                </v-chip>
                <v-spacer></v-spacer>
                <span class="text-h3">{{ formatTemperature(weather.main.temp) }}</span>
              </v-card-title>
              <v-card-subtitle class="text-h6 mt-2">
                {{ weather.weather[0].description }}
              </v-card-subtitle>
              <v-card-text>
                <v-row>
                  <v-col cols="6" sm="3" v-for="(detail, index) in weatherDetails" :key="index">
                    <v-tooltip bottom>
                      <template v-slot:activator="{ on, attrs }">
                        <div class="text-subtitle-1" v-bind="attrs" v-on="on">
                          <v-icon small :color="iconColor">{{ detail.icon }}</v-icon>
                          {{ detail.value }}
                        </div>
                      </template>
                      <span>{{ detail.label }}</span>
                    </v-tooltip>
                  </v-col>
                </v-row>
                <v-divider class="my-4"></v-divider>
                <v-row>
                  <v-col cols="6" sm="6">
                    <div class="text-subtitle-1">
                      <v-icon small :color="iconColor">mdi-weather-sunset-up</v-icon>
                      Sunrise: {{ formatTime(weather.sys.sunrise) }}
                    </div>
                  </v-col>
                  <v-col cols="6" sm="6">
                    <div class="text-subtitle-1">
                      <v-icon small :color="iconColor">mdi-weather-sunset-down</v-icon>
                      Sunset: {{ formatTime(weather.sys.sunset) }}
                    </div>
                  </v-col>
                </v-row>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>

    <v-footer :style="footerStyle" class="py-2">
      <v-container>
        <v-row justify="center" align="center" no-gutters>
          <v-col cols="12" class="text-center">
            <span class="caption">
              &copy; {{ new Date().getFullYear() }} WeatherWise by 
              <a href="https://www.instagram.com/kurtchintaaa/" target="_blank" rel="noopener noreferrer" class="footer-link">
                Kurt Reserva
              </a>
            </span>
          </v-col>
        </v-row>
      </v-container>
    </v-footer>
  </v-app>
</template>

<script>
import { ref, computed } from 'vue';

export default {
  setup() {
    const city = ref('');
    const weather = ref(null);
    const error = ref('');
    const temperatureUnit = ref('celsius');
    const staticColor = ref('#1976D2'); // A static color for icons and input

    const fetchWeather = async () => {
      if (city.value.trim() === '') {
        error.value = 'Please enter a city name.';
        return;
      }
      try {
        const apiKey = '8c17362cafab5d344b7b3727b87a1fe0'; // Replace with your OpenWeatherMap API key
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${city.value}&units=metric&appid=${apiKey}`
        );
        const data = await response.json();
        if (data.cod === 200) {
          weather.value = data;
          error.value = '';
        } else {
          error.value = 'City not found.';
          weather.value = null;
        }
      } catch (err) {
        error.value = 'Unable to fetch weather data.';
        weather.value = null;
      }
    };

    const getWeatherTheme = computed(() => {
      if (!weather.value) return {};
      const temp = weather.value.main.temp;
      if (temp < 0) return { bg: '#E0F7FA', color: '#01579B', icon: 'mdi-snowflake' }; // Very Cold
      if (temp < 10) return { bg: '#E1F5FE', color: '#0288D1', icon: 'mdi-weather-snowy' }; // Cold
      if (temp < 20) return { bg: '#E8F5E9', color: '#388E3C', icon: 'mdi-weather-partly-cloudy' }; // Cool
      if (temp < 30) return { bg: '#FFFDE7', color: '#FBC02D', icon: 'mdi-weather-sunny' }; // Warm
      return { bg: '#FFF3E0', color: '#E64A19', icon: 'mdi-weather-sunny-alert' }; // Hot
    });

    const appStyle = computed(() => ({
      background: `linear-gradient(135deg, ${getWeatherTheme.value.bg || '#F0F8FF'}, ${getWeatherTheme.value.color || '#333'})`,
      transition: 'all 0.5s ease-in-out',
    }));

    const cardStyle = computed(() => ({
      background: 'rgba(255, 255, 255, 0.1)',
      backdropFilter: 'blur(10px)',
      borderRadius: '20px',
    }));

    const weatherCardStyle = computed(() => ({
      background: 'rgba(255, 255, 255, 0.2)',
      backdropFilter: 'blur(5px)',
      borderRadius: '15px',
    }));

    const footerStyle = computed(() => ({
      background: 'rgba(255, 255, 255, 0.1)',
      backdropFilter: 'blur(10px)',
    }));

    const iconColor = computed(() => getWeatherTheme.value.color || 'primary');
    const weatherIcon = computed(() => getWeatherTheme.value.icon || 'mdi-cloud-question');
    const chipColor = computed(() => getWeatherTheme.value.color || 'grey');

    const formatTemperature = (temp) => {
      const roundedTemp = Math.round(temp);
      return temperatureUnit.value === 'celsius' 
        ? `${roundedTemp}°C` 
        : `${Math.round(roundedTemp * 9/5 + 32)}°F`;
    };

    const formatWindSpeed = (speed) => {
      return `${Math.round(speed * 3.6)} km/h`;
    };

    const formatVisibility = (visibility) => {
      return `${(visibility / 1000).toFixed(1)} km`;
    };

    const formatTime = (timestamp) => {
      const date = new Date(timestamp * 1000);
      return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    };

    const weatherDetails = computed(() => {
      if (!weather.value) return [];
      return [
        { icon: 'mdi-water-percent', label: 'Humidity', value: `${weather.value.main.humidity}%` },
        { icon: 'mdi-weather-windy', label: 'Wind Speed', value: formatWindSpeed(weather.value.wind.speed) },
        { icon: 'mdi-thermometer-low', label: 'Min Temperature', value: formatTemperature(weather.value.main.temp_min) },
        { icon: 'mdi-thermometer-high', label: 'Max Temperature', value: formatTemperature(weather.value.main.temp_max) },
        { icon: 'mdi-eye-outline', label: 'Visibility', value: formatVisibility(weather.value.visibility) },
        { icon: 'mdi-gauge', label: 'Pressure', value: `${weather.value.main.pressure} hPa` },
      ];
    });

    return {
      city,
      weather,
      error,
      fetchWeather,
      appStyle,
      cardStyle,
      weatherCardStyle,
      footerStyle,
      staticColor,
      iconColor,
      weatherIcon,
      chipColor,
      formatTemperature,
      formatTime,
      weatherDetails,
    };
  },
};
</script>

<style scoped>
.weather-card, .weather-details-card, .weather-title-card {
  overflow: hidden;
  position: relative;
}

.weather-card::before, .weather-details-card::before, .weather-title-card::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255,255,255,0.2) 0%, rgba(255,255,255,0) 70%);
  animation: shimmer 15s linear infinite;
}

@keyframes shimmer {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.v-application {
  transition: background-color 0.3s ease;
}

.weather-title {
  font-size: 2.5rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 2px;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
  margin-bottom: 0.5rem;
}

.weather-subtitle {
  font-size: 1.2rem;
  font-weight: 300;
  letter-spacing: 1px;
}

.footer-link {
  color: inherit;
  text-decoration: none;
  font-weight: bold;
}

.footer-link:hover {
  text-decoration: underline;
}

@media (max-width: 600px) {
  .weather-title {
    font-size: 2rem;
  }

  .weather-subtitle {
    font-size: 1rem;
  }
}
</style>