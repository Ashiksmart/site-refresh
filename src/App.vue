<template>
  <div class="app-container">
    <div v-if="!base.isStop" class="container-layout">
      <div class="header">
        <h1>Welcome to My App</h1>
        <p>Your go-to solution for [describe the purpose of your app, e.g., tracking business hours].</p>
      </div>
      <div class="container-inside">
        <div class="features">
          <ul>
            <li>✔️ Real-time updates on business statuses</li>
            <li>✔️ User-friendly interface</li>
            <li>✔️ Customizable notifications</li>
            <li>✔️ Comprehensive reporting tools</li>
          </ul>
        </div>
        <div class="content-container .pd-20">
          <el-form :model="base" :rules="rules" ref="formRef" label-width="0">
            <el-col>
              <el-row>
                <el-form-item prop="textInput">
                  <el-input v-model="base.textInput" placeholder="Enter a valid URL"
                    :class="{ invalid: !base.isValidUrl && base.textInput !== '' }" @input="validateUrl" />
                </el-form-item>
              </el-row>
              <el-row>
                <el-form-item>
                  <el-input-number v-model="base.number" :min="1" :max="60" controls-position="right"
                    @change="handleChange" />
                </el-form-item>
                <el-form-item>
                  <el-row class="switch">
                    SECONDS
                    <el-switch v-model="base.timeValue" />
                    MINUTES
                  </el-row>
                </el-form-item>
                <el-form-item>
                  <el-button type="primary" @click="runAction">START TO REFRESH</el-button>
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

      const iframe = iframeRef.value
      if (iframe) {
        iframe.src = refreshValue.url
      }

      setTimeout(() => {
        location.reload()
      }, convertToMilliseconds(refreshValue.timeCount, refreshValue.timeMode))

      base.isStop = true
    } else {
      ElMessage.error('Please check the form fields and try again.')
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
  console.log(value)
}

const convertToMilliseconds = (value, timeValue) => {
  console.log("value :", value, timeValue);

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
.app-container {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  font-family: 'Roboto', sans-serif;
}

.content-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
}

.pd-20 {
  padding: 20px;
}

.el-input {
  width: 600px;
  height: 50px;
  border-radius: 5px;
}

.invalid {
  border-color: red !important;
}

.floating-button {
  position: absolute;
  right: 20px;
  margin-top: 20px;
  background-color: #e74c3c;
  border: none;
  color: white;
  padding: 10px 20px;
  font-size: 16px;
  border-radius: 5px;
  cursor: pointer;
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
  bottom: 20px;
}

.floating-button:hover {
  background-color: #c0392b;
}

.responsive-iframe {
  flex: 1;
  width: 100%;
  height: inherit;
  border: none;
}

.el-input-number.is-controls-right {
  width: 75px;
}

.switch {
  width: 225px;
  justify-content: space-evenly;
}

button.el-button.el-button--primary {
  width: 300px
}

.container-layout {
  /* background: blanchedalmond; */
  width: inherit;
  padding: 20px;
}

.container-inside {
  display: flex;
  /* background: aquamarine; */
  flex-direction: row;
  justify-content: center;
  height: 200px;
  padding: 20px;
}

.header {
  margin-bottom: 40px;
}

.features {
  margin: 20px 0;
}

.features ul {
  list-style-type: none;
}
</style>
