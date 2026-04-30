<script setup>
import { ref, computed, watch } from 'vue';
import html2pdf from 'html2pdf.js';
import CalculatorInputs from './CalculatorInputs.vue';
import CalculatorResults from './CalculatorResults.vue';
import CalculatorChart from './CalculatorChart.vue';
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
    // Top-level explicit fields to match requested data
    "Was the accident in Florida?": formData.accidentInFlorida,
    "When did your accident occur?": formData.whenOccurred,
    "What type of accident or incident were you involved in?": formData.accidentType,
    "Were you or someone else injured in this accident?": formData.wereInjured,
    "Full Name": formData.name,
    "Email Address": formData.email,
    "Phone Number": formData.phone,
    "Medical Expenses": inputs.value.medicalExpenses || 0,
    "Lost Income": inputs.value.lostIncome || 0,
    "Property Damage": inputs.value.propertyDamage || 0,
    "Out-of-Pocket Expenses": inputs.value.outOfPocket || 0,
    "Pain & Suffering Multiplier": inputs.value.multiplier,
    "Fault or Liability Level": inputs.value.fault,
    "Policy Limit": inputs.value.policyLimit || 'No Limit',
    "Estimated Case Value": formatCurrency(results.value.finalSettlement),

    // Structured data
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
    const originalElement = document.getElementById('pdf-report');
    
    // Create an isolated clone to insert into DOM for guaranteed layout calculation
    const clone = originalElement.cloneNode(true);
    clone.id = 'pdf-report-clone';
    clone.style.display = 'block';
    
    const wrapper = document.createElement('div');
    wrapper.style.position = 'fixed';
    wrapper.style.top = '0';
    wrapper.style.left = '0';
    wrapper.style.opacity = '0.001';
    wrapper.style.pointerEvents = 'none';
    wrapper.style.zIndex = '-9999';
    wrapper.appendChild(clone);
    document.body.appendChild(wrapper);

    // Wait for the browser to recalculate the layout bounding box
    await new Promise(resolve => setTimeout(resolve, 50));

    const pdfBase64 = await html2pdf().from(clone).set({
      margin: 0,
      filename: `Case-Evaluation-Report.pdf`,
      image: { type: 'jpeg', quality: 0.98 },
      pagebreak: { mode: 'css' },
      html2canvas: { 
        scale: 2, 
        useCORS: true,
        scrollY: 0,
        scrollX: 0,
        backgroundColor: '#030712',
        width: 794, // 210mm at 96dpi is ~794px
        onclone: (clonedDoc) => {
          clonedDoc.body.style.backgroundColor = '#030712';
          const report = clonedDoc.getElementById('pdf-report-clone');
          if (report) {
            report.parentNode.style.opacity = '1';
            report.style.display = 'block';
          }
        }
      },
      jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait', compress: true }
    }).outputPdf('datauristring');
    
    // Clean up
    document.body.removeChild(wrapper);
    
    payload['PDF Attachment'] = pdfBase64;
  } catch (err) {
    console.error('PDF generation failed:', err);
  }

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

// DEV TEST FUNCTION
const previewPdf = async () => {
  const originalElement = document.getElementById('pdf-report');
  
  // Create an isolated clone to insert into DOM for guaranteed layout calculation
  const clone = originalElement.cloneNode(true);
  clone.id = 'pdf-report-clone';
  clone.style.display = 'block';
  
  const wrapper = document.createElement('div');
  wrapper.style.position = 'fixed';
  wrapper.style.top = '0';
  wrapper.style.left = '0';
  wrapper.style.opacity = '0.001';
  wrapper.style.pointerEvents = 'none';
  wrapper.style.zIndex = '-9999';
  wrapper.appendChild(clone);
  document.body.appendChild(wrapper);

  // Wait for the browser to recalculate the layout bounding box
  await new Promise(resolve => setTimeout(resolve, 50));

  await html2pdf().from(clone).set({
    margin: 0,
    filename: `Preview-Case-Evaluation-Report.pdf`,
    image: { type: 'jpeg', quality: 0.98 },
    pagebreak: { mode: 'css' },
    html2canvas: { 
      scale: 2, 
      useCORS: true,
      scrollY: 0,
      scrollX: 0,
      backgroundColor: '#030712',
      width: 794, 
      onclone: (clonedDoc) => {
        clonedDoc.body.style.backgroundColor = '#030712';
        const report = clonedDoc.getElementById('pdf-report-clone');
        if (report) {
          report.parentNode.style.opacity = '1';
          report.style.display = 'block';
        }
      }
    },
    jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait', compress: true }
  }).save();

  // Clean up
  document.body.removeChild(wrapper);
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
    <div class="container capture-area">
      <header class="calculator-header text-center fade-up" style="margin-bottom: 3rem;">
        <h1 class="main-title">ESTIMATE YOUR<br>CASE VALUE</h1>
        <p style="color: var(--text-secondary); margin-top: 1.5rem; font-weight: 500;" class="header-subtitle">
          Personal Injury Calculator based on Florida legal standards
        </p>
        <button class="btn-primary pdf-hidden" @click="previewPdf" style="margin-top: 1rem; padding: 0.5rem 1rem; background: #666; font-size: 0.875rem;">
          Download PDF Preview
        </button>
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

    <!-- Executive Report for PDF Generator (Off-screen) -->
    <div class="pdf-executive-report" id="pdf-report">
      
      <!-- PAGE 1 -->
      <div class="pdf-page">
        <div class="report-header">
          <div class="header-left">
            <h1 class="report-firm-name">YOUR CASE VALUE ESTIMATE</h1>
            <p class="report-firm-tagline">Disclaimer: The figure provided is an estimate. Consult with a qualified attorney to discuss the merits of your case and confirm your evaluation.</p>
          </div>
        </div>

        <div class="report-body-wrapper">
        <div class="report-title-section">
          <h2>EXECUTIVE SUMMARY</h2>
          <p>Prepared on: {{ new Date().toLocaleDateString() }}</p>
        </div>

        <div class="report-grid">
          <div class="report-section">
            <h3>1. Input Variables</h3>
            <table class="report-table">
              <tbody>
                <tr><td>Medical Expenses</td><td>{{ formatCurrency(inputs.medicalExpenses) }}</td></tr>
                <tr><td>Lost Income</td><td>{{ formatCurrency(inputs.lostIncome) }}</td></tr>
                <tr><td>Property Damage</td><td>{{ formatCurrency(inputs.propertyDamage) }}</td></tr>
                <tr><td>Out-of-Pocket</td><td>{{ formatCurrency(inputs.outOfPocket) }}</td></tr>
                <tr><td>Pain/Suffering Multiplier</td><td>{{ inputs.multiplier }}x</td></tr>
                <tr><td>Fault Proportion</td><td>{{ inputs.fault }}%</td></tr>
                <tr v-if="inputs.policyLimit"><td>Policy Limit</td><td>{{ formatCurrency(inputs.policyLimit) }}</td></tr>
              </tbody>
            </table>
          </div>

          <div class="report-section">
            <h3>2. Evaluation Breakdown</h3>
            <table class="report-table">
              <tbody>
                <tr><td><strong>Economic Losses</strong></td><td><strong>{{ formatCurrency(results.economicLosses) }}</strong></td></tr>
                <tr><td><strong>Pain and Suffering</strong></td><td><strong>{{ formatCurrency(results.painAndSuffering) }}</strong></td></tr>
                <tr><td>Total Before Adjustments</td><td>{{ formatCurrency(results.totalBeforeAdjustment) }}</td></tr>
                <tr v-if="results.fault > 0"><td class="red">Fault Deduction</td><td class="red">-{{ formatCurrency(results.faultDeduction) }}</td></tr>
                <tr v-if="results.isLimited"><td class="red">Policy Limit Applied</td><td class="red">Cap Reached</td></tr>
              </tbody>
            </table>
          </div>
        </div>
        
        <div class="report-summary-box">
          <h3>ESTIMATED CASE VALUE</h3>
          <h1 class="final-value">{{ formatCurrency(results.finalSettlement) }}</h1>
          <p v-if="results.fault >= 50" class="red text-center" style="margin-top: 10px; color: #ff8a8a !important;">Note: Florida law prevents recovery if fault is 50% or higher.</p>
        </div>
        </div> <!-- Close report-body-wrapper -->
      </div>

      <!-- PAGE 2 -->
      <div class="pdf-page">
        <div class="report-header">
          <div class="header-left">
            <h1 class="report-firm-name">YOUR CASE VALUE ESTIMATE</h1>
            <p class="report-firm-tagline">Disclaimer: The figure provided is an estimate. Consult with a qualified attorney to discuss the merits of your case and confirm your evaluation.</p>
          </div>
        </div>

        <div class="report-body-wrapper" style="flex: 1; display: flex; flex-direction: column;">
          <div class="report-section report-chart-section">
            <h3>3. Visual Breakdown</h3>
            <div class="chart-wrapper">
               <CalculatorChart v-if="results.totalBeforeAdjustment > 0" :economicLosses="results.economicLosses" :painAndSuffering="results.painAndSuffering" />
            </div>
          </div>
        
          <div class="report-footer" style="margin-top: auto;">
            <p><strong>Disclaimer:</strong> This executive summary is an algorithmic estimation based on standard Florida personal injury guidelines. It does not constitute legal advice or a guarantee of recovery. Insurance companies actively work to minimize payouts. Consult with our qualified attorneys to establish formal legal strategy.</p>
          </div>
        </div>
      </div>
      
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
          <h2 style="color: white; font-size: 1.5rem;">Finalize Your case estimate</h2>
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
  width: 1200px;
  max-width: 100%;
  border-radius: 12px;
  padding: 2rem;
  position: relative;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.5);
  max-height: 90vh;
  overflow-y: auto;
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
