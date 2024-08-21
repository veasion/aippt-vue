<template>
  <div class="content">
      <h1>🤖 AI智能生成PPT演示文稿</h1>
      <div class="desc">生成大纲 ---&gt; 挑选模板 --&gt; 实时生成PPT</div>
      <div v-if="genStatus == 0" class="input_div">
        <select v-model="selectType">
          <option value="subject">根据主题</option>
          <option value="text">根据内容</option>
          <option value="file">根据文件</option>
        </select>
        <div v-if="selectType == 'subject'">
          <input v-model="subject" placeholder="请输入PPT主题" maxlength="20" />
        </div>
        <div v-else-if="selectType == 'text'">
          <textarea v-model="text" placeholder="请输入内容" rows="5" cols="50" maxlength="6000"></textarea>
        </div>
        <div v-else-if="selectType == 'file'">
          <input id="input_file" type="file" placeholder="请选择文件" accept=".doc, .docx, .xls, .xlsx, .pdf, .ppt, .pptx, .txt" />
        </div>
        <button @click="generateOutline">生成大纲</button>
      </div>
      <div v-if="genStatus == 1" v-html="outlineHtml" class="outline"></div>
      <div v-if="genStatus == 2">
        <button @click="nextStep">下一步：选择模板</button>
        <OutlineEdit :outlineTree="outlineTree" @update="updateOutline" class="outline_edit" />
      </div>
  </div>
</template>

<script setup lang="ts">
import { ref, defineEmits } from 'vue'
import OutlineEdit from './OutlineEdit.vue'
import { marked } from 'marked'
import { SSE } from '../utils/sse.js'

const props = defineProps<{
  token: string
}>()

const selectType = ref('subject')
const subject = ref('')
const text = ref('')
const dataUrl = ref()

// 生成状态: 0未开始 1生成中 2已完成
const genStatus = ref(0)
const outline = ref('')
const outlineTree = ref()
const outlineHtml = ref('')
const $emit = defineEmits(['nextStep'])

marked.setOptions({
  renderer: new marked.Renderer(),
  gfm: true,
  async: false,
  breaks: false,
  pedantic: false,
  silent: true
})

function parseFileData(formData: FormData) {
  let url = 'https://docmee.cn/api/ppt/parseFileData'
  let xhr = new XMLHttpRequest()
  xhr.open('POST', url, false)
  xhr.setRequestHeader('token', props.token)
  xhr.send(formData)
  if (xhr.readyState === 4) {
    if (xhr.status === 200) {
        let resp = JSON.parse(xhr.responseText)
        if (resp.code != 0) {
            alert('解析文件异常：' + resp.message)
            return null
        }
        return resp.data.dataUrl
    } else {
      alert('解析文件网络异常, httpStatus: ' + xhr.status)
      return null
    }
  }
}

function generateOutline() {
  if (genStatus.value != 0) {
    return
  }
  genStatus.value = 1
  outlineHtml.value = '<h3>正在生成中，请稍后....</h3>'
  const inputData = {} as any
  if (selectType.value == 'subject') {
    // 根据主题
    if (!subject.value) {
      alert('请输入主题')
      genStatus.value = 0
      return
    }
    inputData.subject = subject.value
  } else if (selectType.value == 'text') {
    // 根据内容
    if (!text.value) {
      alert('请输入内容')
      genStatus.value = 0
      return
    }
    const formData = new FormData()
		formData.append('content', text.value)
    inputData.dataUrl = parseFileData(formData)
  } else if (selectType.value == 'file') {
    // 根据文件
    const file = (document.getElementById('input_file') as any)?.files[0]
    if (!file) {
      alert('请选择文件')
      genStatus.value = 0
      return
    }
    const formData = new FormData()
		formData.append('file', file)
    inputData.dataUrl = parseFileData(formData)
  }
  if (!inputData.subject && !inputData.dataUrl) {
    genStatus.value = 0
    return
  }
  genStatus.value = 1
  dataUrl.value = inputData.dataUrl
  const url = 'https://docmee.cn/api/ppt/generateOutline'
  var source = new SSE(url, {
      method: 'POST',
      headers: {
          'token': props.token,
          'Cache-Control': 'no-cache',
          'Content-Type': 'application/json'
      },
      payload: JSON.stringify(inputData),
  }) as any
  source.onmessage = function (data: any) {
    let json = JSON.parse(data.data)
    if (json.status == -1) {
      alert('生成大纲异常：' + json.error)
      genStatus.value = 0
      return
    }
    if (json.status == 4 && json.result) {
      outlineTree.value = json.result
    } else {
      outline.value += json.text
      outlineHtml.value = marked.parse(outline.value.replace('```markdown', '').replace(/```/g, '')) as string
      window.scrollTo({ behavior: 'smooth', top: document.body.scrollHeight })
    }
  }
  source.onend = function (data: any) {
    if (data.data.startsWith('{') && data.data.endsWith('}')) {
      let json = JSON.parse(data.data)
      if (json.code != 0) {
        alert('生成大纲异常：' + json.message)
        genStatus.value = 0
        return
      }
    }
    genStatus.value = 2
    window.scrollTo(0, 0)
  }
  source.onerror = function (err: any) {
    console.error('生成大纲异常', err)
    alert('生成大纲异常')
    genStatus.value = 0
  }
  source.stream()
}

function updateOutline(md: any) {
  outline.value = md + ''
}

function nextStep() {
  $emit('nextStep', { outline: outline.value, dataUrl: dataUrl.value as string })
}

</script>

<style scoped>
.content {
  padding-top: 4em;
  text-align: center;
}
.input_div {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
}
select, button {
  margin: 0 .8em;
}
input[type=file] {
  border: 1px dashed #ccc;
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
.outline_edit {
  width: 32rem;
  margin: 1rem auto;
}
</style>
