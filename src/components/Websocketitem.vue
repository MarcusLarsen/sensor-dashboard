<template>
  <div class="websocket-container">
    <div v-if="!connected" class="status-connecting">Connecting...</div>

    <div v-else>
      <!-- Counters -->
      <div class="counters">
        <div class="counter-item">
          <span class="counter-label">Total:</span>
          <span class="counter-value">{{ counters.total }}</span>
        </div>

        <div class="counter-item">
          <span class="status-dot red"></span>
          <span class="counter-label">Open:</span>
          <span class="counter-value">{{ counters.open }}</span>
        </div>

        <div class="counter-item">
          <span class="status-dot green"></span>
          <span class="counter-label">Closed:</span>
          <span class="counter-value">{{ counters.closed }}</span>
        </div>
      </div>

      <!-- Entity list -->
      <div class="entity-list">
        <div
          v-for="entity in sortedEntities"
          :key="entity.entity_id"
          class="entity-card"
          :ref="el => setEntityRef(entity.entity_id, el)"
          :class="[
            entity.state === 'on' ? 'open' : 'closed',
            entity.entity_id === props.selectedEntity ? 'highlight' : ''
          ]"
        >
          <!-- clicking this area selects/toggles the entity -->
          <div class="entity-info" @click="emit('focus-entity', entity.entity_id)">
            <span class="entity-name">{{ entity.attributes?.friendly_name || entity.entity_id }}</span>
            <span class="entity-state">{{ entity.state === 'on' ? 'Open ❌' : 'Closed ✅' }}</span>
          </div>

          <!-- delete wrapper stops propagation so clicking delete doesn't select -->
          <div class="delete-wrapper" @click.stop>
            <DeleteBtn />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch } from "vue"
import DeleteBtn from "./DeleteBtn.vue"

const token = import.meta.env.VITE_HOME_ASSISTANT_TOKEN
const baseUrl = import.meta.env.VITE_HOME_ASSISTANT_URL

const connected = ref(false)

const props = defineProps({
  entities: { type: Array, default: () => [] },
  selectedEntity: { type: String, default: null }
})

// emitted events
const emit = defineEmits(["update:entities", "focus-entity"])

// sort: "on" first, then by friendly_name/entity_id
const sortedEntities = computed(() =>
  [...props.entities].sort((a, b) => {
    if (a.state === "on" && b.state !== "on") return -1
    if (a.state !== "on" && b.state === "on") return 1
    return (a.attributes?.friendly_name || a.entity_id)
      .localeCompare(b.attributes?.friendly_name || b.entity_id)
  })
)

// counters
const counters = computed(() => {
  const total = props.entities.length
  const open = props.entities.filter(e => e.state === "on").length
  const closed = props.entities.filter(e => e.state === "off").length
  return { total, open, closed }
})

// refs for scrolling into view
const entityRefs = ref({})
function setEntityRef(id, el) {
  if (el) entityRefs.value[id] = el
  else delete entityRefs.value[id]
}

// when selection changes, scroll selected item into view
watch(
  () => props.selectedEntity,
  (newVal) => {
    if (newVal && entityRefs.value[newVal]) {
      // ensure the container is scrolled so the item is visible
      entityRefs.value[newVal].scrollIntoView({
        behavior: "smooth",
        block: "nearest"
      })
    }
  }
)

onMounted(() => {
  const ws = new WebSocket(baseUrl)

  ws.onopen = () => console.log("Connected to Home Assistant WS")

  ws.onmessage = (event) => {
    const msg = JSON.parse(event.data)

    if (msg.type === "auth_required") {
      ws.send(JSON.stringify({
        type: "auth",
        access_token: token
      }))
    }

    if (msg.type === "auth_ok") {
      connected.value = true
      ws.send(JSON.stringify({ id: 2, type: "get_states" }))
      ws.send(JSON.stringify({ id: 3, type: "subscribe_events", event_type: "state_changed" }))
    }

    if (msg.type === "event" && msg.event?.data?.new_state) {
      const entity = msg.event.data.new_state
      if (entity.entity_id.startsWith("binary_sensor.")) {
        const updatedEntities = [...props.entities]
        const index = updatedEntities.findIndex(e => e.entity_id === entity.entity_id)
        if (index >= 0) updatedEntities[index] = { ...entity }
        else updatedEntities.push({ ...entity })

        const withPos = updatedEntities.map((e, i) => ({
          ...e,
          pos: e.pos ?? { top: `${50 + i * 50}px`, left: "50px" }
        }))

        emit("update:entities", withPos)
      }
    }

    if (msg.type === "result" && msg.id === 2 && Array.isArray(msg.result)) {
      const filtered = msg.result
        .filter(e => e.entity_id.startsWith("binary_sensor.") && e.entity_id !== "binary_sensor.rpi_power_status")
        .map(e => ({ ...e }))

      const withPos = filtered.map((e, i) => ({
        ...e,
        pos: { top: `${50 + i * 50}px`, left: "50px" }
      }))

      emit("update:entities", withPos)
    }
  }
})
</script>

<style>
.websocket-container {
  padding: 24px;
  background-color: #f3f4f6;
  min-height: 5vh;
}

.status-connecting {
  color: #dc2626;
  font-weight: bold;
}

.counters {
  display: flex;
  gap: 1.25rem;
  align-items: center;
  justify-content: flex-end;
  margin-bottom: 1rem;
  padding: 0.5rem 0.75rem;
  background: #f8fafc;
  border-radius: 8px;
}

.counter-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-weight: 600;
  color: #111827;
}

.status-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  display: inline-block;
  flex: 0 0 12px;
  box-shadow: 0 1px 2px rgba(0,0,0,0.08);
}

.status-dot.red {
  background-color: #dc2626;
}

.status-dot.green {
  background-color: #16a34a;
}

/* Scrollable list */
.entity-list {
  max-height: 300px;
  overflow-y: auto;
  padding-right: 6px;
}

/* Card */
.entity-card {
  padding: 12px;
  border-radius: 8px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.08);
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  margin-bottom: 8px;
  transition: background 0.15s, border 0.15s, box-shadow 0.15s;
}

/* Clickable info area */
.entity-info {
  flex-grow: 1;
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: pointer;
  user-select: none;
}

/* Hover state */
.entity-card:hover {
  opacity: 0.98;
}

/* States */
.entity-card.open {
  background-color: #fecaca;
}

.entity-card.closed {
  background-color: #bbf7d0;
}

/* Highlighted selection */
.entity-card.highlight {
  border: 2px solid #facc15;
  box-shadow: 0 0 8px rgba(250, 204, 21, 0.85);
}

/* Delete button wrapper (keeps DeleteBtn aligned) */
.delete-wrapper {
  display: flex;
  align-items: center;
  margin-left: 8px;
}

/* Name & state */
.entity-name {
  font-weight: 500;
}

.entity-state {
  margin-left: 8px;
  opacity: 0.95;
}
</style>