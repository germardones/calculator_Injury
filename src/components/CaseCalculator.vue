<script setup>
import { ref, computed, watch } from 'vue';
import CalculatorInputs from './CalculatorInputs.vue';
import CalculatorResults from './CalculatorResults.vue';
import SocialProof from './SocialProof.vue';
import '../calculator.css';

const inputs = ref({
  medicalExpenses: 0,
  lostIncome: 0,
  propertyDamage: 0,
  outOfPocket: 0,
  multiplier: 1.5,
  fault: 0,
  policyLimit: null
});

const results = computed(() => {
  const economicLosses = 
    Number(inputs.value.medicalExpenses) + 
    Number(inputs.value.lostIncome) + 
    Number(inputs.value.propertyDamage) + 
    Number(inputs.value.outOfPocket);

  const painAndSuffering = economicLosses * Number(inputs.value.multiplier);
  const totalBeforeAdjustment = economicLosses + painAndSuffering;
  
  let finalSettlement = totalBeforeAdjustment;

  // Apply Fault (Georgia Rule)
  const faultPercent = Number(inputs.value.fault);
  const faultDeduction = totalBeforeAdjustment * (faultPercent / 100);
  
  if (faultPercent >= 50) {
    finalSettlement = 0;
  } else {
    finalSettlement -= faultDeduction;
  }

  // Apply Policy Limit
  let isLimited = false;
  if (inputs.value.policyLimit && finalSettlement > inputs.value.policyLimit) {
    finalSettlement = Number(inputs.value.policyLimit);
    isLimited = true;
  }

  return {
    economicLosses,
    painAndSuffering,
    totalBeforeAdjustment,
    faultDeduction,
    finalSettlement,
    fault: faultPercent,
    isLimited
  };
});
</script>

<template>
  <div class="calculator-wrapper">
    <div class="container">
      <header class="calculator-header text-center fade-up" style="margin-bottom: 5rem;">
        <h1 style="font-size: 3.5rem; line-height: 1;">ESTIMATE YOUR<br>CASE VALUE</h1>
        <p style="color: var(--text-secondary); margin-top: 1.5rem; font-weight: 500;">
          Personal Injury Calculator based on Georgia legal standards.
        </p>
      </header>

      <div class="calculator-grid">
        <!-- Lado Izquierdo: Entradas -->
        <CalculatorInputs v-model="inputs" />

        <!-- Lado Derecho: Resultados -->
        <CalculatorResults :results="results" />
      </div>

      <SocialProof />
    </div>
  </div>
</template>

<style scoped>
.text-center { text-align: center; }
</style>
