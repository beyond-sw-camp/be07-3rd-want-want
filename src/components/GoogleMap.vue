<template>
  <div class="container">
    <div class="map-container" ref="mapContainer"></div>
    <div class="search-container">
      <input id="place" type="text" placeholder="Enter a location" />
      <div v-if="placeInfo" class="place-info">
        <p><strong>Place Name:</strong> {{ placeInfo.name }}</p>
        <p><strong>Latitude:</strong> {{ placeInfo.lat }}</p>
        <p><strong>Longitude:</strong> {{ placeInfo.lng }}</p>
      </div>
      <v-row justify="left">
        <v-btn color="secondary" @click="savePlace">SAVE</v-btn>
      </v-row>
    </div>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue';
import { loadGoogleMapsApi } from '@/plugins/google-maps'; // plugins 폴더의 경로에 맞게 수정
import axios from "axios";

export default {
  emits: ['place-selected'],
  props: {
    projectId: {
      type: Number,
      required: true,
    }
  },
  setup(props, {emit}) {
    const mapContainer = ref(null);
    const apiKey = process.env.VUE_APP_GOOGLE_MAPS_API_KEY; // 환경 변수에서 API 키 가져오기
    const placeInfo = ref(null); // 핀의 정보를 저장할 변수
    let map, autocomplete, marker; // map, autocomplete, marker 변수 선언

    // 나라 이름과 나라 코드 간의 매핑
    const countryCodeMap = {
      '한국': 'kr',
      '일본': 'jp',
      '미국': 'us',
      'China': 'cn', // 예시 추가
      'Germany': 'de' // 예시 추가
    };

    const getCountryCode = (name) => {
      return countryCodeMap[name] || 'kr'; // 기본값으로 'kr' 사용
    };

    const initializeMap = (countryCode) => {
      const centers = {
        'kr': {lat: 37.5665, lng: 126.9780}, // 서울
        'us': {lat: 37.7749, lng: -122.4194}, // 샌프란시스코
        'jp': {lat: 35.6895, lng: 139.6917}, // 도쿄
        'fr': {lat: 48.8566, lng: 2.3522}, // 파리
        'tw': {lat: 25.0330, lng: 121.5654}, // 타이베이
      };

      if (centers[countryCode]) {
        map.setCenter(centers[countryCode]);
        map.setZoom(10);
      } else {
        console.warn('No center defined for the selected country.');
      }
    };

    onMounted(async () => {
      try {
        const googleMaps = await loadGoogleMapsApi(apiKey);

        // 지도 생성
        map = new googleMaps.Map(mapContainer.value, {
          center: {lat: 37.5665, lng: 126.9780}, // 서울의 위도와 경도
          zoom: 10,
        });

        // Places API 로드
        const Places = googleMaps.places;
        if (!Places) {
          throw new Error('Places library is not available.');
        }

        const input = document.getElementById('place');
        if (!input) {
          console.error('Input element not found');
          return;
        }

        autocomplete = new googleMaps.places.Autocomplete(input, {
          types: ['establishment'],
          componentRestrictions: {country: 'kr'},
          fields: ['address_components', 'geometry', 'icon', 'name'],
          strictBounds: false,
        });

        autocomplete.addListener('place_changed', () => {
          const place = autocomplete.getPlace();
          if (place.geometry) {
            const location = place.geometry.location;
            const lat = location.lat();
            const lng = location.lng();

            map.setCenter(location);
            map.setZoom(15);

            if (marker) {
              marker.setMap(null);
            }

            marker = new googleMaps.Marker({
              position: location,
              map: map,
              title: place.name,
            });

            placeInfo.value = {
              name: place.name,
              lat: lat.toFixed(6),
              lng: lng.toFixed(6),
            };
          } else {
            placeInfo.value = null;
          }
        });

        alert(props.projectId)
        // Fetch project details and update map and autocomplete
        const response = await axios.get(`http://localhost:8088/api/v1/project/${props.projectId}/detail`);
        if (response.data.result && response.data.result.projectStates && response.data.result.projectStates[0]) {
          const fetchedCountryName = response.data.result.projectStates[0].country;
          alert(fetchedCountryName);
          const countryCode = getCountryCode(fetchedCountryName);

          // Update autocomplete options
          autocomplete.setOptions({
            componentRestrictions: {country: countryCode},
          });

          // Initialize map based on the country code
          initializeMap(countryCode);
        } else {
          console.error('Unexpected response structure:', response.data);
        }
      } catch (error) {
        console.error('Error loading Google Maps API or fetching project details:', error);
      }
    });

    const savePlace = () => {
      if (placeInfo.value) {
        emit('place-selected', placeInfo.value);
      }
    };

    return {mapContainer, placeInfo, savePlace};
  }
};
</script>

<style scoped>
.container {
  display: flex;
  align-items: flex-start;
}

.map-container {
  width: 70%;
  height: 500px;
}

.search-container {
  width: 30%;
  padding: 10px;
}

.place-info {
  margin-top: 20px;
}

input {
  margin-bottom: 10px;
  width: 100%;
  padding: 5px;
}
</style>
