<script setup>
import { computed } from 'vue';
import CalculatorChart from './CalculatorChart.vue';

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
  <div class="results-container">
    <div class="results-card fade-up">
      <div class="result-header">
        <p class="result-label">ESTIMATED CASE VALUE</p>
        <h1 class="result-value">{{ formatCurrency(results.finalSettlement) }}</h1>
        
        <div class="info-alert" style="margin-top: 1rem;">
          This is what your case <strong>could be worth</strong> — but insurance companies won't hand it over easily.
        </div>

        <div class="cta-alert" style="margin-top: 1rem;">
          <h4 style="margin-bottom: 0.5rem; font-weight: 700; font-size: 1.1rem; color: #fff;">Want to maximize your payout?</h4>
          <p style="margin-bottom: 1rem; font-size: 0.9rem; color: #e0e0e0;">Our experienced attorneys fight to make sure you get every dollar you deserve.</p>
          <button class="btn-primary" style="width: 100%;">
            TALK TO A TRUSTED LAWYER NOW 
          </button>
        </div>
      </div>

      <div class="breakdown-section">
        <h3 class="section-title" style="margin-bottom: 1rem;">ESTIMATED BREAKDOWN</h3>
        
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
    
    <!-- Dynamic SVG Donut Chart -->
    <CalculatorChart 
      :economicLosses="results.economicLosses" 
      :painAndSuffering="results.painAndSuffering" 
    />
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

.results-container {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.info-alert {
  background: rgba(255, 255, 255, 0.05);
  border-left: 3px solid #666;
  padding: 1rem;
  border-radius: 4px;
  font-size: 0.9rem;
  color: #ccc;
  line-height: 1.5;
}

.info-alert strong {
  color: white;
}

.cta-alert {
  background: linear-gradient(135deg, rgba(30,150,100,0.1) 0%, rgba(10,50,30,0.1) 100%);
  border: 1px solid rgba(30, 150, 100, 0.3);
  padding: 1.5rem;
  border-radius: 8px;
  text-align: left;
}

/* Custom button color for CTA */
.cta-alert .btn-primary {
  background: white;
  color: black;
  font-size: 0.85rem;
  padding: 0.875rem 1rem;
}
.cta-alert .btn-primary:hover {
  background: #eee;
  transform: translateY(-2px);
}
</style>
