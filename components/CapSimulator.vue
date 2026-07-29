<script setup lang="ts">
import { ref, computed } from 'vue'

const partitionActive = ref(false)
const strategy = ref<'none' | 'consistency' | 'availability'>('none')
const writeValue = ref(42)
const nodeAValue = ref(42)
const nodeBValue = ref(42)
const nodeAStatus = ref<'ok' | 'error' | 'stale'>('ok')
const nodeBStatus = ref<'ok' | 'error' | 'stale'>('ok')
const lastAction = ref('')

function togglePartition() {
  partitionActive.value = !partitionActive.value
  if (!partitionActive.value) {
    // Heal partition — sync nodes
    strategy.value = 'none'
    nodeBValue.value = nodeAValue.value
    nodeAStatus.value = 'ok'
    nodeBStatus.value = 'ok'
    lastAction.value = 'Partition healed. Nodes re-synced.'
  } else {
    strategy.value = 'none'
    lastAction.value = 'Network partition active! Choose a strategy.'
  }
}

function writeToLeader() {
  writeValue.value += 1
  nodeAValue.value = writeValue.value

  if (!partitionActive.value) {
    // No partition — both nodes stay in sync
    nodeBValue.value = writeValue.value
    nodeAStatus.value = 'ok'
    nodeBStatus.value = 'ok'
    lastAction.value = `Write v=${writeValue.value} → replicated to both nodes.`
  } else if (strategy.value === 'consistency') {
    // CP: write fails because we can't confirm replication
    nodeAValue.value = writeValue.value - 1 // rollback
    writeValue.value -= 1
    nodeAStatus.value = 'error'
    nodeBStatus.value = 'error'
    lastAction.value = 'Write REJECTED — cannot guarantee consistency across partition.'
  } else if (strategy.value === 'availability') {
    // AP: write succeeds on Node A, Node B becomes stale
    nodeAStatus.value = 'ok'
    nodeBStatus.value = 'stale'
    lastAction.value = `Write v=${writeValue.value} accepted on Node A. Node B is stale (v=${nodeBValue.value}).`
  } else {
    lastAction.value = 'Pick a strategy first!'
    nodeAValue.value = writeValue.value - 1
    writeValue.value -= 1
  }
}

function chooseStrategy(s: 'consistency' | 'availability') {
  strategy.value = s
  lastAction.value = s === 'consistency'
    ? 'CP mode: writes will be rejected if replication cannot be confirmed.'
    : 'AP mode: writes will be accepted locally, risking stale reads on Node B.'
}

const linkColor = computed(() => {
  if (!partitionActive.value) return '#4ade80'
  return '#ef4444'
})

const linkLabel = computed(() => {
  if (!partitionActive.value) return 'synced'
  return '✕ partitioned'
})

function statusColor(s: string) {
  if (s === 'ok') return '#4ade80'
  if (s === 'error') return '#ef4444'
  return '#fbbf24'
}

function statusLabel(s: string) {
  if (s === 'ok') return '✓ OK'
  if (s === 'error') return '✕ UNAVAILABLE'
  return '⚠ STALE'
}
</script>

<template>
  <div class="cap-sim">
    <div class="controls">
      <button
        :class="['btn', partitionActive ? 'btn-danger' : 'btn-success']"
        @click="togglePartition"
      >
        {{ partitionActive ? '🔌 Heal Partition' : '💥 Trigger Partition' }}
      </button>

      <div v-if="partitionActive" class="strategy-buttons">
        <button
          :class="['btn', 'btn-cp', { active: strategy === 'consistency' }]"
          @click="chooseStrategy('consistency')"
        >
          🔒 Prioritize Consistency (CP)
        </button>
        <button
          :class="['btn', 'btn-ap', { active: strategy === 'availability' }]"
          @click="chooseStrategy('availability')"
        >
          🌐 Prioritize Availability (AP)
        </button>
      </div>

      <button class="btn btn-write" @click="writeToLeader">
        ✏️ Write to Leader
      </button>
    </div>

    <div class="nodes">
      <div class="node">
        <div class="node-header">Node A (Leader)</div>
        <div class="node-value">v = {{ nodeAValue }}</div>
        <div class="node-status" :style="{ color: statusColor(nodeAStatus) }">
          {{ statusLabel(nodeAStatus) }}
        </div>
      </div>

      <div class="link">
        <div class="link-line" :style="{ borderColor: linkColor }">
          <span class="link-label" :style="{ color: linkColor }">{{ linkLabel }}</span>
        </div>
      </div>

      <div class="node">
        <div class="node-header">Node B (Follower)</div>
        <div class="node-value">v = {{ nodeBValue }}</div>
        <div class="node-status" :style="{ color: statusColor(nodeBStatus) }">
          {{ statusLabel(nodeBStatus) }}
        </div>
      </div>
    </div>

    <div class="action-log" v-if="lastAction">
      {{ lastAction }}
    </div>
  </div>
</template>

<style scoped>
.cap-sim {
  font-family: 'Inter', 'Segoe UI', sans-serif;
  padding: 1rem;
  max-width: 600px;
  margin: 0 auto;
}

.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  justify-content: center;
  margin-bottom: 1.5rem;
}

.strategy-buttons {
  display: flex;
  gap: 0.5rem;
  width: 100%;
  justify-content: center;
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
.btn-success { border-color: #4ade80; color: #4ade80; }
.btn-danger { border-color: #ef4444; color: #ef4444; }
.btn-cp { border-color: #60a5fa; color: #60a5fa; }
.btn-ap { border-color: #fbbf24; color: #fbbf24; }
.btn-write { border-color: #a78bfa; color: #a78bfa; }
.btn.active { background: rgba(255,255,255,0.18); font-weight: 700; }

.nodes {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0;
}

.node {
  background: rgba(255,255,255,0.06);
  border: 1px solid rgba(255,255,255,0.12);
  border-radius: 10px;
  padding: 1rem 1.4rem;
  text-align: center;
  min-width: 140px;
}

.node-header {
  font-weight: 700;
  font-size: 0.85rem;
  margin-bottom: 0.5rem;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.node-value {
  font-size: 1.6rem;
  font-weight: 800;
  font-family: 'JetBrains Mono', monospace;
  color: #f1f5f9;
  margin-bottom: 0.4rem;
}

.node-status {
  font-size: 0.75rem;
  font-weight: 600;
}

.link {
  display: flex;
  align-items: center;
  padding: 0 0.5rem;
}

.link-line {
  border-top: 2px dashed;
  width: 60px;
  position: relative;
  display: flex;
  justify-content: center;
}

.link-label {
  position: absolute;
  top: 4px;
  font-size: 0.65rem;
  font-weight: 600;
  white-space: nowrap;
}

.action-log {
  margin-top: 1rem;
  padding: 0.6rem 1rem;
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 6px;
  font-size: 0.75rem;
  color: #94a3b8;
  text-align: center;
}
</style>
