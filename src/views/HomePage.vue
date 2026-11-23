<script setup lang="ts">
// 1. Impor dari Vue 
import { ref, computed, onMounted } from 'vue';

// 2. Impor Komponen UI IONIC 
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonButton,
  IonButtons,
  IonSpinner,
  IonIcon
} from '@ionic/vue';

// 3. Impor Ikon Ionic
import {
  refreshOutline,
  timeOutline,
  thermometerOutline,
  chevronBackOutline,
  chevronForwardOutline,
  calendarOutline
} from 'ionicons/icons';

// Interface Data
interface HourlyItem {
  hourString: string; // e.g. "00.00"
  timeISO: string;
  temperature: number | null;
}

interface DayForecast {
  dateLabel: string; // e.g. "Senin, 23 Nov 2026"
  dateKey: string;   // e.g. "2026-11-23"
  first12Hours: HourlyItem[];  // Jam 00:00 - 11:00
  second12Hours: HourlyItem[]; // Jam 12:00 - 23:00
}

interface RawWeatherData {
  hourly: {
    time: string[];
    temperature_2m: (number | null)[];
  };
  hourly_units: {
    temperature_2m: string;
  };
}

// State
const dailyForecasts = ref<DayForecast[]>([]);
const currentDayIndex = ref<number>(0);
const unit = ref<string>('°C');
const loading = ref<boolean>(true);
const error = ref<string | null>(null);

// Data hari yang sedang aktif
const activeDay = computed(() => {
  if (dailyForecasts.value.length === 0) return null;
  return dailyForecasts.value[currentDayIndex.value] || null;
});

// Navigasi Hari
function prevDay() {
  if (currentDayIndex.value > 0) {
    currentDayIndex.value--;
  }
}

function nextDay() {
  if (currentDayIndex.value < dailyForecasts.value.length - 1) {
    currentDayIndex.value++;
  }
}

// Fungsi Fetch & Pemrosesan Data per Hari (12 jam kiri, 12 jam kanan)
async function fetchWeatherData() {
  loading.value = true;
  error.value = null;

  const apiUrl = "https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m&timezone=auto";

  try {
    const response = await fetch(apiUrl);
    if (!response.ok) {
      throw new Error("Gagal mengambil data dari API");
    }

    const data: RawWeatherData = await response.json();
    unit.value = data.hourly_units.temperature_2m || '°C';

    // Kelompokkan data per tanggal (YYYY-MM-DD)
    const groupedByDate: { [key: string]: HourlyItem[] } = {};

    data.hourly.time.forEach((timeStr, index) => {
      const dateObj = new Date(timeStr);
      // Buat date key berbasis tanggal lokal
      const year = dateObj.getFullYear();
      const month = String(dateObj.getMonth() + 1).padStart(2, '0');
      const date = String(dateObj.getDate()).padStart(2, '0');
      const dateKey = `${year}-${month}-${date}`;

      const hourString = dateObj.toLocaleTimeString('id-ID', {
        hour: '2-digit',
        minute: '2-digit'
      });

      const item: HourlyItem = {
        hourString,
        timeISO: timeStr,
        temperature: data.hourly.temperature_2m[index] ?? null
      };

      if (!groupedByDate[dateKey]) {
        groupedByDate[dateKey] = [];
      }
      groupedByDate[dateKey].push(item);
    });

    // Susun array DayForecast: 12 jam pertama (kiri) & 12 jam kedua (kanan)
    const days: DayForecast[] = Object.keys(groupedByDate).map(dateKey => {
      const items = groupedByDate[dateKey];
      const sampleDate = new Date(items[0].timeISO);
      
      const dateLabel = sampleDate.toLocaleDateString('id-ID', {
        weekday: 'long',
        day: 'numeric',
        month: 'short',
        year: 'numeric'
      });

      return {
        dateLabel,
        dateKey,
        first12Hours: items.slice(0, 12),  // 00.00 - 11.00
        second12Hours: items.slice(12, 24) // 12.00 - 23.00
      };
    });

    dailyForecasts.value = days;
    currentDayIndex.value = 0;

  } catch (err) {
    if (err instanceof Error) {
      error.value = err.message;
    } else {
      error.value = "Terjadi kesalahan yang tidak diketahui";
    }
  } finally {
    loading.value = false;
  }
}

onMounted(() => {
  fetchWeatherData();
});
</script>


<template>
  <ion-page class="weather-page">

    <ion-header class="ion-no-border">
      <ion-toolbar color="primary" class="main-toolbar">
        <ion-title class="app-header-title">Weather App Ionic</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="fetchWeatherData" :disabled="loading" fill="clear">
            <ion-spinner v-if="loading" name="crescent" color="light"></ion-spinner>
            <ion-icon v-else :icon="refreshOutline" class="refresh-icon"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="page-content">
      
      <div class="content-wrapper">
        <!-- Kartu Header Informasi Lokasi & Identitas -->
        <div class="header-card">
          <div class="header-card-inner">
            <div class="city-badge-container">
              <span class="city-badge">DKI Jakarta</span>
            </div>
            <h1 class="main-heading">Prakiraan Cuaca Per Jam</h1>
            <div class="author-box">
              <p class="author-name">Rhenald Aditya B.P.  NIM: 048907153</p>
              <p class="univ-name">Sistem Informasi Universitas Terbuka</p>
            </div>
          </div>
        </div>

        <!-- Pesan Error -->
        <div v-if="error" class="error-box">
          <h3>Terjadi Kesalahan</h3>
          <p>{{ error }}</p>
          <ion-button size="small" color="danger" @click="fetchWeatherData">Coba Lagi</ion-button>
        </div>

        <!-- Loading State -->
        <div v-else-if="loading && dailyForecasts.length === 0" class="loading-state">
          <ion-spinner name="crescent" color="primary"></ion-spinner>
          <p>Memuat data prakiraan cuaca...</p>
        </div>

        <!-- Kartu Utama Tampilan 2 Kolom (12 Jam Kiri + 12 Jam Kanan) -->
        <div v-else-if="activeDay" class="main-forecast-card">
          
          <!-- Banner Tanggal Hari yang Sedang Dilihat -->
          <div class="active-date-banner">
            <!-- <ion-icon :icon="calendarOutline" class="calendar-icon"></ion-icon> -->
            <span class="active-date-text">{{ activeDay.dateLabel }}</span>
            <span class="day-badge">Hari ke-{{ currentDayIndex + 1 }} dari {{ dailyForecasts.length }}</span>
          </div>

          <!-- Wadah Grid 2 Kolom (Kiri: 00:00-11:00, Kanan: 12:00-23:00) -->
          <div class="two-column-grid">
            
            <!-- Kolom 1 (12 Baris Pertama: 00.00 - 11.00) -->
            <div class="sub-table-card">
              <div class="sub-table-header">
                <span class="sub-title">Jam (00.00 - 11.00)</span>
              </div>
              <div class="sub-table-subheader">
                <span class="col-head">Waktu</span>
                <span class="col-head text-right">Suhu</span>
              </div>
              <div class="rows-container">
                <div 
                  v-for="(row, idx) in activeDay.first12Hours" 
                  :key="row.timeISO"
                  class="data-row"
                  :class="{ 'row-even': idx % 2 === 0 }"
                >
                  <span class="jam-text">{{ row.hourString }} WIB</span>
                  <span class="temp">
                    {{ row.temperature !== null ? `${row.temperature} ${unit}` : 'N/A' }}
                  </span>
                </div>
              </div>
            </div>

            <!-- Kolom 2 (12 Baris Kedua: 12.00 - 23.00) -->
            <div class="sub-table-card">
              <div class="sub-table-header sub-table-header-afternoon">
                <span class="sub-title">Jam (12.00 - 23.00)</span>
              </div>
              <div class="sub-table-subheader">
                <span class="col-head">Waktu</span>
                <span class="col-head text-right">Suhu</span>
              </div>
              <div class="rows-container">
                <div 
                  v-for="(row, idx) in activeDay.second12Hours" 
                  :key="row.timeISO"
                  class="data-row"
                  :class="{ 'row-even': idx % 2 === 0 }"
                >
                  <span class="jam-text">{{ row.hourString }} WIB</span>
                  <span class="temp">
                    {{ row.temperature !== null ? `${row.temperature} ${unit}` : 'N/A' }}
                  </span>
                </div>
              </div>
            </div>

          </div>

          <!-- Navigasi Hari di Bawah Tabel -->
          <div class="day-navigation-footer">
            <button 
              class="nav-btn prev-btn" 
              :disabled="currentDayIndex === 0" 
              @click="prevDay"
              aria-label="Hari Sebelumnya"
            >
              <ion-icon :icon="chevronBackOutline" class="nav-icon"></ion-icon>
              <span>Hari Sebelumnya</span>
            </button>

            <div class="nav-page-indicator">
              <strong>{{ currentDayIndex + 1 }}</strong> / {{ dailyForecasts.length }}
            </div>

            <button 
              class="nav-btn next-btn" 
              :disabled="currentDayIndex >= dailyForecasts.length - 1" 
              @click="nextDay"
              aria-label="Hari Berikutnya"
            >
              <span>Hari Berikutnya</span>
              <ion-icon :icon="chevronForwardOutline" class="nav-icon"></ion-icon>
            </button>
          </div>

        </div>
      </div>

    </ion-content>
  </ion-page>
</template>


<style scoped>
.weather-page {
  background-color: #f1f5f9;
}

.page-content {
  --background: #f1f5f9;
  background-color: #f1f5f9;
}

.main-toolbar {
  --background: #2563eb;
  --color: #ffffff;
}

.app-header-title {
  font-weight: 700;
  letter-spacing: 0.5px;
}

.refresh-icon {
  font-size: 1.4rem;
  color: #ffffff;
}

/* Lebar kontainer responsif diperlebar agar muat 2 kolom berdampingan secara lega */
.content-wrapper {
  max-width: 860px;
  margin: 0 auto;
  padding: 16px 14px 36px 14px;
}

/* Card Header Identitas */
.header-card {
  background: #ffffff;
  border-radius: 16px;
  border: 1px solid #e2e8f0;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  margin-bottom: 16px;
  overflow: hidden;
}

.header-card-inner {
  padding: 16px 18px;
}

.city-badge-container {
  margin-bottom: 8px;
}

.city-badge {
  display: inline-block;
  background-color: #eff6ff;
  color: #2563eb;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  letter-spacing: 0.6px;
  border: 1px solid #dbeafe;
}

.main-heading {
  font-size: 1.35rem;
  font-weight: 800;
  color: #0f172a;
  margin: 0 0 10px 0;
  line-height: 1.25;
}

.author-box {
  border-top: 1px solid #f1f5f9;
  padding-top: 10px;
}

.author-name {
  font-size: 0.9rem;
  font-weight: 600;
  color: #334155;
  margin: 0;
}

.univ-name {
  font-size: 0.8rem;
  color: #64748b;
  margin: 2px 0 0 0;
}

/* Main Forecast Card */
.main-forecast-card {
  background: #ffffff;
  border-radius: 16px;
  border: 1px solid #e2e8f0;
  box-shadow: 0 4px 14px rgba(0, 0, 0, 0.05);
  overflow: hidden;
}

/* Banner Tanggal Aktif */
.active-date-banner {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
  padding: 14px 18px;
  background: linear-gradient(135deg, #1e40af 0%, #2563eb 100%);
  color: #ffffff;
}

.calendar-icon {
  font-size: 1.25rem;
}

.active-date-text {
  font-size: 1.05rem;
  font-weight: 700;
  letter-spacing: 0.3px;
  flex: 1;
}

.day-badge {
  background: rgba(255, 255, 255, 0.2);
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 0.78rem;
  font-weight: 600;
  backdrop-filter: blur(4px);
}

/* Grid 2 Kolom (Berdampingan) */
.two-column-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 14px;
  padding: 16px;
  background-color: #f8fafc;
}

/* Responsif: Jika di layar HP sangat sempit, tetap rapi */
@media (max-width: 580px) {
  .two-column-grid {
    grid-template-columns: 1fr;
    gap: 16px;
    padding: 12px;
  }
}

/* Sub-tabel Card untuk 12 Jam */
.sub-table-card {
  background: #ffffff;
  border-radius: 12px;
  border: 1px solid #e2e8f0;
  overflow: hidden;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.03);
}

.sub-table-header {
  padding: 10px 14px;
  background-color: #eff6ff;
  border-bottom: 1px solid #dbeafe;
}

.sub-table-header-afternoon {
  background-color: #fef3c7;
  border-bottom: 1px solid #fde68a;
}

.sub-title {
  font-size: 0.82rem;
  font-weight: 800;
  color: #1e3a8a;
  text-transform: uppercase;
  letter-spacing: 0.4px;
}

.sub-table-header-afternoon .sub-title {
  color: #92400e;
}

.sub-table-subheader {
  display: flex;
  justify-content: space-between;
  padding: 8px 14px;
  background-color: #f8fafc;
  border-bottom: 1px solid #e2e8f0;
}

.col-head {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  font-size: 0.75rem;
  font-weight: 700;
  color: #64748b;
  text-transform: uppercase;
}

.text-right {
  text-align: right;
}

/* Baris Data */
.rows-container {
  display: flex;
  flex-direction: column;
}

.data-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 7px 14px;
  border-bottom: 1px solid #f1f5f9;
  transition: background-color 0.15s ease;
}

.row-even {
  background-color: #fbfcfe;
}

.data-row:hover {
  background-color: #eff6ff;
}

.jam-text {
  font-size: 0.88rem;
  font-weight: 700;
  color: #1e293b;
}

/* Pil Suhu Berwarna Kontras */
.temp-pill {
  display: inline-block;
  background-color: #2563eb;
  color: #ffffff;
  font-size: 0.84rem;
  font-weight: 700;
  padding: 3px 10px;
  border-radius: 8px;
  min-width: 60px;
  text-align: center;
}

/* Footer Navigasi Hari dengan Tombol Panah Kanan/Kiri */
.day-navigation-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 14px 18px;
  background-color: #ffffff;
  border-top: 1px solid #e2e8f0;
}

.nav-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background-color: #2563eb;
  color: #ffffff;
  border: none;
  padding: 8px 14px;
  border-radius: 10px;
  font-size: 0.84rem;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 2px 6px rgba(37, 99, 235, 0.25);
  transition: all 0.2s ease;
}

.nav-btn:hover:not(:disabled) {
  background-color: #1d4ed8;
  transform: translateY(-1px);
}

.nav-btn:disabled {
  background-color: #cbd5e1;
  color: #94a3b8;
  box-shadow: none;
  cursor: not-allowed;
  opacity: 0.7;
}

.nav-icon {
  font-size: 1.15rem;
}

.nav-page-indicator {
  font-size: 0.88rem;
  color: #475569;
  font-weight: 500;
}

.nav-page-indicator strong {
  color: #2563eb;
  font-size: 1rem;
}

/* Loading & Error */
.loading-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px 20px;
  gap: 12px;
  color: #64748b;
  font-weight: 500;
}

.error-box {
  background: #fef2f2;
  border: 1px solid #fee2e2;
  border-radius: 14px;
  padding: 20px;
  text-align: center;
  color: #991b1b;
}

.error-box h3 {
  margin: 0 0 8px 0;
  font-weight: 700;
}

.error-box p {
  margin: 0 0 14px 0;
  font-size: 0.9rem;
}
</style>
