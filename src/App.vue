<script setup>
import { ref, computed } from "vue";
import Search from "./components/Search.vue";
import { onMounted } from "vue";

const city = ref("paris");
const searchText = ref("");
const arrayClouds = ref([]);
const arrayFuturTemps = ref([]);
const weather = ref(null);
const isloading = ref(true);
const error = ref(null);

const apiKey = import.meta.env.VITE_WEATHER_API_KEY;

// const heures = now.getHours().toString().padStart(2, '0');
// const minutes = now.getMinutes().toString().padStart(2, '0');
// const secondes = now.getSeconds().toString().padStart(2, '0');

const dateFormatNow = (dateDuJour) => {
  const annee = dateDuJour.getFullYear();
  const mois = (dateDuJour.getMonth() + 1).toString().padStart(2, "0");
  const jour = dateDuJour.getDate().toString().padStart(2, "0");

  return `${annee}-${mois}-${jour}`;
};

const dateFormatee = dateFormatNow(new Date());

const dailyForecasts = computed(() => {
  const forecasts = {};

  if (arrayFuturTemps.value && arrayFuturTemps.value.length) {
    arrayFuturTemps.value.forEach((item) => {
      // Convertir le timestamp en date et extraire juste la date (sans l'heure)
      const date = new Date(item.dt * 1000);
      const dateKey = date.toISOString().split("T")[0]; // Format YYYY-MM-DD

      // Si nous n'avons pas encore d'entrée pour ce jour, la créer
      if (!forecasts[dateKey]) {
        forecasts[dateKey] = {
          date: date,
          dayName: getDayName(date),
          temp: item.main.temp,
          description: item.weather[0].description,
          icon: item.weather[0].icon,
          // Pour calculer la moyenne plus tard
          temps: [item.main.temp],
          dt: item.dt,
        };
      } else {
        // Sinon, ajouter cette température à la liste pour calculer la moyenne
        forecasts[dateKey].temps.push(item.main.temp);

        // Si c'est midi, utiliser cette prévision comme représentative du jour
        const hour = date.getHours();
        if (hour >= 12 && hour <= 14) {
          forecasts[dateKey].temp = item.main.temp;
          forecasts[dateKey].description = item.weather[0].description;
          forecasts[dateKey].icon = item.weather[0].icon;
        }
      }
    });
  }

  // Convertir l'objet en tableau pour v-for et trier par date
  return Object.values(forecasts).sort((a, b) => a.dt - b.dt);
});

// Computed séparé pour les prévisions d'aujourd'hui
const todayForecasts = computed(() => {
  const today = [];

  if (arrayFuturTemps.value && arrayFuturTemps.value.length) {
    arrayFuturTemps.value.forEach((ele) => {
      const newDateReviewApi = dateFormatNow(new Date(ele.dt_txt));

      if (newDateReviewApi === dateFormatee) {
        today.push(ele);
      }
    });
  }

  return today;
});

function getDayName(date) {
  const days = [
    "Dimanche",
    "Lundi",
    "Mardi",
    "Mercredi",
    "Jeudi",
    "Vendredi",
    "Samedi",
    "Dimanche",
  ];
  return days[date.getDay()].substring(0, 3);
}

async function getWeather() {
  try {
    isloading.value = true;
    error.value = null;

    if (!city.value.trim()) {
      city.value = "paris";
    }

    // Vérifier si l'API key est disponible
    if (!apiKey) {
      throw new Error(
        "Clé API OpenWeatherMap manquante. Veuillez configurer VITE_WEATHER_API_KEY dans vos variables d'environnement."
      );
    }

    const url = `https://api.openweathermap.org/data/2.5/weather?q=${city.value}&appid=${apiKey}&units=metric&lang=fr`;
    const url2 = `https://api.openweathermap.org/data/2.5/forecast?q=${city.value}&appid=${apiKey}&units=metric&lang=fr`;

    // Faire les deux appels API en parallèle pour de meilleures performances
    const [forecastResponse, weatherResponse] = await Promise.all([
      fetch(url2),
      fetch(url),
    ]);

    onMounted(() => {
      getWeather();

      // Debug - à supprimer après
      console.log("Date formatée aujourd'hui:", dateFormatee);
      console.log("Données arrayFuturTemps:", arrayFuturTemps.value);
    });

    if (!forecastResponse.ok || !weatherResponse.ok) {
      if (forecastResponse.status === 404 || weatherResponse.status === 404) {
        throw new Error(
          `Ville "${city.value}" non trouvée. Veuillez vérifier l'orthographe.`
        );
      } else if (
        forecastResponse.status === 401 ||
        weatherResponse.status === 401
      ) {
        throw new Error(
          "Clé API invalide. Veuillez vérifier votre configuration."
        );
      } else {
        throw new Error(
          `Erreur du serveur (${
            forecastResponse.status || weatherResponse.status
          }). Veuillez réessayer plus tard.`
        );
      }
    }

    const [forecastData, weatherData] = await Promise.all([
      forecastResponse.json(),
      weatherResponse.json(),
    ]);

    arrayFuturTemps.value = forecastData.list;
    weather.value = weatherData;
    arrayClouds.value = weatherData.weather;
    searchText.value = "";
  } catch (e) {
    console.error("Erreur lors de la récupération des données météo:", e);
    error.value = e.message;

    // Si la ville n'est pas trouvée et ce n'est pas Paris, essayer Paris par défaut
    if (
      e.message.includes("non trouvée") &&
      city.value.toLowerCase() !== "paris"
    ) {
      console.log("Tentative avec Paris par défaut...");
      city.value = "paris";
      searchText.value = "";
      await getWeather();
    }
  } finally {
    isloading.value = false;
  }
}

function handleSearch(value) {
  const newCity = value?.trim() || "paris";

  // Éviter les appels API redondants
  if (newCity.toLowerCase() === city.value.toLowerCase()) {
    return;
  }

  city.value = newCity;
  getWeather();
}

onMounted(() => {
  getWeather();
});
</script>

<template>
  <div
    class="min-h-screen bg-gradient-to-br from-blue-400 via-blue-500 to-blue-600"
  >
    <div class="container mx-auto px-4 py-6">
      <Search v-model="searchText" @search="handleSearch" />

      <!-- Loading State -->
      <div v-if="isloading" class="flex justify-center items-center h-64">
        <div class="bg-white/20 backdrop-blur-md rounded-2xl p-8 shadow-xl">
          <div class="flex items-center space-x-3">
            <div
              class="animate-spin rounded-full h-8 w-8 border-b-2 border-white"
            ></div>
            <h1 class="text-2xl font-semibold text-white">Chargement...</h1>
          </div>
        </div>
      </div>

      <!-- Main Weather Content -->
      <div v-else-if="weather && !error" class="space-y-6 animate-fade-in">
        <!-- Current Weather Card -->
        <div
          class="bg-white/15 backdrop-blur-md rounded-3xl p-8 shadow-2xl border border-white/20 transform hover:scale-[1.02] transition-all duration-300"
        >
          <div
            class="flex flex-col lg:flex-row justify-between items-start lg:items-center"
            v-for="cloud in arrayClouds"
            :key="cloud.id"
          >
            <div class="flex-1">
              <h2
                class="text-4xl lg:text-5xl font-bold text-white uppercase mb-2"
              >
                {{ city }}
              </h2>
              <p class="text-xl text-white/90 mb-4 capitalize">
                {{ weather.weather[0].description }}
              </p>
              <div
                class="flex flex-col lg:flex-row lg:items-center lg:space-x-6"
              >
                <h3
                  class="text-5xl lg:text-6xl font-bold text-white mb-4 lg:mb-0"
                >
                  {{ Math.round(weather.main.temp) }}°C
                </h3>
                <div class="text-white/80 space-y-1">
                  <p>Ressenti: {{ Math.round(weather.main.feels_like) }}°C</p>
                  <p>Humidité: {{ weather.main.humidity }}%</p>
                  <p>
                    Vent: {{ Math.round(weather.wind?.speed * 3.6 || 0) }} km/h
                  </p>
                </div>
              </div>
            </div>

            <div class="mt-6 lg:mt-0 animate-bounce-slow">
              <img
                class="w-32 h-32 lg:w-40 lg:h-40 drop-shadow-2xl"
                :src="`https://openweathermap.org/img/wn/${cloud.icon}@4x.png`"
                :alt="cloud.description"
              />
            </div>
          </div>
        </div>

        <div class="grid grid-cols-1 xl:grid-cols-3 gap-6">
          <!-- Today's Hourly Forecast -->
          <div
            class="xl:col-span-2 bg-white/15 backdrop-blur-md rounded-3xl p-6 shadow-xl border border-white/20"
          >
            <h2
              class="text-2xl font-bold text-white text-center mb-6 uppercase"
            >
              Prévisions d'aujourd'hui
            </h2>
            <div class="flex gap-4 overflow-x-auto pb-4 custom-scrollbar">
              <div
                class="bg-white/10 backdrop-blur-sm rounded-2xl p-4 min-w-[120px] text-center border border-white/10 hover:bg-white/20 hover:scale-105 transition-all duration-300"
                v-if="todayForecasts && todayForecasts.length"
                v-for="value in todayForecasts"
                :key="value.dt"
              >
                <p class="text-white/80 text-sm mb-2 font-medium">
                  {{
                    new Date(value.dt_txt).toLocaleTimeString("fr-FR", {
                      hour: "2-digit",
                      minute: "2-digit",
                    })
                  }}
                </p>

                <img
                  :src="`https://openweathermap.org/img/wn/${value.weather[0].icon}@2x.png`"
                  :alt="value.weather[0].description"
                  class="w-16 h-16 mx-auto mb-2"
                />

                <p class="text-white font-semibold text-lg">
                  {{ Math.round(value.main.temp) }}°C
                </p>

                <p class="text-white/70 text-xs mt-1 capitalize">
                  {{ value.weather[0].description }}
                </p>
              </div>

              <div
                v-if="!todayForecasts || !todayForecasts.length"
                class="w-full text-center py-8"
              >
                <p class="text-white/80">
                  Aucune prévision disponible pour aujourd'hui
                </p>
              </div>
            </div>
          </div>

          <!-- Daily Forecast -->
          <div
            class="bg-white/15 backdrop-blur-md rounded-3xl p-6 shadow-xl border border-white/20"
          >
            <h2 class="text-xl font-bold text-white text-center mb-6 uppercase">
              Prévisions 5 jours
            </h2>
            <div class="space-y-3">
              <div
                class="bg-white/10 backdrop-blur-sm rounded-2xl p-4 border border-white/10 hover:bg-white/20 hover:scale-[1.02] transition-all duration-300"
                v-for="forecast in dailyForecasts.slice(0, 5)"
                :key="forecast.dt"
              >
                <div class="flex items-center justify-between">
                  <div class="flex items-center space-x-3">
                    <h3 class="text-white font-medium min-w-[50px]">
                      {{ forecast.dayName }}
                    </h3>
                    <img
                      class="w-12 h-12"
                      :src="`https://openweathermap.org/img/wn/${forecast.icon}@2x.png`"
                      :alt="forecast.description"
                    />
                  </div>
                  <div class="text-right">
                    <p class="text-white font-semibold text-lg">
                      {{ Math.round(forecast.temp) }}°C
                    </p>
                    <p class="text-white/70 text-sm capitalize">
                      {{ forecast.description }}
                    </p>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Error State -->
      <div v-else-if="error" class="flex justify-center items-center h-64">
        <div
          class="bg-red-500/20 backdrop-blur-md rounded-2xl p-8 shadow-xl border border-red-300/30 max-w-md"
        >
          <div class="text-center">
            <div class="text-6xl mb-4">⚠️</div>
            <h2 class="text-2xl font-semibold text-white mb-4">
              Oups ! Une erreur s'est produite
            </h2>
            <p class="text-white/90 mb-6">
              {{ error }}
            </p>
            <button
              @click="getWeather()"
              class="bg-white/20 hover:bg-white/30 text-white font-semibold py-2 px-6 rounded-xl transition-all duration-300 border border-white/30 hover:scale-105"
            >
              Réessayer
            </button>
          </div>
        </div>
      </div>

      <!-- No data state -->
      <div v-else class="flex justify-center items-center h-64">
        <div
          class="bg-white/10 backdrop-blur-md rounded-2xl p-8 shadow-xl border border-white/20"
        >
          <h2 class="text-2xl font-semibold text-white text-center">
            Aucune donnée météo disponible
          </h2>
          <p class="text-white/80 text-center mt-2">
            Veuillez réessayer plus tard
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes bounce-slow {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}

.animate-fade-in {
  animation: fade-in 0.6s ease-out;
}

.animate-bounce-slow {
  animation: bounce-slow 3s ease-in-out infinite;
}

.custom-scrollbar {
  scrollbar-width: thin;
  scrollbar-color: rgba(255, 255, 255, 0.3) rgba(255, 255, 255, 0.1);
}

.custom-scrollbar::-webkit-scrollbar {
  height: 6px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 10px;
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.5);
}
</style>
