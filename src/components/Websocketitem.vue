<template>
  <div class="websocket-container">
    <div v-if="!connected" class="status-connecting">Connecting...</div>

    <div v-else class="entity-grid">
      <div 
        v-for="entity in entities" 
        :key="entity.entity_id"
        class="entity-card"
        :class="entity.state === 'on' ? 'open' : 'closed'"
      >
        <span class="entity-name">{{ entity.attributes.friendly_name }}</span>
        <span class="entity-state">{{ entity.state === 'on' ? 'Open ❌' : 'Closed ✅' }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue"

const connected = ref(false)

// ✅ receive entities from parent via v-model
const props = defineProps({
  entities: {
    type: Array,
    default: () => []
  }
})
const emit = defineEmits(["update:entities"])

onMounted(() => {
  const ws = new WebSocket("ws://homeassistant.local:8123/api/websocket")

  ws.onopen = () => console.log("Connected to Home Assistant WS")

  ws.onmessage = (event) => {
    const msg = JSON.parse(event.data)
    console.log("WS Message:", msg)

    if (msg.type === "auth_required") {
      ws.send(JSON.stringify({
        type: "auth",
        access_token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiI5OTAxMzI4NGVhOGM0YTQxYWExMTkyMjVmNmEwZjJmMyIsImlhdCI6MTc1NTg0NzA2MCwiZXhwIjoyMDcxMjA3MDYwfQ.L0MnC_rLNavCzGpa-vmTAHCpdhoVoYxdo7udHFyzLPw"
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

        // add positions
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
  background-color: #f3f4f6; /* gray-100 */
  min-height: 5vh;
}

.status-connecting {
  color: #dc2626; /* red */
  font-weight: bold;
}

.entity-grid {
  display: grid;
  gap: 16px; /* grid gap-4 */
}

.entity-card {
  padding: 16px; /* p-4 */
  border-radius: 8px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.1);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.entity-card.open {
  background-color: #fecaca; /* red-200 */
}

.entity-card.closed {
  background-color: #bbf7d0; /* green-200 */
}

.entity-name {
  font-weight: 500;
}

.entity-state {
  margin-left: 8px;
}
</style>