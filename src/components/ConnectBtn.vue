<template>
  <button class="connect-btn" @click="goToPairing">
    Connect New Device
  </button>
</template>

<script setup>
function goToPairing() {
  window.open("http://homeassistant.local:8123/config/zha/add", "_blank");
}
</script>

<style>
.connect-btn {
  padding: 0.5rem 1rem;
  background-color: #2563eb;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
.connect-btn:hover {
  background-color: #1e40af;
}
</style>