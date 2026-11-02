<template>
  <ion-page>
    <ion-header class="ion-no-border">
      <ion-toolbar class="custom-toolbar">
        <ion-title>Cuaca Jakarta</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="custom-content">

      <div class="hero-section">
        <h1>Prakiraan Cuaca Jakarta</h1>
        <p>Data temperatur per jam dari Open-Meteo API</p>
      </div>

      <ion-card v-if="loading" class="modern-card">
        <ion-card-content>
          <div class="loading-box">
            <ion-spinner name="crescent"></ion-spinner>
            <p>Memuat data cuaca...</p>
          </div>
        </ion-card-content>
      </ion-card>

      <ion-card v-if="errorMessage" class="modern-card error-card">
        <ion-card-content class="error-text">
          {{ errorMessage }}
        </ion-card-content>
      </ion-card>

      <ion-card v-if="!loading && !errorMessage" class="modern-card">
        <ion-card-header>
          <ion-card-title>10 Data Pertama</ion-card-title>
          <ion-card-subtitle>
            Waktu pengukuran dan suhu per jam
          </ion-card-subtitle>
        </ion-card-header>

        <ion-card-content>
          <div class="table-wrapper">
            <table class="weather-table">
              <thead>
                <tr>
                  <th>No</th>
                  <th>Waktu</th>
                  <th>Suhu</th>
                </tr>
              </thead>

              <tbody>
                <tr
                  v-for="(item, index) in weatherData"
                  :key="index"
                >
                  <td class="number-col">{{ index + 1 }}</td>
                  <td>{{ formatWaktu(item.time) }}</td>
                  <td class="temp-col">
                    <span class="temp-badge">
                      {{ item.temperature }} {{ temperatureUnit }}
                    </span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </ion-card-content>
      </ion-card>

      <div class="button-wrapper">
        <ion-button
          expand="block"
          class="reload-button"
          @click="ambilDataCuaca"
        >
          Muat Ulang Data
        </ion-button>
      </div>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardSubtitle,
  IonCardContent,
  IonButton,
  IonSpinner
} from '@ionic/vue';

import { onMounted, ref } from 'vue';

interface WeatherItem {
  time: string;
  temperature: number;
}

interface WeatherResponse {
  hourly: {
    time: string[];
    temperature_2m: number[];
  };

  hourly_units: {
    time: string;
    temperature_2m: string;
  };
}

const apiUrl =
  'https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m';

const weatherData = ref<WeatherItem[]>([]);
const temperatureUnit = ref<string>('°C');
const loading = ref<boolean>(false);
const errorMessage = ref<string>('');

async function ambilDataCuaca(): Promise<void> {
  loading.value = true;
  errorMessage.value = '';

  try {
    const response = await fetch(apiUrl);

    if (!response.ok) {
      throw new Error(`Gagal mengambil data. Status: ${response.status}`);
    }

    const data: WeatherResponse = await response.json();

    console.log('Data API:', data);

    const waktu = data.hourly.time.slice(0, 10);
    const suhu = data.hourly.temperature_2m.slice(0, 10);

    weatherData.value = waktu.map((time, index) => {
      return {
        time: time,
        temperature: suhu[index]
      };
    });

    temperatureUnit.value = data.hourly_units.temperature_2m;
  } catch (error) {
    console.error(error);
    errorMessage.value =
      'Data cuaca gagal dimuat. Pastikan perangkat terhubung ke internet.';
  } finally {
    loading.value = false;
  }
}

function formatWaktu(time: string): string {
  const date = new Date(time);

  return date.toLocaleString('id-ID', {
    day: '2-digit',
    month: 'short',
    year: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  });
}

onMounted(() => {
  ambilDataCuaca();
});
</script>

<style scoped>
.custom-toolbar {
  --background: #ffffff;
  --color: #1e293b;
  --border-color: transparent;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.04);
}

.custom-content {
  --background: linear-gradient(180deg, #eef4ff 0%, #f8fbff 40%, #ffffff 100%);
  color: #1e293b;
}

.hero-section {
  text-align: center;
  padding: 28px 18px 10px;
}

.hero-section h1 {
  margin: 0;
  font-size: 28px;
  font-weight: 700;
  color: #0f172a;
}

.hero-section p {
  margin-top: 8px;
  color: #64748b;
  font-size: 15px;
}

.modern-card {
  margin: 16px;
  border-radius: 22px;
  box-shadow: 0 12px 30px rgba(37, 99, 235, 0.10);
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(6px);
}

.error-card {
  border: 1px solid #fecaca;
}

.error-text {
  color: #dc2626;
  text-align: center;
  font-weight: 500;
}

.loading-box {
  text-align: center;
  padding: 10px 0;
}

.loading-box p {
  margin-top: 12px;
  color: #64748b;
}

.table-wrapper {
  overflow-x: auto;
}

.weather-table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  overflow: hidden;
  border-radius: 16px;
  background: #ffffff;
}

.weather-table thead th {
  background: linear-gradient(135deg, #60a5fa, #2563eb);
  color: #ffffff;
  font-weight: 600;
  padding: 14px 12px;
  text-align: center;
  font-size: 14px;
}

.weather-table thead th:first-child {
  border-top-left-radius: 16px;
}

.weather-table thead th:last-child {
  border-top-right-radius: 16px;
}

.weather-table tbody td {
  padding: 14px 12px;
  border-bottom: 1px solid #e2e8f0;
  color: #334155;
  font-size: 14px;
  background: #ffffff;
}

.weather-table tbody tr:nth-child(even) td {
  background: #f8fbff;
}

.weather-table tbody tr:hover td {
  background: #eef4ff;
  transition: 0.2s ease-in-out;
}

.number-col {
  text-align: center;
  width: 60px;
  font-weight: 600;
  color: #2563eb;
}

.temp-col {
  text-align: center;
  width: 120px;
}

.temp-badge {
  display: inline-block;
  padding: 8px 14px;
  border-radius: 999px;
  background: linear-gradient(135deg, #dbeafe, #bfdbfe);
  color: #1d4ed8;
  font-weight: 600;
  font-size: 13px;
}

.button-wrapper {
  padding: 0 16px 24px;
}

.reload-button {
  --background: linear-gradient(135deg, #3b82f6, #2563eb);
  --color: #ffffff;
  --border-radius: 16px;
  --box-shadow: 0 10px 22px rgba(37, 99, 235, 0.25);
  font-weight: 600;
  height: 50px;
}

@media (max-width: 768px) {
  .hero-section h1 {
    font-size: 23px;
  }

  .hero-section p {
    font-size: 14px;
  }

  .weather-table thead th,
  .weather-table tbody td {
    padding: 11px 8px;
    font-size: 13px;
  }

  .temp-badge {
    padding: 6px 10px;
    font-size: 12px;
  }
}
</style>