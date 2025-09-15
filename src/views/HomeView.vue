<template>
  <main class="main-container">
    <!-- Header -->
    <div class="header">
      <h1>Home Assistant Dashboard</h1>
      <ConnectBtn />
    </div>

    <!-- Sensor list -->
    <Websocketitem v-model:entities="entities" />

    <!-- Map -->
    <div class="map-wrapper">
      <Mapitem :entities="entities" />
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue"
import Websocketitem from "../components/Websocketitem.vue"
import ConnectBtn from "../components/ConnectBtn.vue"
import Mapitem from "../components/Mapitem.vue"


const entities = ref([])
</script>

<style>
.main-container {
  padding: 1.5rem;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.map-wrapper {
  margin-top: 2rem;
  width: 100%;
  height: 600px;
  position: relative;
  border: 1px solid #ccc;
}
</style>