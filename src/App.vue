<template>
  <div class="app-container">
    <div class="content-container">
      <!-- <div class="card" :class="{ 'card-expanded': base.isStop }"> -->
      <el-form v-if="!base.isStop" :model="base" :rules="rules" ref="formRef" label-width="0">
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
      <el-button v-if="base.isStop" class="floating-button" type="danger" @click="stopAction">
        STOP
      </el-button>
      <iframe v-if="base.isStop" :src="base.textInput" ref="iframeRef" class="responsive-iframe"></iframe>
      <!-- </div> -->
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
  refreshUrl: 'refreshUrl',
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
      localStorage.setItem(base.refreshUrl, base.textInput)

      const iframe = iframeRef.value
      if (iframe) {
        iframe.src = base.textInput
      }

      setTimeout(() => {
        location.reload()
      }, convertToMilliseconds(base.number,base.timeValue))

      base.isStop = true
    } else {
      ElMessage.error('Please check the form fields and try again.')
      return false
    }
  })
}

const stopAction = () => {
  localStorage.removeItem(base.refreshUrl)
  base.textInput = ''
  base.isStop = false
}

const handleChange = (value) => {
  console.log(value)
}

const convertToMilliseconds = (value, timeValue) => {
  console.log("value :",value);
  
  return (!timeValue) ? value * 1000 : value * 60 * 1000;
}

onMounted(() => {
  const getUrl = localStorage.getItem(base.refreshUrl)
  if (getUrl) {
    base.textInput = getUrl
    base.isValidUrl = true
    runAction()
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
  margin-top: 20px;
  background-color: #e74c3c;
  border: none;
  color: white;
  padding: 10px 20px;
  font-size: 16px;
  border-radius: 5px;
  cursor: pointer;
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
}

.floating-button:hover {
  background-color: #c0392b;
}

.responsive-iframe {
  flex: 1;
  width: 100%;
  height: 100vh;
  border: none;
}

.el-input-number.is-controls-right {
  width: 75px;
}

.switch {
  width: 225px;
    justify-content: space-evenly;
}

button.el-button.el-button--primary{
  width: 300px
}
</style>
