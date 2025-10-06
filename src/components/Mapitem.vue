<script setup>
import { ref, computed, watch, onMounted, nextTick } from "vue"

const props = defineProps({
  entities: { type: Array, default: () => [] },
  selectedEntity: { type: String, default: null }
})

const emit = defineEmits(["focus-entity"])

// Load saved positions
const saved = localStorage.getItem("sensorPositions")
const sensorPositions = ref(saved ? JSON.parse(saved) : {})

// Dragging state
let dragging = null
let offsetX = 0
let offsetY = 0
let container = null

function startDrag(event, entity) {
  dragging = entity.entity_id

  const dot = event.target
  const rect = dot.getBoundingClientRect()
  container = dot.parentElement.getBoundingClientRect()

  offsetX = event.clientX - rect.left - rect.width / 2
  offsetY = event.clientY - rect.top - rect.height / 2

  document.addEventListener("mousemove", onDrag)
  document.addEventListener("mouseup", stopDrag)
}

function onDrag(event) {
  if (!dragging || !container) return

  const newTop = event.clientY - container.top - offsetY
  const newLeft = event.clientX - container.left - offsetX

  sensorPositions.value[dragging] = {
    top: `${newTop}px`,
    left: `${newLeft}px`
  }

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

// Auto-scroll to center highlighted dot
watch(
  () => props.selectedEntity,
  async (newId) => {
    if (!newId) return
    await nextTick() // wait for DOM update
    const dot = document.querySelector(".dot.highlight")
    const containerEl = document.querySelector(".map-container")
    if (dot && containerEl) {
      const scrollLeft = dot.offsetLeft - containerEl.clientWidth / 2 + dot.clientWidth / 2
      const scrollTop = dot.offsetTop - containerEl.clientHeight / 2 + dot.clientHeight / 2

      containerEl.scrollTo({
        left: scrollLeft,
        top: scrollTop,
        behavior: "smooth"
      })
    }
  }
)

watch(
  () => props.entities,
  val => console.log("Mapitem entities:", val),
  { deep: true, immediate: true }
)

onMounted(() => {
  console.log("Loaded positions:", sensorPositions.value)
})

// Emit click on dot
function handleDotClick(entityId) {
  emit("focus-entity", entityId)
}
</script>

<template>
  <div class="map-container">
    <img src="@/assets/Sko.png" alt="Plan" class="floorplan" />

    <div
      v-for="entity in positionedEntities"
      :key="entity.entity_id"
      class="dot"
      :class="[
        entity.state === 'on' ? 'open' : 'closed',
        entity.entity_id === props.selectedEntity ? 'highlight' : ''
      ]"
      :style="{ top: entity.pos.top, left: entity.pos.left }"
      :title="entity.attributes?.friendly_name || entity.entity_id"
      @mousedown="startDrag($event, entity)"
      @click.stop="handleDotClick(entity.entity_id)"
    ></div>
  </div>
</template>

<style>
.map-container {
  position: relative;
  width: 100%;
  height: 600px;
  overflow: auto;
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

.dot.highlight {
  border: 3px solid yellow;
  box-shadow: 0 0 12px 4px rgba(255, 255, 0, 0.7);
  z-index: 10;
}
</style>