<template>
  <button class="delete-btn" @click="goToRemove">
    Delete
  </button>
</template>

<script setup>
function goToRemove() {
  window.open("http://homeassistant.local:8123/config/integrations/integration/zha", "_blank");
}
</script>

<style>
.delete-btn {
  padding: 0.5rem 1rem;
  background-color: #9b9b9b;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
.delete-btn:hover {
  background-color: #f1130c;
}
</style>