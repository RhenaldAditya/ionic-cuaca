<script setup lang="ts">
// 1. Impor dari Vue 
import { ref, onMounted } from 'vue';

// 2. Impor Komponen UI IONIC 
// impor setiap komponen yang akan di gunakan pada template
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonList,
  IonItem,
  IonLabel,
  IonButton,
  IonButtons,
  IonSpinner,
  IonIcon,
  IonListHeader
} from '@ionic/vue';

// 3. Impor Ikon untuk tombol refresh data cuaca
import { refreshOutline } from 'ionicons/icons';


// Tentukan "bentuk" data yang di proses
interface ForecastItem {
  time: string;
  temperature: number | null; // 
}

// Tentukan "bentuk" data mentah dari API
interface RawWeatherData {
  hourly: {
    time: string[]; // day, month, year, time
    temperature_2m: (number | null)[]; // data suhu
  };
  hourly_units: {
    temperature_2m: string; // satuan suhu Celcius
  };
}

// bagian Reactive State Vue
const forecasts = ref<ForecastItem[]>([]); // daftar data cuaca
const unit = ref<string>(''); // satuan suhu
const loading = ref<boolean>(true); // state penanda loading data
const error = ref<string | null>(null); // menyimpan pesan error

// fungsi untuk mengambil dan memproses data 
async function fetchWeatherData() {
  // dipanggil setiap kali fungsi fetchWeatherData dipanggil untuk membersihkan data dan pesan error
  loading.value = true;
  error.value = null;

  // API data cuaca dengan zona waktu jakarta
  const apiUrl = "https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m&timezone=auto";

  // blok untuk eksesui data
  try {
    // await berfungsi agar komputer menunggu data yg sedang diambil dari API cuaca
    const response = await fetch(apiUrl);
    // Jika komputer gagal mengambil data, maka mengeksekusi fungsi ini
    if (!response.ok) {
      throw new Error("Gagal mengambil data dari API");
    }

    // ubah data JSON menjadi object
    const data: RawWeatherData = await response.json();

    // Mengubah data mentah menjadi format yang sesuatu ketentuan / ForecastItem
    const processedData = data.hourly.time.map((time, index) => {
      const formattedTime = new Date(time).toLocaleString('id-ID', {
        day: 'numeric',
        month: 'short',
        year: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      });

      return {
        time: formattedTime,
        temperature: data.hourly.temperature_2m[index] ?? null
      };
    });

    // memasukkan data yang telah rapi ke forecast
    forecasts.value = processedData;
    unit.value = data.hourly_units.temperature_2m;

  } 
  // jika terjadi gagal dalam blok try, maka komputer menampilkan pesan error
  catch (err) {
    if (err instanceof Error) {
      error.value = err.message;
    } else {
      error.value = "Terjadi kesalahan yang tidak diketahui";
    }
  } finally {
    loading.value = false;
  }
}

// Jalankan fungsi saat komponen dimuat
onMounted(() => {
  fetchWeatherData();
});
</script>


<template>
  <ion-page>

    <ion-header>
      <ion-toolbar>
        <ion-title>Prakiraan Cuaca Jakarta</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="fetchWeatherData" :disabled="loading">
            <ion-spinner v-if="loading" name="crescent"></ion-spinner>
            <ion-icon v-else :icon="refreshOutline"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      
      <div v-if="error" class="error-container">
        <ion-label color="danger">
          <h3>Terjadi kesalahan</h3>
          <p>{{ error }}</p>
        </ion-label>
      </div>

      <ion-list v-else>
        <ion-list-header>
            <ion-label>
              <h2>Prakiraan Cuaca Jakarta</h2>
              <p>(Rhenald Aditya BP - 048907153)</p>
            </ion-label>
        </ion-list-header>

        <ion-item lines="full">
          <ion-label><strong>Waktu Pengukuran</strong></ion-label>
          <ion-label slot="end"><strong>Suhu per Jam</strong></ion-label>
        </ion-item>
        
        <ion-item v-for="item in forecasts" :key="item.time">
          <ion-label>{{ item.time }}</ion-label>
          <ion-label slot="end">
            {{ item.temperature !== null ? `${item.temperature} ${unit}` : 'N/A' }}
          </ion-label>
        </ion-item>
      </ion-list>

      <div v-if="loading && forecasts.length === 0" class="loading-container">
        <ion-spinner name="crescent"></ion-spinner>
        <p>Memuat data...</p>
      </div>

    </ion-content>
  </ion-page>
</template>


<style scoped>
.loading-container, .error-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 60vh;
  text-align: center;
}

ion-list-header h2 {
  font-size: 1.2rem;
  font-weight: bold;
}
ion-list-header p {
  font-size: 0.8rem;
}

ion-label[slot="end"] {
  text-align: right;
}
</style>
