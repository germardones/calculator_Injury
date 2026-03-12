<script setup>
import { computed } from 'vue';

const props = defineProps({
  economicLosses: { type: Number, required: true },
  painAndSuffering: { type: Number, required: true }
});

// SVG configuration
const size = 300;
const cx = size / 2;
const cy = size / 2;
const radius = 100;
const strokeWidth = 50;

// Calculate dash array properties for the SVG circle
const calculateDashArray = (value, total) => {
  if (total === 0) return '0 1000';
  const circumference = 2 * Math.PI * radius;
  const percentage = value / total;
  const dash = percentage * circumference;
  return `${dash} ${circumference}`;
};

const chartData = computed(() => {
  const total = props.economicLosses + props.painAndSuffering;
  
  if (total === 0) {
    return [
      { color: '#333', dashArray: calculateDashArray(1, 1), dashOffset: 0, label: 'No Data', value: 0, percent: 0 }
    ];
  }

  const economicPercent = Math.round((props.economicLosses / total) * 100);
  const painPercent = 100 - economicPercent;

  const ecoDashArray = calculateDashArray(props.economicLosses, total);
  const painDashArray = calculateDashArray(props.painAndSuffering, total);
  
  // Calculate offset for the second segment
  const circumference = 2 * Math.PI * radius;
  const ecoDashOffset = 0;
  const painDashOffset = - (props.economicLosses / total) * circumference;

  return [
    { 
      color: '#e74c3c', // Red/Orange for Pain 
      dashArray: painDashArray, 
      dashOffset: painDashOffset, 
      label: 'Pain & Suffering', 
      value: props.painAndSuffering, 
      percent: painPercent 
    },
    { 
      color: '#3498db', // Blue for Economic
      dashArray: ecoDashArray, 
      dashOffset: ecoDashOffset, 
      label: 'Economic Losses', 
      value: props.economicLosses, 
      percent: economicPercent 
    }
  ];
});

const formatCurrency = (val) => {
  return parseFloat(val || 0).toLocaleString('en-US', {
    style: 'currency',
    currency: 'USD',
    maximumFractionDigits: 0
  });
};
</script>

<template>
  <div class="calculator-chart fade-up">
    <div class="chart-container">
      <svg :width="size" :height="size" viewBox="0 0 300 300" class="donut-chart">
        <!-- Background circle -->
        <circle 
          :cx="cx" :cy="cy" :r="radius" 
          fill="transparent" 
          stroke="#1a1a1a" 
          :stroke-width="strokeWidth" 
        />
        
        <!-- Data segments -->
        <circle 
          v-for="(segment, index) in chartData" 
          :key="index"
          :cx="cx" :cy="cy" :r="radius" 
          fill="transparent" 
          :stroke="segment.color" 
          :stroke-width="strokeWidth" 
          :stroke-dasharray="segment.dashArray" 
          :stroke-dashoffset="segment.dashOffset"
          stroke-linecap="butt"
          transform="rotate(-90 150 150)"
          class="chart-segment"
        />
        
        <!-- Center Text -->
        <text :x="cx" :y="cy - 10" text-anchor="middle" fill="#888" font-size="12" font-weight="600" letter-spacing="1">TOTAL VALUE</text>
        <text :x="cx" :y="cy + 15" text-anchor="middle" fill="#fff" font-size="20" font-weight="800">
          {{ formatCurrency(props.economicLosses + props.painAndSuffering) }}
        </text>
      </svg>
    </div>

    <div class="chart-legend">
      <div v-for="(segment, index) in chartData" :key="index" class="legend-item" v-show="segment.value > 0">
        <div class="legend-color" :style="{ backgroundColor: segment.color }"></div>
        <div class="legend-text">
          <span class="legend-label">{{ segment.label }}</span>
          <span class="legend-value">{{ segment.percent }}%</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.calculator-chart {
  background: var(--card-bg);
  border: 1px solid var(--border-color);
  border-radius: 12px;
  padding: 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.chart-container {
  position: relative;
  width: 300px;
  height: 300px;
  margin-bottom: 2rem;
}

.chart-segment {
  transition: stroke-dasharray 0.6s ease-out, stroke-dashoffset 0.6s ease-out;
}

.chart-segment:hover {
  opacity: 0.8;
  cursor: pointer;
}

.chart-legend {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  width: 100%;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  background: rgba(255,255,255,0.03);
  padding: 1rem;
  border-radius: 8px;
}

.legend-color {
  width: 16px;
  height: 16px;
  border-radius: 50%;
}

.legend-text {
  display: flex;
  justify-content: space-between;
  width: 100%;
  align-items: center;
}

.legend-label {
  color: var(--text-secondary);
  font-size: 0.9rem;
  font-weight: 500;
}

.legend-value {
  color: white;
  font-weight: 700;
  font-size: 1rem;
}
</style>
