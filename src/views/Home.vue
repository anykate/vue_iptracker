<template>
  <div class="flex flex-col h-screen max-h-screen">
    <div
      class="
        flex
        justify-center
        relative
        z-20
        bg-hero-pattern bg-cover
        px-4
        pt-8
        pb-32
      "
    >
      <div class="w-full max-w-screen-sm">
        <h1 class="text-3xl text-white text-center font-bold pb-4">
          IP Address Tracker
        </h1>
        <div class="flex items-center">
          <input
            v-model="queryIp"
            type="text"
            class="
              flex-1
              py-3
              px-2
              rounded-tl-md rounded-bl-md
              focus:outline-none
            "
            placeholder="Search for IP Address..."
          />
          <svg
            @click="getIpInfo"
            xmlns="http://www.w3.org/2000/svg"
            class="
              h-12
              w-12
              px-2
              bg-black
              text-white
              cursor-pointer
              rounded-tr-md rounded-br-md
            "
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M9 5l7 7-7 7"
            />
          </svg>
        </div>
        <div class="text-white text-justify">
          OR leave this field empty to get info about your location's public IP
          address...
        </div>
      </div>
      <IPInfo v-if="ipInfo" :ipInfo="ipInfo" />
    </div>
    <div id="mapid" class="h-full z-10"></div>
  </div>
</template>

<script>
import IPInfo from '@/components/IPInfo.vue'
import leaflet from 'leaflet'
import { onMounted, ref } from 'vue'
import axios from 'axios'
export default {
  name: 'Home',
  components: {
    IPInfo
  },
  setup() {
    let mymap;
    var MAPBOX_ACCESS_TOKEN = "YOUR_MAPBOX_ACCESS_TOKEN_GOES_HERE"
    var IPIFY_ORG_API_KEY = "YOUR_IPIFY_API_KEY_GOES_HERE"
    const queryIp = ref('')
    const ipInfo = ref(null)
    onMounted(() => {
      mymap = leaflet.map('mapid').setView([19.218330, 72.978088], 13);
      leaflet.tileLayer(`https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=${MAPBOX_ACCESS_TOKEN}`, {
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
        maxZoom: 18,
        id: 'mapbox/streets-v11',
        tileSize: 512,
        zoomOffset: -1,
        accessToken: `${MAPBOX_ACCESS_TOKEN}`
      }).addTo(mymap);
    })
    const getIpInfo = async () => {
      try {
        const data = await axios.get(`https://geo.ipify.org/api/v1?apiKey=${IPIFY_ORG_API_KEY}&ipAddress=${queryIp.value}`)
        const result = await data.data
        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng
        }
        leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(mymap);
        mymap.setView([ipInfo.value.lat, ipInfo.value.lng], 13);
      } catch (error) {
        alert(error.message)
      }
    }
    return {
      queryIp,
      ipInfo,
      getIpInfo
    }
  }
}
</script>
