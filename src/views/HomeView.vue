<script setup>
import { ref } from "vue"
import Websocketitem from "../components/Websocketitem.vue"
import ConnectBtn from "../components/ConnectBtn.vue"
import Mapitem from "../components/Mapitem.vue"

const entities = ref([])
const selectedEntity = ref(null)

// toggle selection on click
function handleFocusEntity(entityId) {
  if (selectedEntity.value === entityId) {
    selectedEntity.value = null 
  } else {
    selectedEntity.value = entityId
  }
}
</script>

<template>
  <main class="main-container">
    <!-- Header -->
    <div class="header">
      <h1>Home Assistant Dashboard</h1>
      <ConnectBtn />
    </div>

    <!-- Sensor list -->
    <Websocketitem
      v-model:entities="entities"
      :selectedEntity="selectedEntity" 
      @focus-entity="handleFocusEntity" 
    />

    <!-- Map -->
    <div class="map-wrapper">
      <Mapitem 
        :entities="entities" 
        :selectedEntity="selectedEntity"
        @focus-entity="handleFocusEntity" 
      /> 
    </div>
  </main>
</template>

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