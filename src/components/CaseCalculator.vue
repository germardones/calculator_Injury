<script setup>
import { ref, computed, watch } from 'vue';
import CalculatorInputs from './CalculatorInputs.vue';
import CalculatorResults from './CalculatorResults.vue';
import SocialProof from './SocialProof.vue';
import ContactForm from './ContactForm.vue';
import '../calculator.css';

const formatCurrency = (val) => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    maximumFractionDigits: 0
  }).format(val || 0);
};

const isModalOpen = ref(false);
const submissionStatus = ref('idle');
const submissionError = ref('');

const inputs = ref({
  medicalExpenses: '',
  lostIncome: '',
  propertyDamage: '',
  outOfPocket: '',
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

  // Apply Fault (Florida Rule)
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

const handleZapierSubmit = async (formData) => {
  submissionStatus.value = 'loading';
  submissionError.value = '';

  const webhookUrl = import.meta.env.VITE_ZAPIER_WEBHOOK_URL;

  if (!webhookUrl) {
    console.warn('Zapier Webhook URL is missing in .env');
    submissionStatus.value = 'error';
    submissionError.value = 'Configuration Error: Zapier Webhook URL is missing. Please add VITE_ZAPIER_WEBHOOK_URL to your .env file.';
    return;
  }

  const payload = {
    contact: formData,
    calculator: {
      inputs: {
        medicalExpenses: inputs.value.medicalExpenses || 0,
        lostIncome: inputs.value.lostIncome || 0,
        propertyDamage: inputs.value.propertyDamage || 0,
        outOfPocket: inputs.value.outOfPocket || 0,
        multiplier: inputs.value.multiplier,
        fault: inputs.value.fault,
        policyLimit: inputs.value.policyLimit
      },
      results: {
        economicLosses: results.value.economicLosses,
        painAndSuffering: results.value.painAndSuffering,
        totalBeforeAdjustment: results.value.totalBeforeAdjustment,
        faultDeduction: results.value.faultDeduction,
        finalSettlement: results.value.finalSettlement,
        isLimited: results.value.isLimited
      },
      formattedSettlement: formatCurrency(results.value.finalSettlement)
    },
    meta: {
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent,
      pageUrl: window.location.href
    }
  };

  try {
    // Note: We use no-cors to avoid preflight issues with typical Zapier Webhook setups
    // unless the user specifically enables CORS on the Zap side.
    await fetch(webhookUrl, {
      method: 'POST',
      mode: 'no-cors', 
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(payload)
    });

    // Since mode: 'no-cors' makes the response opaque, we assume success if no error is thrown
    submissionStatus.value = 'success';
  } catch (error) {
    console.error('Zapier Submission Error:', error);
    submissionStatus.value = 'error';
    submissionError.value = 'There was a technical problem sending your data. Please try again or contact support.';
  }
};

const resetSubmission = () => {
  submissionStatus.value = 'idle';
  submissionError.value = '';
};

// Reset form when modal opens
watch(isModalOpen, (newVal) => {
  if (newVal) {
    resetSubmission();
  }
});
</script>

<template>
  <div class="calculator-wrapper">
    <div class="container">
      <header class="calculator-header text-center fade-up" style="margin-bottom: 3rem;">
        <h1 class="main-title">ESTIMATE YOUR<br>CASE VALUE</h1>
        <p style="color: var(--text-secondary); margin-top: 1.5rem; font-weight: 500;">
          Personal Injury Calculator based on Florida legal standards.
        </p>
      </header>

      <div class="dashboard-grid">
        <!-- Left Column: Inputs -->
        <div class="inputs-column" style="position: relative; z-index: 20;">
          <CalculatorInputs v-model="inputs" @open-modal="isModalOpen = true" />
        </div>
        
        <!-- Right Column: Results & Features -->
        <div class="results-sidebar" style="position: relative; z-index: 10;">
          <CalculatorResults :results="results" @open-modal="isModalOpen = true" />
          <!-- The chart will be injected here inside CalculatorResults or placed below it -->
        </div>
      </div>

      <SocialProof />
    </div>

    <!-- Sticky Bottom Summary Bar -->
    <div class="sticky-bottom-bar" style="z-index: 99999;">
      <div class="container sticky-bar-content">
        <div class="sticky-text">
          ESTIMATED CASE VALUE <span class="sticky-value">{{ formatCurrency(results.finalSettlement) }}</span>
        </div>
        <button class="btn-primary" @click="isModalOpen = true">
          Click here to maximize your payout
        </button>
      </div>
    </div>

    <!-- Contact Modal Base -->
    <div v-if="isModalOpen" class="modal-overlay" @click="isModalOpen = false">
      <div class="modal-content fade-up" @click.stop>
        <div class="modal-header">
          <h2 style="color: white; font-size: 1.5rem;">Talk to a Lawyer</h2>
          <button class="close-btn" @click="isModalOpen = false">&times;</button>
        </div>
        <div class="modal-body">
          <ContactForm 
            :status="submissionStatus" 
            :error-message="submissionError"
            @submit="handleZapierSubmit"
            @reset-status="resetSubmission"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Specific sticky bar or component overrides if needed */
.main-title {
  font-size: 3.5rem;
  line-height: 1;
}
@media (max-width: 768px) {
  .main-title {
    font-size: 2.25rem;
  }
}
.text-center { text-align: center; }

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(5px);
  -webkit-backdrop-filter: blur(5px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 999999;
  padding: 1rem;
}

.modal-content {
  background: var(--surface-color);
  border: 1px solid var(--border-color);
  width: 100%;
  max-width: 500px;
  border-radius: 12px;
  padding: 2rem;
  position: relative;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.5);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}

.close-btn {
  background: transparent;
  border: none;
  color: var(--text-secondary);
  font-size: 2rem;
  line-height: 1;
  cursor: pointer;
  transition: color 0.2s;
}

.close-btn:hover {
  color: white;
}
</style>
