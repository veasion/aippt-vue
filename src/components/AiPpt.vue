<template>
  <div>
    <GenerateOutline v-if="step == 1" :token="token" @nextStep="handleOutline" />
    <SelectTemplate v-if="step == 2" :token="token" @nextStep="selectTemplate" />
    <GeneratePpt v-if="step == 3" :token="token" :params="{ templateId, outline }" />
  </div>
</template>

<script setup lang="ts">
import { ref, watchEffect } from 'vue'
import GenerateOutline from './GenerateOutline.vue'
import SelectTemplate from './SelectTemplate.vue'
import GeneratePpt from './GeneratePpt.vue'

// 文多多AiPPT
// 官网 https://docmee.cn
// 开放平台 https://docmee.cn/open-platform/api#接口鉴权

// api key
const apiKey = ''
// 用户ID（数据隔离）
const uid = 'test'

const step = ref(1)
const outline = ref('')
const templateId = ref("")
const token = ref(localStorage.getItem('token') || '')

// 这里只是demo演示，创建token请在服务端调用
async function createApiToken() {
  if (!apiKey) {
    alert('请在代码中设置apiKey')
    return
  }
  const url = 'https://docmee.cn/api/user/createApiToken'
  const resp = await (await fetch(url, {
    method: 'POST',
    headers: {
      'Api-Key': apiKey,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      uid,
      limit: null
    })
  })).json()
  if (resp.code != 0) {
    alert('创建接口token异常：' + resp.message)
    return
  }
  token.value = resp.data.token
  watchEffect(() => {
    localStorage.setItem('token', token.value || '')
  })
}

function handleOutline(_outline: string) {
  step.value++
  outline.value = _outline
}

function selectTemplate(id: string) {
  step.value++
  templateId.value = id
}

createApiToken()
</script>

<style scoped>
</style>
