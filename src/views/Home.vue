<template>
  <div class="home row no-gutters">
    <div class="col-sm-3">
      <div class="control_section">
        <h1>控制區塊</h1>
      </div>
    </div>
    <div class="col-sm-9">
      <div id="map"></div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex';
import L from 'leaflet';
import 'leaflet.markercluster/dist/leaflet.markercluster';
// @ is an alias to /src

const iconsConfig = {
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41],
};

export default {
  name: 'home',
  components: {},
  data() {
    return {
      data: [],
      icons: {
        grey: this.customIcon('grey'),
        yellow: this.customIcon('yellow'),
        green: this.customIcon('green'),
        red: this.customIcon('red'),
      },
    };
  },
  methods: {
    getMaskData() {
      return new Promise((resolve, reject) => {
        this.$http.get('https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json')
          .then((res) => {
            this.data = res.data.features;
            resolve();
          })
          .catch((err) => {
            console.log(err);
            reject();
          });
      });
    },
    customIcon(color) {
      return L.icon({
        ...iconsConfig,
        iconUrl: `https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-${color}.png`,
      });
    },
    initMap() {
      const map = L.map('map', {
        center: [25.067007, 121.529922],
        zoom: 8,
      });
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      }).addTo(map);
      const markers = new L.MarkerClusterGroup().addTo(map);
      this.data.forEach((item) => {
        const phone = `(${item.properties.phone.substr(0, 2)}) ${item.properties.phone.substr(4, 4)}-${item.properties.phone.substr(8)}`;
        markers.addLayer(L.marker(
          [item.geometry.coordinates[1], item.geometry.coordinates[0]],
          { icon: this.icons.red },
        ).bindPopup(`
          <h3 class="store-title">${item.properties.name}</h3>
          <div class="store-info">
            <div><i class="fas fa-phone"></i> <a href="tel:+886-${item.properties.phone.substring(1).replace(' ', '')}">${phone}</a></div>
            <div><i class="fas fa-directions"></i> <a target="_blank" href="https://www.google.com.tw/maps/place/${item.properties.address}">${item.properties.address}</a></div>
            <hr>
            <div>成人口罩: ${item.properties.mask_adult}</div>
            <div>小孩口罩: ${item.properties.mask_child}</div>
            <div>更新時間: ${item.properties.updated === '' ? '(不明)' : item.properties.updated}</div>
          </div>
        `));
      });
      map.addLayer(markers);
    },
  },
  computed: {
    ...mapGetters(['isLoading']),
  },
  mounted() {
    this.$store.dispatch('updateLoading', true);
    this.getMaskData()
      .then(() => {
        this.initMap();
      })
      .then(() => {
        this.$store.dispatch('updateLoading', false);
      });
  },
};
</script>

<style lang="scss">
#map {
  height: 100vh;
}
.home {
  position: relative;
}
.store-title {
  font-size: 1.5em;
  line-height: 1.7em;
  font-weight: 900;
  margin-bottom: 6px;
  text-align: center;
}
.store-info {
  font-size: 1.3em;
  line-height: 1.5;
}
.fa-phone, .fa-directions {
  font-size: 0.85em;
  font-weight: 600;
}
</style>
