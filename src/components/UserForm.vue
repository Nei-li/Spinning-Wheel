<template>
  <div class="form-container">
    <form @submit.prevent="submitForm">

      <!-- the validation check must be done from backend -->

      <h2>Please fill the form to spin the wheel</h2>

      <!-- Name -->
      <div class="form-group">
        <label for="name">Full Name</label>
        <input 
          id="name"
          v-model="form.name"
          type="text"
          placeholder="Enter your full name"
          required
        />
      </div>

      <!-- Email -->
      <div class="form-group">
        <label for="email">Email Address</label>
        <input 
          id="email"
          v-model="form.email"
          type="email"
          placeholder="Enter your email"
          required
        />
      </div>

      <!-- Address -->
      <div class="form-group">
        <label for="address">Address</label>
        <input 
          id="address"
          v-model="form.address"
          type="text"
          placeholder="Enter your address"
        />
      </div>

      <!-- Phone Number -->
      <div class="form-group">
        <label for="number">Phone Number</label>
        <input 
          id="number"
          v-model="form.number"
          type="tel"
          placeholder="Enter your phone number"
        />
      </div>

      <button type="submit">Play</button>

    </form>

  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'

const emit = defineEmits(['formSubmitted'])     

const form = reactive({
  name: '',
  email: '',
  address: '',
  number: ''
})

const showWheel = ref(false)

const submitForm = () => {
  console.log("Form Submitted:", form)
    emit('formSubmitted', form)
}
</script>

<style scoped>
.form-container {
  max-width: 400px;
  margin: 50px auto;
  padding: 25px;
  border: 1px black solid;
  border-radius: 10px;
  background: #667eea;
  box-shadow: 0 4px 10px rgba(0,0,0,0.1);
}

h2 {
  text-align: center;
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 15px;
  display: flex;
  flex-direction: column;
}

label {
  font-weight: 600;
  margin-bottom: 5px;
}

input {
  padding: 8px;
  border-radius: 6px;
  border: 1px solid #ccc;
  transition: 0.3s;
}

input:focus {
  border-color: #42b983;        
  outline: none;
}

button {
  width: 100%;
  padding: 10px;
  border: none;
  border-radius: 6px;
  background: #764ba2;
  color: white;
  font-weight: bold;
  cursor: pointer;
  transition: 0.3s;
}

button:hover {
  background: #369f6e;
}
</style>
