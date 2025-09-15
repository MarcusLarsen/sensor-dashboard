<script setup>
import { ref, computed, watch, onMounted } from "vue"

const props = defineProps({
  entities: { type: Array, default: () => [] }
})

// Load saved positions from localStorage
const saved = localStorage.getItem("sensorPositions")
const sensorPositions = ref(saved ? JSON.parse(saved) : {})

// Dragging state
let dragging = null
let offsetX = 0
let offsetY = 0
let containerRect = null

function startDrag(event, entity) {
  dragging = entity.entity_id

  const dot = event.target
  const rect = dot.getBoundingClientRect()
  containerRect = dot.parentElement.getBoundingClientRect()

  // Center mouse on dot
  offsetX = event.clientX - rect.left - rect.width / 2
  offsetY = event.clientY - rect.top - rect.height / 2

  document.addEventListener("mousemove", onDrag)
  document.addEventListener("mouseup", stopDrag)
}

function onDrag(event) {
  if (!dragging || !containerRect) return

  // Relative to container (floorplan)
  const newTop = event.clientY - containerRect.top - offsetY
  const newLeft = event.clientX - containerRect.left - offsetX

  sensorPositions.value[dragging] = {
    top: `${newTop}px`,
    left: `${newLeft}px`
  }

  // Save positions to localStorage
  localStorage.setItem("sensorPositions", JSON.stringify(sensorPositions.value))
}

function stopDrag() {
  dragging = null
  document.removeEventListener("mousemove", onDrag)
  document.removeEventListener("mouseup", stopDrag)
}

const positionedEntities = computed(() =>
  props.entities.map(e => {
    const pos = sensorPositions.value[e.entity_id] || { top: "50px", left: "50px" }
    return { ...e, pos }
  })
)

watch(
  () => props.entities,
  val => console.log("Mapitem entities:", val),
  { deep: true, immediate: true }
)

onMounted(() => {
  console.log("Loaded positions:", sensorPositions.value)
})
</script>

<template>
  <div class="map-container">
    <img src="@/assets/Sko.png" alt="Plan" class="floorplan" />

    <div
      v-for="entity in positionedEntities"
      :key="entity.entity_id"
      class="dot"
      :class="entity.state === 'on' ? 'open' : 'closed'"
      :style="{ top: entity.pos.top, left: entity.pos.left }"
      :title="entity.attributes?.friendly_name || entity.entity_id"
      @mousedown="startDrag($event, entity)"
    ></div>
  </div>
</template>

<style>
.map-container {
  position: relative;
  width: 100%;
  height: 600px;
  overflow: hidden;
}

.floorplan {
  position: absolute;
  top: 0;
  left: 0;
  max-width: 100%;
  max-height: 100%;
}

.dot {
  position: absolute;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  border: 2px solid white;
  cursor: grab;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
}

.dot.open {
  background-color: #dc2626;
}

.dot.closed {
  background-color: #16a34a;
}
</style>