<template>
  <div class="app-container">
    <div v-if="!base.isStop" class="container-layout">
      <div class="header">
        <h1>Welcome to Site Refresh Pro</h1>
        <p>Refresh any website at intervals you set, perfect for monitoring live content changes in real-time.</p>
      </div>
      <div class="container-inside">
        <div class="features">
          <ul>
            <li>✔️ Real-time URL refresh capabilities</li>
            <li>✔️ Intuitive and easy-to-use interface</li>
          </ul>
          <ul>
            <li>✔️ Customizable refresh intervals in seconds or minutes</li>
            <li>✔️ Enhanced performance for consistent monitoring</li>
          </ul>
        </div>
        <div class="content-container pd-20">
          <el-form :model="base" :rules="rules" ref="formRef" label-width="0">
            <el-col>
              <el-row>
                <el-form-item prop="textInput">
                  <el-input v-model="base.textInput" placeholder="Enter a valid URL"
                    :class="{ invalid: !base.isValidUrl && base.textInput !== '' }" @input="validateUrl" />
                </el-form-item>
              </el-row>
              <el-row class="button-section">
                <el-form-item>
                  <el-input-number v-model="base.number" :min="1" :max="60" controls-position="right"
                    @focus="handleChange" />
                </el-form-item>
                <el-form-item>
                  <el-row class="switch">
                    <el-switch v-model="base.timeValue" active-text="MINUTES" inactive-text="SECONDS" />
                  </el-row>
                </el-form-item>
                <el-form-item class="start-refresh-btn">
                  <el-button type="primary" @click="runAction">START</el-button>
                </el-form-item>
              </el-row>
            </el-col>
          </el-form>
        </div>
      </div>
    </div>
    <div class="content-container" v-else>
      <el-button class="floating-button" type="danger" @click="stopAction">
        STOP
      </el-button>
      <iframe :src="base.textInput" ref="iframeRef" class="responsive-iframe"></iframe>
    </div>
  </div>
</template>

<script setup>
import { onMounted, reactive, ref } from 'vue'
import { ElMessage } from 'element-plus'

const iframeRef = ref(null)
const formRef = ref(null)

const base = reactive({
  textInput: '',
  number: 0,
  timeValue: false,
  refreshValue: 'refreshValue',
  isValidUrl: false,
  isStop: false
})

const rules = {
  textInput: [
    { required: true, message: 'Please input a URL', trigger: 'blur' },
    {
      pattern: /^(https?):\/\/[^\s$.?#].[^\s]*$/,
      message: 'Please enter a valid URL starting with http:// or https://',
      trigger: 'blur'
    }
  ]
}

const validateUrl = () => {
  formRef.value.validateField('textInput')
  base.isValidUrl = base.textInput === ''
}

const runAction = () => {
  formRef.value.validate((valid) => {
    if (valid) {
      const refreshValue = {
        url: base.textInput,
        timeCount: base.number,
        timeMode: base.timeValue
      }
      localStorage.setItem(base.refreshValue, JSON.stringify(refreshValue))

      // Create a new iframe element
      const iframe = document.createElement('iframe')
      iframe.src = refreshValue.url
      iframe.className = 'responsive-iframe'

      // Add an event listener to handle errors
      iframe.addEventListener('error', () => {
        // Display an error message
        ElMessage.error('Failed to load site. Please check the URL or try again later.')
        // Stop the refresh interval
        stopAction()
      })

      // Replace the existing iframe element with the new one
      const iframeRefElement = iframeRef.value
      if (iframeRefElement) {
        iframeRefElement.parentNode.replaceChild(iframe, iframeRefElement)
      }

      setTimeout(() => {
        location.reload()
      }, convertToMilliseconds(refreshValue.timeCount, refreshValue.timeMode))

      base.isStop = true
    }
  })
}

const stopAction = () => {
  localStorage.removeItem(base.refreshValue)
  base.textInput = ''
  base.number = 0
  base.timeValue = false
  base.isValidUrl = false
  base.isStop = false
}

const handleChange = (value) => {
  if (!value) base.number = 1
}

const convertToMilliseconds = (value, timeValue) => {
  return (!timeValue) ? value * 1000 : value * 60 * 1000;
}

onMounted(() => {
  const refreshValue = localStorage.getItem(base.refreshValue)
  if (refreshValue) {
    const { url, timeCount, timeMode } = JSON.parse(refreshValue)
    if (url) {
      base.textInput = url
      base.number = timeCount
      base.timeValue = timeMode
      base.isValidUrl = true
      runAction()
    }
  }
})
</script>

<style scoped>
/* Dark Theme */
.app-container {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
  background: linear-gradient(135deg, #2c3e50 0%, #4ca1af 100%);
  font-family: 'Roboto', sans-serif;
  color: #ecf0f1;
}

.content-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
}

/* Header Styling */
.header {
  text-align: center;
  margin-bottom: 40px;
}

.header h1 {
  font-size: 2.5rem;
  color: #ecf0f1;
}

.header p {
  font-size: 1.2rem;
  color: #bdc3c7;
}

/* Features Section */
.features {
  margin: 20px;
  padding: 20px;
  background-color: #34495e;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
  display: flex;
}

.features ul {
  list-style-type: none;
  padding: 0;
}

.features li {
  font-size: 1.1rem;
  color: #ecf0f1;
  margin-bottom: 10px;
}

/* Form Styling */
.el-input {
  width: 500px;
  height: 45px;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.invalid {
  border-color: red !important;
}

/* Buttons */
button.el-button.el-button--primary {
  width: 100%;
  max-width: 300px;
  height: 50px;
  background-color: #2980b9;
  border-radius: 8px;
  color: white;
  font-size: 16px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  margin: 0 auto;
}

button.el-button.el-button--primary:hover {
  background-color: #3498db;
}

/* Align Start Button */
.start-refresh-btn {
  display: flex;
  justify-content: center;
  align-items: center;
}

.start-refresh-btn button.el-button.el-button--primary {
  height: 30px;
}

/* Floating Button */
.floating-button {
  position: fixed;
  right: 20px;
  bottom: 20px;
  background-color: #e74c3c;
  color: white;
  padding: 15px 25px;
  border-radius: 50px;
  font-size: 18px;
  cursor: pointer;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.floating-button:hover {
  background-color: #c0392b;
}

/* Iframe */
.responsive-iframe {
  width: 100%;
  height: inherit;
  border: none;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
}

/* General Utility Classes */
.pd-20 {
  padding: 20px;
}

.button-section {
  justify-content: space-around;
}
</style>
