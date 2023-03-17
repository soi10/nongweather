<template>
  <Navbar />
  <div class="container my-2">
    <!-- <p>latitude: {{ latitude }}</p>
    <p>longitude: {{ longitude }}</p>
    <p v-if="city">City: {{ city }}</p> -->

    <div class="mb-3">
      <div class="row">
        <div class="col-2">City :</div>
        <div class="col-6">
          <input
            type="text"
            class="form-control"
            id="exampleFormControlInput1"
            placeholder="City"
            v-model="textSearch"
          />
        </div>
        <div class="col-2">
          <button type="button" class="btn btn-primary" @click="searchCity">
            Search
          </button>
        </div>
      </div>
    </div>
    <div class="my-2"></div>
    <!-- {{ states }} -->
    <div v-for="(state, index) in states" :key="index">
      <div v-for="(weather, index) in state.weather" :key="index">
        <div class="card border-info my-3">
          <div class="card-header">
            {{ state.name }} as {{ Date(state.dt) }}
          </div>
          <div class="card-body">
            <p class="card-text">
              อุณหภูมิ : {{ (state.main.temp - 273.15).toFixed(2) }} &#8451;
            </p>
            <div class="row">
              <div class="col">
                <p class="card-text">{{ weather.main }}</p>
              </div>
              <div class="col text-end">
                <img
                  :src="`https://openweathermap.org/img/wn/${weather.icon}@2x.png`"
                  class="img-thumbnail"
                />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import Navbar from "./components/Navbar.vue";
import { ref, reactive } from "vue";
import axios from "axios";
import Swal from "sweetalert2";

const latitude = ref("");
const longitude = ref("");
const city = ref("");
const textSearch = ref("");

const states = reactive([]);

const showPosition = async (position) => {
  latitude.value = position.coords.latitude;
  longitude.value = position.coords.longitude;

  try {
    const response = await axios.get(
      `https://api.opencagedata.com/geocode/v1/json?key=7e84f538f24b4d249ed94e8a18e32a84&q=${latitude.value}%2C${longitude.value}`
    );

    if (!response.data.results[0].components.city) {
      city.value = "Amnat Charoen";
      textSearch.value = "Amnat Charoen";
    } else {
      city.value = response.data.results[0].components.city;
      textSearch.value = response.data.results[0].components.city;
    }
    try {
      const response2 = await axios.get(
        `https://api.openweathermap.org/data/2.5/weather?q=${city.value}&appid=e7dd9917dc19f4cf7878f4cedf47e862`
      );
      //console.log(response2);
      states.push(response2.data);
    } catch (error) {
      console.log(error);
    }
  } catch (error) {
    console.log(error);
  }
};

const getLocation = () => {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(showPosition);
  } else {
    location.value = "Geolocation is not supported by this browser.";
  }
};

getLocation();

const searchCity = async () => {
  if (textSearch.value === "") {
    Swal.fire({
      title: "เกิดข้อผิดผลาด!",
      text: "กรอกชื่อ City",
      icon: "error",
    });
  } else {
    try {
      const response2 = await axios.get(
        `https://api.openweathermap.org/data/2.5/weather?q=${textSearch.value}&appid=e7dd9917dc19f4cf7878f4cedf47e862`
      );
      //console.log(response2);
      states.length = 0;
      states.push(response2.data);
    } catch (error) {
      console.log(error);
      Swal.fire({
        title: "เกิดข้อผิดผลาด!",
        text: "ไม่พบข้อมูล หรือ ชื่อไม่ถูกต้อง",
        icon: "error",
      });
    }
  }
};
</script>

<style scoped></style>
