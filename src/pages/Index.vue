<template>
  <q-page class="flex flex-center page_container">
    <q-google-map 
    :center="{ lat: center.lat, lng: center.lng }" 
    :zoom="13" 
    :style="{ width: '100%', height: '100%' }" 
    :options="mapOptions">
      <q-google-map-marker 
        :icon="locationSVG" 
        :position="center" 
        :draggable="false">
      </q-google-map-marker>
      <q-google-map-marker 
        v-for="m in hub_list" 
        v-bind:key="m.id" 
        @click="clickMarker(m)" 
        :position="m.position"
        :clickable="true"
        :draggable="false" 
        :icon="hubIcon(m)" 
        :label="hubLabel(m)">
      </q-google-map-marker>
    </q-google-map>
    <q-list class="hub-list">
      <q-item-label
        header
        class="text-grey-8">
        Hubs Near You
      </q-item-label>
      <q-item
        tag="b"
        clickable
        target="_blank"
        v-for="(m) in filterHubList"
        v-bind:key="m.id">
        <div class="hub-item">
          <q-item-label>{{ m.name }} {{ m.distance }}</q-item-label>
          <svg width="40" height="40" xmlns="http://www.w3.org/2000/svg">
            <g id="Layer">
              <title>Circle</title>
              <ellipse stroke-width="4" ry="15" rx="15" id="svg_1" cy="20" cx="20" stroke="#ffffff" fill="#f19727" />
              <text transform="matrix(1 0 0 1 0 0)" xml:space="preserve" text-anchor="start" 
              font-family="sans-serif" font-size="18" id="svg_2" y="26.07468" x="13.50075" 
              stroke-width="0" stroke="#ffffff" fill="#ffffff">{{m.marker_label}}</text>
            </g>
          </svg>
        </div>
      </q-item>
    </q-list>
  </q-page>
</template>

<script>
  import hubsData from '../assets/hubs.json';
  import locationSVG from '../assets/my_location.svg';
  import MarkerSVG from '../assets/hub_marker.svg';
  import axios from "axios";

  export default {
    name: 'HubPage',
    data() {
      return {
        center: {
          lat: 1.333333,
          lng: 133.222222
        },
        content_height: 0,
        hub_list: hubsData['data'],
        mapOptions: {
          zoomControl: false,
          streetViewControl: false,
          mapTypeControl: false,
          fullscreen: false
        },
        locationSVG: locationSVG,
        selected_hub: null
      }
    },

    created() {
      this.content_height = window.innerHeight - 100;
    },

    async mounted() {
      this.getMyPosition();
    },

    computed: {
      filterHubList() {
        if (this.selected_hub) {
          return this.hub_list.filter(item => item.id !== this.selected_hub.id);
        }
        return this.hub_list;
      },
    },

    methods: {
      clickMarker(obj) {
        if (this.selected_hub && this.selected_hub.id === obj.id) {
          this.selected_hub = null;
        } else {
          this.selected_hub = obj;
        }
      },
      
      getMyPosition() {
        navigator.geolocation.getCurrentPosition(
          position => {
            this.center = {
              lat: position.coords.latitude,
              lng: position.coords.longitude
            };
            this.updateHubData();
          },
          error => {
            console.log(error.message);
          },
        );
      },

      async updateHubData() {
        for (let i = 0; i < this.hub_list.length; i++) {
          const item = this.hub_list[i];
          item.distance = 0;
          item.position = await this.getLat_Lng(item.name);
          if (item.position) {
            item.distance = Math.pow(item.position.lat - this.center.lat, 2) + 
            Math.pow(item.position.lng - this.center.lng, 2);
          }
          this.hub_list[i] = item;
        }

        this.hub_list.sort((a, b) => {
          return a.distance < b.distance ? -1 : a.distance > b.distance ? 1 : 0;
        });

        for (let i = 0; i < this.hub_list.length; i++) {
          this.hub_list[i].marker_label = this.getHubLabel(i);
        }
      },
      
      hubIcon(m) {
        if (this.selected_hub && this.selected_hub.id === m.id) {
          return null;
        }

        return MarkerSVG;
      },

      getHubLabel(index) {
        return String.fromCharCode(65 + index);
      },
      
      async getLat_Lng(hub_address) {
        try {
          var { data } = await axios.get(
            `https://maps.googleapis.com/maps/api/geocode/json?address=${hub_address}&key=AIzaSyA4WqhfBUyBLtcPvOIlSM5GDeEZzT2PZs4`
          );
          return data.results[0].geometry.location;
        } catch (error) {
          console.log(error);
          return null;
        }
      },

      hubLabel(m) {
        if (this.selected_hub && this.selected_hub.id === m.id) {
          return null;
        }

        return {
          fontSize: '16px',
          text: m.marker_label ?? '',
          color: 'white'
        }
      }
    }
  }
</script>
<style>
  .vue-map-container {
    position: unset !important;
  }

  .hub-list {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    overflow-y: scroll;
    background: white;
    border-left: 1px solid gray;
    width: 300px;
  }

  .hub-item {
    display: flex;
    width: 100%;
    padding-top: 10px;
    padding-bottom: 10px;
    align-items: center;
    flex-direction: row;
    justify-content: space-between;
  }

  @media only screen and (max-width: 560px) {
    .hub-list {
      top: initial;
      right: 0;
      bottom: 0;
      left: 0;
      height: 200px;
      border-top: 1px solid gray;
      border-left: 0;
      width: initial;
    }
  } 
</style>
