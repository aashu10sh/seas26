<script setup lang="ts">
import { ref } from 'vue'

const isRunning = ref(false)
const showSpans = ref(false)
const currentSpanIndex = ref(-1)

const spans = [
  { id: 1, name: 'GET /api/checkout', service: 'api-gateway', start: 0, duration: 1200, color: '#60a5fa', depth: 0 },
  { id: 2, name: 'AUTH /verify', service: 'auth-service', start: 50, duration: 100, color: '#a78bfa', depth: 1 },
  { id: 3, name: 'POST /payments/charge', service: 'payment-service', start: 150, duration: 900, color: '#f472b6', depth: 1 },
  { id: 4, name: 'POST /stripe/v1/charges', service: 'stripe-api', start: 200, duration: 800, color: '#94a3b8', depth: 2 },
  { id: 5, name: 'POST /inventory/reserve', service: 'inventory-service', start: 1050, duration: 100, color: '#4ade80', depth: 1 },
]

const maxDuration = 1200

async function runTrace() {
  if (isRunning.value) return
  isRunning.value = true
  showSpans.value = false
  currentSpanIndex.value = -1

  await new Promise(r => setTimeout(r, 200))
  showSpans.value = true
  
  for (let i = 0; i < spans.length; i++) {
    currentSpanIndex.value = i
    await new Promise(r => setTimeout(r, 400))
  }
  
  isRunning.value = false
}
</script>

<template>
  <div class="trace-waterfall">
    <div class="controls">
      <button class="btn btn-trace" @click="runTrace" :disabled="isRunning">
        {{ isRunning ? '⏳ Tracing Request...' : '🔍 Trace Request: GET /api/checkout' }}
      </button>
    </div>

    <div class="waterfall-container">
      <div class="timeline-header">
        <span class="ms-marker" style="left: 0%">0ms</span>
        <span class="ms-marker" style="left: 25%">300ms</span>
        <span class="ms-marker" style="left: 50%">600ms</span>
        <span class="ms-marker" style="left: 75%">900ms</span>
        <span class="ms-marker" style="left: 100%">1200ms</span>
      </div>

      <div class="spans" v-if="showSpans">
        <div 
          v-for="(span, index) in spans" 
          :key="span.id" 
          class="span-row"
          :style="{ opacity: index <= currentSpanIndex ? 1 : 0 }"
        >
          <div class="span-info" :style="{ paddingLeft: `${span.depth * 1.5}rem` }">
            <span class="service-badge" :style="{ backgroundColor: `${span.color}22`, color: span.color, borderColor: `${span.color}55` }">
              {{ span.service }}
            </span>
            <span class="span-name">{{ span.name }}</span>
          </div>
          <div class="span-bar-container">
            <div 
              class="span-bar" 
              :style="{ 
                left: `${(span.start / maxDuration) * 100}%`, 
                width: `${(span.duration / maxDuration) * 100}%`,
                backgroundColor: span.color
              }"
            >
              <span class="duration-text">{{ span.duration }}ms</span>
            </div>
          </div>
        </div>
      </div>
      
      <div v-else class="empty-state">
        Click to initiate trace visualization.
      </div>
    </div>
  </div>
</template>

<style scoped>
.trace-waterfall {
  font-family: 'Inter', 'Segoe UI', sans-serif;
  padding: 1rem;
  max-width: 750px;
  margin: 0 auto;
}

.controls {
  display: flex;
  justify-content: center;
  margin-bottom: 1.5rem;
}

.btn {
  padding: 0.5rem 1rem;
  border-radius: 6px;
  border: 1px solid rgba(255,255,255,0.15);
  background: rgba(255,255,255,0.08);
  color: #e2e8f0;
  cursor: pointer;
  font-size: 0.85rem;
  font-weight: 600;
  transition: all 0.2s;
}
.btn:hover { background: rgba(255,255,255,0.15); }
.btn:disabled { opacity: 0.6; cursor: not-allowed; }
.btn-trace { border-color: #60a5fa; color: #60a5fa; }

.waterfall-container {
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 8px;
  padding: 1rem;
}

.timeline-header {
  position: relative;
  height: 20px;
  border-bottom: 1px solid rgba(255,255,255,0.1);
  margin-bottom: 0.5rem;
  margin-left: 35%; /* align with the bar area */
}

.ms-marker {
  position: absolute;
  font-size: 0.6rem;
  color: #64748b;
  transform: translateX(-50%);
  bottom: 4px;
}

.spans {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.span-row {
  display: flex;
  align-items: center;
  transition: opacity 0.3s ease;
}

.span-info {
  width: 35%;
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
  box-sizing: border-box;
  padding-right: 1rem;
}

.service-badge {
  font-size: 0.6rem;
  padding: 0.1rem 0.4rem;
  border-radius: 4px;
  border: 1px solid;
  font-weight: 700;
  text-transform: uppercase;
  width: fit-content;
}

.span-name {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.7rem;
  color: #cbd5e1;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.span-bar-container {
  width: 65%;
  position: relative;
  height: 24px;
  border-left: 1px solid rgba(255,255,255,0.1);
}

.span-bar {
  position: absolute;
  height: 16px;
  top: 4px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

.duration-text {
  font-size: 0.6rem;
  color: #fff;
  font-weight: 600;
  margin-left: 0.4rem;
  text-shadow: 0 1px 2px rgba(0,0,0,0.8);
  white-space: nowrap;
}

.empty-state {
  text-align: center;
  padding: 2rem;
  color: #64748b;
  font-size: 0.85rem;
  font-style: italic;
}
</style>
