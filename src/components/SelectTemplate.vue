<template>
    <div class="content">
      <div>---- 选择模板 ----</div>
      <div class="but_div">
        <button @click="nextStep">下一步: 生成PPT</button>
      </div>
      <div class="template_div">
        <div v-for="template in templates" :key="template.id" :class="template.id == templateId ? 'template select' : 'template'" @click="selectTemplate(template)">
          <img :src="template.coverUrl + '?token=' + token" />
        </div>
        <div class="template_but">
          <button @click="loadTemplates">换一批</button>
        </div>
        <div v-if="templates.length == 0">模板加载中...</div>
      </div>
    </div>
</template>

<script setup lang="ts">
import { ref, defineEmits } from 'vue'
const props = defineProps<{
  token: string
}>()

const templates = ref([] as any)
const templateId = ref("")
const $emit = defineEmits(['nextStep'])

const background = document.body.style.background

async function loadTemplates() {
  const url = 'https://docmee.cn/api/ppt/randomTemplates'
  const resp = await (await fetch(url, {
    method: 'POST',
    headers: {
      'token': props.token,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ page: 1, size: 28, filters: { type: 1 } })
  })).json()
  if (resp.code != 0) {
    alert('获取模板异常：' + resp.message)
    return
  }
  templates.value = resp.data || []
  selectTemplate(resp.data[0])
}

function selectTemplate(template: any) {
  templateId.value = template.id
  const src = template.coverUrl + '?token=' + props.token
  calcSubjectColor(src).then((color) => {
    document.body.style.background = color
  })
}

async function calcSubjectColor(src: string) {
  let img = new Image()
  await new Promise(resolve => {
      let eqOrigin = src.startsWith('data:') || src.startsWith(document.location.origin) || (src.startsWith('//') && (document.location.protocol + src).startsWith(document.location.origin))
      if (!eqOrigin) {
          img.crossOrigin = 'anonymous'
      }
      img.src = src
      img.onload = function() {
          resolve(img)
      }
      img.onerror = function (e) {
          resolve(null)
      }
  })
  let canvas = document.createElement('canvas')
  let ctx = canvas.getContext('2d') as any
  canvas.width = img.width
  canvas.height = img.height
  ctx.drawImage(img, 0, 0)
  let imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
  let data = imageData.data
  let map = {} as any
  for (let i = 0; i < data.length; i += 4) {
      let r = data[i]
      let g = data[i + 1]
      let b = data[i + 2]
      let rgb = r + g + b
      if (rgb <= 50 || rgb >= 660) {
          // 忽略黑白
          continue
      }
      let color = `${r},${g},${b}`
      map[color] = (map[color] || 0) + 1
  }
  let valueMap = {} as any
  for (let k in map) {
      let v = map[k]
      let ks = valueMap[v]
      if (ks) {
          ks.push(k)
      } else {
          valueMap[v] = [k]
      }
  }
  let colors = []
  let values = Object.values(map).sort() as any
  for (let i = values.length - 1; i >= 0; i--) {
      let ks = valueMap[values[i]]
      for (let j = 0; j < ks.length; j++) {
          colors.push(ks[j])
          if (colors.length >= 3) {
              break
          }
      }
      if (colors.length >= 3) {
          break
      }
  }
  // return `rgb(${colors[0]})`
  return `linear-gradient(to bottom right, rgb(${colors[0]}), rgb(${colors[1]}), rgb(${colors[2]}))`
}

function nextStep() {
  if (!templateId.value) {
    alert('请选择模板')
    return
  }
  document.body.style.background = background
  $emit('nextStep', templateId.value)
}

loadTemplates()
</script>
  
<style scoped>
.content {
  padding-top: 4em;
  text-align: center;
}
.but_div {
  margin-top: 0.8em;
}
.template_div {
  width: 80%;
  max-width: 1050px;
  margin: auto;
  display: flex;
  flex-wrap: wrap;
  flex-direction: row;
  align-items: center;
  text-align: center;
}
.template {
  margin: 8px;
  width: 240px;
  height: 136px;
  cursor: pointer;
  overflow: hidden;
  border-radius: 12px;
  background-color: #ecedf4;
  border: 2px solid transparent;
}
.template img {
  width: 100%;
  height: 100%;
  opacity: 1;
  object-fit: cover;
  transition: all .6s ease;
}
.template img:hover {
  transform: scale(1.2);
  transition-duration: .3s;
}
.template_but {
  width: 100%;
  padding: 10px;
  text-align: center;
}
.select {
  border: 2px solid #491ff8;
}
</style>
