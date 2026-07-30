<script setup lang="ts">
import { ref, computed } from 'vue'

const idempotencyEnabled = ref(false)
const intendedPayments = ref(0)
const actualCharges = ref(0)
const attempts = ref<{ id: number; result: string; idempotencyKey: string | null }[]>([])
const isProcessing = ref(false)
const currentIdempotencyKey = ref('')
let attemptCounter = 0
const seenKeys = new Set<string>()

function generateKey() {
  return 'idem_' + Math.random().toString(36).substring(2, 8)
}

async function sendPayment() {
  if (isProcessing.value) return
  isProcessing.value = true
  intendedPayments.value += 1
  const key = idempotencyEnabled.value ? generateKey() : null
  currentIdempotencyKey.value = key || ''

  // Simulate: first attempt always "times out" but succeeds server-side
  // Then we auto-retry
  for (let retry = 0; retry < 3; retry++) {
    attemptCounter++
    const attemptId = attemptCounter
    const networkFails = retry < 1 // First attempt "times out"

    await new Promise(r => setTimeout(r, 400 + Math.random() * 300))

    if (networkFails) {
      // Server DID process it, but client got a timeout
      if (key && seenKeys.has(key)) {
        // Idempotent: server already processed this key
        attempts.value.unshift({
          id: attemptId,
          result: '⏱️ Timeout (server already processed — deduplicated)',
          idempotencyKey: key
        })
      } else {
        // Server processes it
        if (key) seenKeys.add(key)
        actualCharges.value += 1
        attempts.value.unshift({
          id: attemptId,
          result: '⏱️ Timeout (but server DID charge — client retries...)',
          idempotencyKey: key
        })
      }
      continue
    }

    // Retry succeeds — but does the server deduplicate?
    if (key && seenKeys.has(key)) {
      // Idempotent: server recognizes the key, returns cached result
      attempts.value.unshift({
        id: attemptId,
        result: '✅ 200 OK (deduplicated — no extra charge)',
        idempotencyKey: key
      })
    } else {
      // No idempotency: server processes again = duplicate charge!
      if (key) seenKeys.add(key)
      actualCharges.value += 1
      attempts.value.unshift({
        id: attemptId,
        result: '✅ 200 OK (charged AGAIN — duplicate!)',
        idempotencyKey: key
      })
    }
    break
  }

  if (attempts.value.length > 12) {
    attempts.value = attempts.value.slice(0, 12)
  }
  isProcessing.value = false
}

function reset() {
  intendedPayments.value = 0
  actualCharges.value = 0
  attempts.value = []
  attemptCounter = 0
  seenKeys.clear()
  currentIdempotencyKey.value = ''
}

const chargeColor = computed(() => {
  if (actualCharges.value === 0) return '#94a3b8'
  if (actualCharges.value === intendedPayments.value) return '#4ade80'
  return '#ef4444'
})

const chargeStatus = computed(() => {
  if (actualCharges.value === 0) return ''
  if (actualCharges.value === intendedPayments.value) return '✓ Correct'
  return `⚠ ${actualCharges.value - intendedPayments.value} duplicate(s)!`
})
</script>

<template>
  <div class="idem-sim">
    <div class="controls">
      <label class="toggle-label">
        <span class="toggle-text">Idempotency Key</span>
        <div class="toggle-switch" :class="{ active: idempotencyEnabled }" @click="idempotencyEnabled = !idempotencyEnabled">
          <div class="toggle-knob" />
        </div>
        <span class="toggle-state" :style="{ color: idempotencyEnabled ? '#4ade80' : '#ef4444' }">
          {{ idempotencyEnabled ? 'ON' : 'OFF' }}
        </span>
      </label>

      <div class="action-buttons">
        <button class="btn btn-pay" @click="sendPayment" :disabled="isProcessing">
          {{ isProcessing ? '⏳ Processing...' : '💳 Charge $50' }}
        </button>
        <button class="btn btn-reset" @click="reset">↺ Reset</button>
      </div>
    </div>

    <div class="counters">
      <div class="counter-box">
        <div class="counter-label">Intended Payments</div>
        <div class="counter-value">{{ intendedPayments }}</div>
      </div>
      <div class="counter-box">
        <div class="counter-label">Actual Charges</div>
        <div class="counter-value" :style="{ color: chargeColor }">{{ actualCharges }}</div>
        <div class="counter-status" :style="{ color: chargeColor }">{{ chargeStatus }}</div>
      </div>
    </div>

    <div class="log" v-if="attempts.length">
      <div class="log-header">Request Log</div>
      <div
        v-for="a in attempts"
        :key="a.id"
        class="log-entry"
      >
        <span class="log-id">#{{ a.id }}</span>
        <span class="log-result">{{ a.result }}</span>
        <span v-if="a.idempotencyKey" class="log-key">key={{ a.idempotencyKey }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.idem-sim {
  font-family: 'Inter', 'Segoe UI', sans-serif;
  padding: 1rem;
  max-width: 600px;
  margin: 0 auto;
}

.controls {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  align-items: center;
  margin-bottom: 1.25rem;
}

.toggle-label {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  cursor: pointer;
  user-select: none;
}

.toggle-text {
  font-size: 0.8rem;
  font-weight: 600;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.toggle-switch {
  width: 40px;
  height: 22px;
  border-radius: 11px;
  background: rgba(239, 68, 68, 0.3);
  border: 1px solid rgba(239, 68, 68, 0.5);
  position: relative;
  transition: all 0.25s;
  cursor: pointer;
}

.toggle-switch.active {
  background: rgba(74, 222, 128, 0.3);
  border-color: rgba(74, 222, 128, 0.5);
}

.toggle-knob {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #e2e8f0;
  position: absolute;
  top: 2px;
  left: 2px;
  transition: transform 0.25s;
}

.toggle-switch.active .toggle-knob {
  transform: translateX(18px);
}

.toggle-state {
  font-size: 0.75rem;
  font-weight: 700;
  min-width: 28px;
}

.action-buttons {
  display: flex;
  gap: 0.5rem;
}

.btn {
  padding: 0.4rem 0.9rem;
  border-radius: 6px;
  border: 1px solid rgba(255,255,255,0.15);
  background: rgba(255,255,255,0.08);
  color: #e2e8f0;
  cursor: pointer;
  font-size: 0.8rem;
  font-weight: 500;
  transition: all 0.2s;
}
.btn:hover { background: rgba(255,255,255,0.15); }
.btn:disabled { opacity: 0.5; cursor: not-allowed; }
.btn-pay { border-color: #a78bfa; color: #a78bfa; }
.btn-reset { border-color: #94a3b8; color: #94a3b8; }

.counters {
  display: flex;
  gap: 1rem;
  justify-content: center;
  margin-bottom: 1rem;
}

.counter-box {
  background: rgba(255,255,255,0.06);
  border: 1px solid rgba(255,255,255,0.12);
  border-radius: 10px;
  padding: 0.8rem 1.4rem;
  text-align: center;
  min-width: 140px;
}

.counter-label {
  font-weight: 700;
  font-size: 0.75rem;
  margin-bottom: 0.4rem;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.counter-value {
  font-size: 1.6rem;
  font-weight: 800;
  font-family: 'JetBrains Mono', monospace;
  color: #f1f5f9;
}

.counter-status {
  font-size: 0.7rem;
  font-weight: 600;
  margin-top: 0.2rem;
}

.log {
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 6px;
  padding: 0.6rem;
  max-height: 160px;
  overflow-y: auto;
}

.log-header {
  font-size: 0.65rem;
  font-weight: 700;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  margin-bottom: 0.4rem;
}

.log-entry {
  font-size: 0.7rem;
  color: #94a3b8;
  padding: 0.2rem 0;
  border-bottom: 1px solid rgba(255,255,255,0.04);
  display: flex;
  gap: 0.5rem;
  align-items: baseline;
  flex-wrap: wrap;
}

.log-id {
  font-family: 'JetBrains Mono', monospace;
  color: #64748b;
  font-weight: 600;
  min-width: 24px;
}

.log-result {
  flex: 1;
}

.log-key {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.6rem;
  color: #60a5fa;
  opacity: 0.7;
}
</style>
