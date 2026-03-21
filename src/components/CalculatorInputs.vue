<script setup>
import { 
  Stethoscope, 
  TrendingUp, 
  Car, 
  Wallet, 
  ShieldCheck, 
  Info,
  HelpCircle
} from 'lucide-vue-next';
import { ref, onMounted, onUnmounted } from 'vue';

const props = defineProps({
  modelValue: Object
});

const emit = defineEmits(['update:modelValue']);

const activeTooltip = ref(null);

const toggleTooltip = (field) => {
  if (activeTooltip.value === field) {
    activeTooltip.value = null;
  } else {
    activeTooltip.value = field;
  }
};

const closeTooltips = () => {
  activeTooltip.value = null;
};

onMounted(() => {
  document.addEventListener('click', closeTooltips);
});

onUnmounted(() => {
  document.removeEventListener('click', closeTooltips);
});

const updateField = (field, value) => {
  emit('update:modelValue', { ...props.modelValue, [field]: value });
};

const descriptions = {
  medical: "Total costs of medical treatments related to the injury, including hospital visits, surgeries, rehabilitation, medications, and anticipated future care.\n\nExample: $15,000 for ER visit, surgery, and physical therapy after a car accident.",
  lostIncome: "Income lost due to the injury, encompassing time off work, reduced hours, or inability to return to previous employment. Future income loss can also be included.\n\nExample: $8,000 in missed wages during a two-month recovery period.",
  propertyDamage: "Costs to repair or replace property damaged in the incident, such as vehicles, electronics, or personal belongings.\n\nExample: $3,000 to repair a car after a collision.",
  outOfPocket: "Miscellaneous costs personally paid due to the incident, like transportation to medical appointments or hiring assistance at home.\n\nExample: $200 in Uber rides to doctor visits.",
  multiplier: "A number representing the severity of physical pain, emotional distress, and overall life impact. A higher number indicates more serious or long-lasting injuries.\n\n• 1 = Minor cuts and bruises\n• 3 = Broken bone with moderate recovery time\n• 5 = Long-term disability or chronic pain",
  fault: "Percentage of fault attributed to the injured party. In many states, compensation may be reduced if the injured party is partially at fault.\n\nExample: If you are found to be 20% at fault, your compensation may be reduced by 20%."
};
</script>

<template>
  <div class="inputs-section fade-up">
    <h2 class="section-title">
      <TrendingUp :size="16" /> ENTER YOUR NUMBERS
    </h2>

    <div class="input-grid">
      <!-- Gastos Médicos -->
      <div class="input-group">
        <div class="input-label-container">
          <label class="input-label">Medical Expenses</label>
          <button class="info-toggle" @mouseenter="activeTooltip = 'medical'" @mouseleave="activeTooltip = null" @click.stop="toggleTooltip('medical')">
            <HelpCircle :size="14" />
            <div v-if="activeTooltip === 'medical'" class="info-popup">
              <strong>Medical Expenses</strong>
              {{ descriptions.medical }}
            </div>
          </button>
        </div>
        <div class="input-container">
          <Stethoscope class="input-icon" />
          <input 
            type="number" 
            class="currency-input" 
            :value="modelValue.medicalExpenses"
            @input="updateField('medicalExpenses', $event.target.value)"
            placeholder="0"
          />
        </div>
      </div>

      <!-- Ingresos Perdidos -->
      <div class="input-group">
        <div class="input-label-container">
          <label class="input-label">Lost Income</label>
          <button class="info-toggle" @mouseenter="activeTooltip = 'lostIncome'" @mouseleave="activeTooltip = null" @click.stop="toggleTooltip('lostIncome')">
            <HelpCircle :size="14" />
            <div v-if="activeTooltip === 'lostIncome'" class="info-popup">
              <strong>Lost Income</strong>
              {{ descriptions.lostIncome }}
            </div>
          </button>
        </div>
        <div class="input-container">
          <Wallet class="input-icon" />
          <input 
            type="number" 
            class="currency-input" 
            :value="modelValue.lostIncome"
            @input="updateField('lostIncome', $event.target.value)"
            placeholder="0"
          />
        </div>
      </div>

      <!-- Daños Propiedad -->
      <div class="input-group">
        <div class="input-label-container">
          <label class="input-label">Property Damage</label>
          <button class="info-toggle" @mouseenter="activeTooltip = 'propertyDamage'" @mouseleave="activeTooltip = null" @click.stop="toggleTooltip('propertyDamage')">
            <HelpCircle :size="14" />
            <div v-if="activeTooltip === 'propertyDamage'" class="info-popup">
              <strong>Property Damage</strong>
              {{ descriptions.propertyDamage }}
            </div>
          </button>
        </div>
        <div class="input-container">
          <Car class="input-icon" />
          <input 
            type="number" 
            class="currency-input" 
            :value="modelValue.propertyDamage"
            @input="updateField('propertyDamage', $event.target.value)"
            placeholder="0"
          />
        </div>
      </div>

      <!-- Gastos Bolsillo -->
      <div class="input-group">
        <div class="input-label-container">
          <label class="input-label">Out-of-Pocket Expenses</label>
          <button class="info-toggle" @mouseenter="activeTooltip = 'outOfPocket'" @mouseleave="activeTooltip = null" @click.stop="toggleTooltip('outOfPocket')">
            <HelpCircle :size="14" />
            <div v-if="activeTooltip === 'outOfPocket'" class="info-popup">
              <strong>Out-of-Pocket Expenses</strong>
              {{ descriptions.outOfPocket }}
            </div>
          </button>
        </div>
        <div class="input-container">
          <TrendingUp class="input-icon" />
          <input 
            type="number" 
            class="currency-input" 
            :value="modelValue.outOfPocket"
            @input="updateField('outOfPocket', $event.target.value)"
            placeholder="0"
          />
        </div>
      </div>

      <!-- Multiplicador Dolor -->
      <div class="input-group">
        <div class="input-label-container">
          <label class="input-label">Pain & Suffering Multiplier ({{ modelValue.multiplier }}x)</label>
          <button class="info-toggle" @mouseenter="activeTooltip = 'multiplier'" @mouseleave="activeTooltip = null" @click.stop="toggleTooltip('multiplier')">
            <HelpCircle :size="14" />
            <div v-if="activeTooltip === 'multiplier'" class="info-popup">
              <strong>Pain & Suffering Multiplier</strong>
              {{ descriptions.multiplier }}
            </div>
          </button>
        </div>
        <input 
          type="range" 
          min="1" 
          max="5" 
          step="0.5"
          :value="modelValue.multiplier"
          @input="updateField('multiplier', $event.target.value)"
        />
        <div class="slider-info">
          <span>1: Minor</span>
          <span>3: Moderate</span>
          <span>5: Severe</span>
        </div>
      </div>

      <!-- Nivel de Culpa -->
      <div class="input-group">
        <div class="input-label-container">
          <label class="input-label">Fault or Liability Level ({{ modelValue.fault }}%)</label>
          <button class="info-toggle" @mouseenter="activeTooltip = 'fault'" @mouseleave="activeTooltip = null" @click.stop="toggleTooltip('fault')">
            <HelpCircle :size="14" />
            <div v-if="activeTooltip === 'fault'" class="info-popup">
              <strong>Fault or Liability Level</strong>
              {{ descriptions.fault }}
            </div>
          </button>
        </div>
        <input 
          type="range" 
          min="0" 
          max="100" 
          :value="modelValue.fault"
          @input="updateField('fault', $event.target.value)"
        />
        <p class="warning-text" v-if="modelValue.fault >= 50">
          Note: In Florida, if your fault is ≥ 50%, compensation is $0.
        </p>
      </div>

      <!-- Límite Póliza -->
      <div class="input-group">
        <label class="input-label">Policy Limit ($)</label>
        <div class="input-container">
          <ShieldCheck class="input-icon" />
          <input 
            type="number" 
            class="currency-input" 
            :value="modelValue.policyLimit"
            @input="updateField('policyLimit', $event.target.value)"
            placeholder="No limit"
          />
        </div>
        <p class="slider-info" style="margin-top: 0.5rem">
          <Info :size="12" /> Maximum compensation cap from the insurer.
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Individual scoped overrides if necessary, but using global calculator.css */
</style>
