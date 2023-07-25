<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- Search & Results -->
    <div class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32">
      <!-- Search Input Field -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>
        <div class="flex">
          <input 
            class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none" 
            type="text" 
            placeholder="Search for any IP address or leave empty to get your IP info"
            v-model="queryIp"
            @keypress.enter="getIpInfo">
          <i class="cursor-pointer bg-black text-white px-4 rounded-tr-md rounded-br-md flex items-center fas fa-chevron-right" @click="getIpInfo"></i>
        </div>
      </div>
      <!-- IP Info-->
      <IPInfo v-if="ipInfo" :ipInfo="ipInfo"/>
    </div>

    <!-- Map -->
    <div id="map" class="h-full z-10"></div>
  </div>
</template>

<script setup>
import IPInfo from '@/components/IPInfo.vue';
import env from '@/env';
import { onMounted, ref } from 'vue';
import axios from 'axios';

let map;
const queryIp = ref('');
const ipInfo = ref(null);

//Map initialization
onMounted(() => {
  main();
  async function main() {
      await ymaps3.ready;
      const {YMapControls} = ymaps3;
      const {YMapZoomControl} = await ymaps3.import('@yandex/ymaps3-controls@0.0.1');
      map = new ymaps3.YMap(document.getElementById('map'), {
          location: {
              center: [-93.265174, 44.977606],
              zoom: 10
          }
      });
      map.addChild(new ymaps3.YMapDefaultSchemeLayer());
      map.addChild(new YMapControls({position: 'left'}).addChild(new YMapZoomControl({})));
  }
})

const getIpInfo = async () => {
  try {
    const data = await axios.get(`https://geo.ipify.org/api/v2/country,city?apiKey=${env.api_key}&ipAddress=${queryIp.value}`);
    const result = data.data;

    ipInfo.value = {
      address: result.ip,
      state: result.location.region,
      city: result.location.city,
      timezone: result.location.timezone,
      isp: result.isp,
      lat: result.location.lat,
      lng: result.location.lng,
    }
    map.setLocation({
      center: [result.location.lng, result.location.lat],
      zoom: 14
    });

  } catch (err) {
      console.log(err)
      alert(err.message);
  };
};

</script>
