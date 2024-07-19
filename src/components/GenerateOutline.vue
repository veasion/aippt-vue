<template>
  <div class="content">
      <h1>🤖 AI智能生成PPT演示文稿</h1>
      <div class="desc">生成大纲 ---&gt; 挑选模板 --&gt; 实时生成PPT</div>
      <div>
        <span>主题：</span>
        <input v-model="subject" placeholder="请输入PPT主题" />
        <button @click="generateOutline">生成大纲</button>
        <button v-if="genDone" @click="nextStep">下一步：选择模板</button>
      </div>
      <div v-html="outlineHtml" class="outline"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, defineEmits } from 'vue'
import { marked } from 'marked'
import { SSE } from '../utils/sse.js'

const props = defineProps<{
  token: string
}>()

const subject = ref('')
const outline = ref('')
const outlineHtml = ref('')
const genDone = ref(false)
const $emit = defineEmits(['nextStep'])

marked.setOptions({
  renderer: new marked.Renderer(),
  gfm: true,
  async: false,
  breaks: false,
  pedantic: false,
  silent: true
})

function generateOutline() {
  if (!subject.value) {
    alert('请输入主题')
    return
  }
  console.log('主题', subject.value)
  const url = 'https://chatmee.cn/api/ppt/generateOutline'
  var source = new SSE(url, {
      method: 'POST',
      // withCredentials: true,
      headers: {
          'Content-Type': 'application/json',
          'Cache-Control': 'no-cache',
          'token': props.token
      },
      payload: JSON.stringify({ subject: subject.value }),
  })
  source.onmessage = function (data: any) {
    let json = JSON.parse(data.data)
    if (json.status == -1) {
      alert('生成大纲异常：' + json.error)
      return
    }
    outline.value += json.text
    outlineHtml.value = marked.parse(outline.value.replace('```markdown', '').replace(/```/g, '')) as string
    window.scrollTo({ behavior: 'smooth', top: document.body.scrollHeight })
  }
  source.onend = function (data: any) {
    if (data.data.startsWith('{') && data.data.endWith('}')) {
      let json = JSON.parse(data.data)
      if (json.code != 0) {
        alert('生成大纲异常：' + json.message)
        return
      }
    }
    genDone.value = true
    window.scrollTo(0, 0)
  }
  source.onerror = function (err: any) {
    console.error('生成大纲异常', err)
    alert('生成大纲异常')
  }
  source.stream()
}

function nextStep() {
  $emit('nextStep', outline.value)
}

</script>

<style scoped>
.content {
  padding-top: 4em;
  text-align: center;
}
button {
  margin-left: .8em;
}
.desc {
  font-size: 1em;
  margin-top: 1em;
  color: #999;
  margin-bottom: 4em;
}
.outline {
  text-align: left;
  margin: 0 calc(20vw);
}
</style>
