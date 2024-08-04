<template>
  <div class="app-container">
    <div class="content-container"
      :style="!base.isStop ? { maxWidth: '500px', padding: '20px' } : { maxWidth: '100%' }">
      <el-form v-if="!base.isStop" :model="base" :rules="rules" ref="formRef" label-width="0">
        <el-form-item prop="textInput">
          <el-input v-model="base.textInput" :placeholder="'Enter a valid URL'"
            :class="{ invalid: !base.isValidUrl && base.textInput !== '' }" @input="validateUrl" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="runAction">
            RUN
          </el-button>
        </el-form-item>
      </el-form>
      <el-button v-if="base.isStop" class="floating-button" type="danger" @click="stopAction">
        STOP
      </el-button>
      <iframe v-if="base.isStop" :src="base.textInput" ref="iframeRef" class="responsive-iframe"></iframe>
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
    console.log("valid: ", valid);

    if (valid) {
      console.log('Running action with input:', base.textInput)
      localStorage.setItem(base.refreshUrl, base.textInput)

      const iframe = iframeRef.value
      if (iframe) {
        iframe.src = base.textInput
      }

      setTimeout(() => {
        console.log('SETTIMEOUT')
        location.reload();
      }, 5000)

      base.isStop = true
    } else {
      ElMessage.error('Please check the form fields and try again.')
      return false
    }
  })
}

const stopAction = () => {
  console.log('Stop action triggered')
  localStorage.removeItem(base.refreshUrl)
  base.textInput = ''
  base.isStop = false
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
  background: linear-gradient(135deg, #6b73ff 0%, #000dff 100%);
}

.content-container {
  background: rgba(255, 255, 255, 0.8);
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  text-align: center;
  width: 100%;
}

.el-input {
  width: calc(100% - 30px);
}

.invalid {
  border-color: red;
}

.el-button {
  margin-top: 10px;
}

.floating-button {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background-color: #dc3545;
  border: none;
  color: white;
  padding: 15px 20px;
  font-size: 16px;
  border-radius: 50%;
  cursor: pointer;
}

.floating-button:hover {
  background-color: #c82333;
}

.responsive-iframe {
  flex: 1;
  width: 100%;
  height: 100vh;
  border: none;
}
</style>
