<script setup>
import { ref } from 'vue';
import { User, Mail, Phone, MessageSquare, Send, Loader2, CheckCircle2, AlertCircle } from 'lucide-vue-next';

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
  name: '',
  email: '',
  phone: '',
  message: ''
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
      <h3>Message Sent!</h3>
      <p>Thank you for reaching out. A legal expert will review your calculation and contact you shortly.</p>
    </div>

    <div v-else-if="status === 'error'" class="status-container error fade-up">
      <AlertCircle :size="48" class="status-icon" />
      <h3>Submission Failed</h3>
      <p>{{ errorMessage || 'There was an error sending your message. Please try again or call us directly.' }}</p>
      <button class="btn-primary" @click="$emit('reset-status')">Try Again</button>
    </div>

    <form v-else @submit.prevent="handleSubmit" class="fade-up">
      <div class="form-grid">
        <!-- Name -->
        <div class="input-group">
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
        <div class="input-group">
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
        <div class="input-group">
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

        <!-- Message -->
        <div class="input-group full-width">
          <label class="input-label">Brief Description of the Incident (Optional)</label>
          <div class="input-container">
            <MessageSquare class="input-icon textarea-icon" />
            <textarea 
              v-model="formData.message" 
              placeholder="Tell us what happened..."
              :disabled="status === 'loading'"
              rows="4"
            ></textarea>
          </div>
        </div>
      </div>

      <button type="submit" class="btn-primary submit-btn" :disabled="status === 'loading'">
        <template v-if="status === 'loading'">
          <Loader2 class="spinner" :size="20" />
          Sending...
        </template>
        <template v-else>
          <Send :size="20" />
          Maximize My Payout
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

input, textarea {
  width: 100%;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid var(--border-color);
  color: white;
  padding: 0.75rem 1rem 0.75rem 3rem;
  border-radius: 8px;
  font-family: inherit;
  font-size: 1rem;
  transition: all 0.2s;
}

textarea {
  resize: vertical;
}

input:focus, textarea:focus {
  outline: none;
  border-color: white;
  background: rgba(255, 255, 255, 0.08);
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
