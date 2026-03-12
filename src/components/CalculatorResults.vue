<script setup>
import { computed } from 'vue';

const props = defineProps({
  results: Object
});

const formatCurrency = (val) => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    maximumFractionDigits: 0
  }).format(val || 0);
};

// Calculate percentages for the breakdown bar
const economicPercent = computed(() => {
  if (props.results.totalBeforeAdjustment <= 0) return 0;
  return (props.results.economicLosses / props.results.totalBeforeAdjustment) * 100;
});

const painPercent = computed(() => {
  if (props.results.totalBeforeAdjustment <= 0) return 0;
  return (props.results.painAndSuffering / props.results.totalBeforeAdjustment) * 100;
});
</script>

<template>
  <div class="results-card fade-up">
    <div class="result-header">
      <p class="result-label">ESTIMATED CASE VALUE</p>
      <h1 class="result-value">{{ formatCurrency(results.finalSettlement) }}</h1>
      <button class="btn-primary" style="width: 100%; margin-top: 1.5rem;">
        TALK TO A LAWYER NOW
      </button>
    </div>

    <div class="breakdown-section">
      <h3 class="section-title" style="margin-bottom: 1rem;">ESTIMATED BREAKDOWN</h3>
      
      <!-- Custom Breakdown Bar -->
      <div class="breakdown-bar">
        <div 
          class="bar-segment economic" 
          :style="{ width: economicPercent + '%' }"
          title="Economic Losses"
        ></div>
        <div 
          class="bar-segment pain" 
          :style="{ width: painPercent + '%' }"
          title="Pain and Suffering"
        ></div>
      </div>

      <div class="breakdown-details">
        <div class="breakdown-row">
          <span>Economic Losses</span>
          <span class="white-text">{{ formatCurrency(results.economicLosses) }}</span>
        </div>
        <div class="breakdown-row">
          <span>Pain and Suffering</span>
          <span class="white-text">{{ formatCurrency(results.painAndSuffering) }}</span>
        </div>
        <div class="breakdown-row" v-if="results.fault > 0">
          <span>Fault Deduction ({{ results.fault }}%)</span>
          <span class="red-text">-{{ formatCurrency(results.faultDeduction) }}</span>
        </div>
        <div class="breakdown-row" v-if="results.isLimited">
          <span class="red-text">Policy Limit Adjustment</span>
          <span class="red-text">{{ formatCurrency(results.finalSettlement) }} (Cap)</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.white-text { color: white; }
.red-text { color: #ff4d4d; }

.breakdown-bar {
  height: 8px;
  background: var(--border-color);
  display: flex;
  margin-bottom: 1.5rem;
  overflow: hidden;
}

.bar-segment {
  height: 100%;
  transition: width 0.5s ease;
}

.bar-segment.economic {
  background: white;
}

.bar-segment.pain {
  background: #a0a0a0;
}

.breakdown-row span:last-child {
  font-weight: 700;
}
</style>
