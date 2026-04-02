<script setup>
import { ref } from 'vue';
import { User, Mail, Phone, MessageSquare, Send, Loader2, CheckCircle2, AlertCircle, HelpCircle, ChevronDown } from 'lucide-vue-next';

const props = defineProps({
  status: {
    type: String,
    default: 'idle' // 'idle', 'loading', 'success', 'error'
  },
  errorMessage: {
    type: String,
    default: ''
  }
});

const emit = defineEmits(['submit']);

const formData = ref({
  accidentInFlorida: '',
  whenOccurred: '',
  accidentType: '',
  wereInjured: '',
  name: '',
  email: '',
  phone: '',
  receiveEmail: false,
  consentSMS: false
});

const handleSubmit = () => {
  if (props.status === 'loading') return;
  emit('submit', { ...formData.value });
};
</script>

<template>
  <div class="contact-form">
    <div v-if="status === 'success'" class="status-container success fade-up">
      <CheckCircle2 :size="48" class="status-icon" />
      <h3>THANK YOU. <br>YOUR CASE EVALUATION HAS BEEN INITIATED.</h3>
      <p>A representative will contact you shortly at the number provided to discuss the specific details of your claim. Due to the high-stakes nature of these cases, we prioritize evaluations based on the severity of the injury and the urgency of the legal deadlines.</p>
    </div>

    <div v-else-if="status === 'error'" class="status-container error fade-up">
      <AlertCircle :size="48" class="status-icon" />
      <h3>Submission Failed</h3>
      <p>{{ errorMessage || 'There was an error sending your message. Please try again or call us directly.' }}</p>
      <button class="btn-primary" @click="$emit('reset-status')">Try Again</button>
    </div>

    <form v-else @submit.prevent="handleSubmit" class="fade-up">
      <div class="form-grid">
        <!-- Questionnaire Fields -->
        <div class="input-group full-width">
          <label class="input-label">Was the accident in Florida? *</label>
          <div class="input-container">
            <HelpCircle class="input-icon" />
            <select v-model="formData.accidentInFlorida" required :disabled="status === 'loading'">
              <option value="" disabled>Select an option</option>
              <option value="Yes">Yes</option>
              <option value="No">No</option>
            </select>
            <ChevronDown class="select-arrow" />
          </div>
        </div>

        <div class="input-group full-width">
          <label class="input-label">When did your accident occur? *</label>
          <div class="input-container">
            <HelpCircle class="input-icon" />
            <select v-model="formData.whenOccurred" required :disabled="status === 'loading'">
              <option value="" disabled>Select an option</option>
              <option value="Within the last 6 months">Within the last 6 months</option>
              <option value="6–12 months ago">6–12 months ago</option>
              <option value="Over a year ago">Over a year ago</option>
            </select>
            <ChevronDown class="select-arrow" />
          </div>
        </div>

        <div class="input-group full-width">
          <label class="input-label">What type of accident or incident were you involved in? *</label>
          <div class="input-container">
            <HelpCircle class="input-icon" />
            <select v-model="formData.accidentType" required :disabled="status === 'loading'">
              <option value="" disabled>Select an option</option>
              <option value="Car Accident">Car Accident</option>
              <option value="Motorcycle Accident">Motorcycle Accident</option>
              <option value="Truck or Commercial Vehicle Accident">Truck or Commercial Vehicle Accident</option>
              <option value="Bicycle Accident">Bicycle Accident</option>
              <option value="Pedestrian Accident">Pedestrian Accident</option>
              <option value="Work Injury">Work Injury</option>
              <option value="Construction Accident">Construction Accident</option>
              <option value="Other Personal Injury">Other Personal Injury</option>
            </select>
            <ChevronDown class="select-arrow" />
          </div>
        </div>

        <div class="input-group full-width">
          <label class="input-label">Were you or someone else injured in this accident? *</label>
          <div class="input-container">
            <HelpCircle class="input-icon" />
            <select v-model="formData.wereInjured" required :disabled="status === 'loading'">
              <option value="" disabled>Select an option</option>
              <option value="Yes, I needed medical attention.">Yes, I needed medical attention.</option>
              <option value="Minor injuries, but I'm still in pain.">Minor injuries, but I'm still in pain.</option>
              <option value="There were no injuries, only property damage.">There were no injuries, only property damage.</option>
            </select>
            <ChevronDown class="select-arrow" />
          </div>
        </div>

        <hr class="form-separator full-width" />

        <!-- Name -->
        <div class="input-group full-width">
          <label class="input-label">Full Name</label>
          <div class="input-container">
            <User class="input-icon" />
            <input 
              v-model="formData.name" 
              type="text" 
              placeholder="John Doe" 
              required
              :disabled="status === 'loading'"
            />
          </div>
        </div>

        <!-- Email -->
        <div class="input-group full-width">
          <label class="input-label">Email Address</label>
          <div class="input-container">
            <Mail class="input-icon" />
            <input 
              v-model="formData.email" 
              type="email" 
              placeholder="john@example.com" 
              required
              :disabled="status === 'loading'"
            />
          </div>
        </div>

        <!-- Phone -->
        <div class="input-group full-width">
          <label class="input-label">Phone Number</label>
          <div class="input-container">
            <Phone class="input-icon" />
            <input 
              v-model="formData.phone" 
              type="tel" 
              placeholder="(555) 000-0000" 
              required
              :disabled="status === 'loading'"
            />
          </div>
        </div>

        <!-- Consent Checkboxes -->
        <div class="input-group full-width checkbox-container">
          <label class="checkbox-label">
            <input 
              type="checkbox" 
              v-model="formData.receiveEmail" 
              :disabled="status === 'loading'"
              class="custom-checkbox"
            />
            <span class="checkbox-text">Receive the breakdown information via email</span>
          </label>
        </div>

        <div class="input-group full-width checkbox-container">
          <label class="checkbox-label">
            <input 
              type="checkbox" 
              v-model="formData.consentSMS" 
              :disabled="status === 'loading'"
              class="custom-checkbox"
            />
            <span class="checkbox-text">I consent to receive messages related to my case. Message frequency may vary. Message & Data rates may apply. Reply HELP for help or STOP to opt-out.</span>
          </label>
        </div>
      </div>

      <button type="submit" class="btn-primary submit-btn" :disabled="status === 'loading'">
        <template v-if="status === 'loading'">
          <Loader2 class="spinner" :size="20" />
          Sending...
        </template>
        <template v-else>
          <Send :size="20" />
          submit for the most accurate estimate
        </template>
      </button>

      <p class="privacy-note">
        By clicking, you agree to our privacy policy. Your information is encrypted and secure.
      </p>
    </form>
  </div>
</template>

<style scoped>
.contact-form {
  width: 100%;
}

.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
  margin-bottom: 2rem;
}

@media (max-width: 600px) {
  .form-grid {
    grid-template-columns: 1fr;
  }
}

.full-width {
  grid-column: 1 / -1;
}

.input-group {
  margin-bottom: 0;
}

.input-label {
  display: block;
  margin-bottom: 4px;
}

.input-container {
  position: relative;
  display: flex;
  align-items: center;
}

.input-icon {
  position: absolute;
  left: 1rem;
  color: var(--text-secondary);
  width: 18px;
  height: 18px;
  pointer-events: none;
}

.textarea-icon {
  top: 1rem;
}

input, textarea, select {
  width: 100%;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid var(--border-color);
  color: white;
  padding: 0.75rem 1rem 0.75rem 3rem;
  border-radius: 8px;
  font-family: inherit;
  font-size: 1rem;
  transition: all 0.2s;
  appearance: none;
}

select option {
  background-color: #1a1a1a;
  color: white;
}

textarea {
  resize: vertical;
}

.checkbox-container {
  margin-top: 0.5rem;
}

.checkbox-label {
  display: flex;
  align-items: flex-start;
  gap: 0.75rem;
  cursor: pointer;
}

.custom-checkbox {
  width: auto;
  appearance: auto;
  margin-top: 0.25rem;
  padding: 0;
}

.checkbox-text {
  font-size: 0.85rem;
  color: var(--text-secondary);
  line-height: 1.4;
}

input:focus, textarea:focus, select:focus {
  outline: none;
  border-color: white;
  background: rgba(255, 255, 255, 0.08);
}

.select-arrow {
  position: absolute;
  right: 1.25rem;
  color: var(--text-secondary);
  width: 16px;
  height: 16px;
  pointer-events: none;
}

.form-separator {
  border: 0;
  border-top: 1px solid rgba(255,255,255,0.1);
  margin: 1.5rem 0 0.5rem 0;
  width: 100%;
}

.submit-btn {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  font-size: 1.1rem;
  padding: 1.25rem;
}

.privacy-note {
  font-size: 0.75rem;
  color: var(--text-secondary);
  text-align: center;
  margin-top: 1.5rem;
  opacity: 0.6;
}

.status-container {
  text-align: center;
  padding: 3rem 1rem;
}

.status-icon {
  margin-bottom: 1.5rem;
}

.success .status-icon { color: #4caf50; }
.error .status-icon { color: #f44336; }

.status-container h3 {
  font-size: 1.75rem;
  margin-bottom: 1rem;
  color: white;
}

.status-container p {
  color: var(--text-secondary);
  margin-bottom: 2rem;
  max-width: 400px;
  margin-left: auto;
  margin-right: auto;
}

.spinner {
  animation: rotate 2s linear infinite;
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes fadeUp {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.fade-up {
  animation: fadeUp 0.4s ease-out forwards;
}
</style>
