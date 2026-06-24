<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Cuaca Jakarta</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Cuaca Jakarta</ion-title>
        </ion-toolbar>
      </ion-header>
      <div id="container">
        <div v-if="loading">
          <p>Memuat data...</p>
        </div>
        <div v-else-if="error">
          <p>Gagal memuat data cuaca : {{ error.message }}</p>
        </div>
        <ion-list v-else lines="full">
            <ion-item v-for="(item, index) in forecast" :key="index">
              <ion-label>
                Jam: {{ item.time }}
                <p>Suhu: {{ item.temperature }} {{ item.temperature_type }}</p>
              </ion-label>
            </ion-item>
        </ion-list>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonList, IonItem, IonLabel } from '@ionic/vue';
import { ref, onMounted } from 'vue';

interface ForecastItem{
  time: string;
  temperature: number;
  temperature_type: string;
}

interface weatherApiResponse {
  hourly_units: {
    temperature_2m: string;
  },
  hourly:{
    time: string[];
    temperature_2m: number[];
  }
}

const forecast = ref<ForecastItem[]>([]);
const loading = ref(true);
const error = ref<Error | null>(null);

const apiWeather = "https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m";

function formatHour(timestamp: string): string{
  const date = new Date(timestamp);

  return date.toLocaleTimeString('id-ID', {
    hour: '2-digit',
    minute: '2-digit',
    timeZone: 'Asia/Jakarta'
  });
}

async function fetchWeather() {
  
  try{

    const response = await fetch(apiWeather);

    if(!response.ok){
      throw new Error(`HTTP Error! status : ${response.status}`);
    }

    const data: weatherApiResponse = await response.json();
    const processedData = data.hourly.time.map((time, index) => {
      return {
        time: formatHour(time),
        temperature: data.hourly.temperature_2m[index],
        temperature_type: data.hourly_units.temperature_2m
      }
    }).slice(0,24);
    forecast.value = processedData;
    

  }catch(e){
    
    if(e instanceof Error){
      error.value = e;
    }else{
      error.value = new Error("Terjadi Kesalahan")
    }

  }finally{
    
    loading.value = false;

  }  
}

onMounted(() => {
  fetchWeather();
})

</script>

<style scoped>
#container {
  text-align: center;
  
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  
  color: #8c8c8c;
  
  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>
