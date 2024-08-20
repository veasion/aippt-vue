<template>
    <div class="outline_div">
        <div class="subject_div">{{ props.outlineTree.name }}</div>
        <div v-for="(chapter, chapterIdx) in props.outlineTree.children" :key="version + '_' + chapterIdx">
            <div class="chapter_div">
                <div class="chapter_title">第{{ chapterIdx + 1 }}章</div>
                <div class="chapter_name">
                    <input v-model="chapter.name" @change="update" />
                </div>
                <div class="operate_div">
                    <span title="在上方新增同级" @click="operate(props.outlineTree.children, chapterIdx, 1)">=+</span>
                    <span title="在下方新增下属" @click="operate(props.outlineTree.children, chapterIdx, 2)">+</span>
                    <span title="删除" @click="operate(props.outlineTree.children, chapterIdx, 3)">x</span>
                </div>
            </div>
            <div v-for="(page, pageIdx) in chapter.children" :key="chapterIdx + '-' + pageIdx">
                <div class="page_div">
                    <span class="page_number">{{ chapterIdx + 1 }}.{{ pageIdx + 1 }}</span>
                    <div class="page_name">
                        <input v-model="page.name" @change="update" />
                    </div>
                    <div class="operate_div">
                        <span title="在上方新增同级" @click="operate(chapter.children, pageIdx, 1)">=+</span>
                        <span title="在下方新增下属" @click="operate(chapter.children, pageIdx, 2)">+</span>
                        <span title="删除" @click="operate(chapter.children, pageIdx, 3)">x</span>
                    </div>
                </div>
                <div v-for="(title, titleIdx) in page.children" :key="chapterIdx + '-' + pageIdx + '-' + titleIdx">
                    <div class="title_div">
                        <span class="title_number">{{ chapterIdx + 1 }}.{{ pageIdx + 1 }}.{{ titleIdx + 1 }}</span>
                        <div class="title_name">
                            <input v-model="title.name" @change="update" />
                        </div>
                        <div class="operate_div">
                            <span title="在上方新增同级" @click="operate(page.children, titleIdx, 1)">=+</span>
                            <span title="删除" @click="operate(page.children, titleIdx, 3)">x</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
  </template>
  
<script setup lang="ts">
import { ref, defineEmits } from 'vue'

const props = defineProps<{
  outlineTree: any
}>()

const version = ref(0)
const $emit = defineEmits(['update'])

function appendMd(children: any) {
  let str = ''
  for (let i = 0; i < children.length; i++) {
      let level = children[i].level
      if (level == 0) {
          str += '- '
      } else {
          for (let j = 0; j < level; j++) {
              str += '#'
          }
          str += ' '
      }
      str += children[i].name + '\n'
      if (children[i].children) {
          str += appendMd(children[i].children)
      }
  }
  return str
}

function operate(children: any, idx: number, type: number) {
  let current = children[idx]
  if (type == 1) {
      // 在上方新增同级
      children.splice(idx, 0, {
          level: current.level,
          name: '请输入文字',
          children: []
      })
  } else if (type == 2) {
      // 在下方新增下属
      current.children.splice(0, 0, {
          level: current.level + 1,
          name: '请输入文字',
          children: []
      })
  } else if (type == 3) {
      // 删除
      children.splice(idx, 1)
  }
  update()
}

function update() {
  let outlineMd = ''
  outlineMd += '# ' + props.outlineTree.name + '\n'
  outlineMd += appendMd(props.outlineTree.children)
  version.value++
  $emit('update', outlineMd)
}

</script>

<style scoped>
.outline_div {
  padding: 15px;
  display: flex;
  /*align-items: center;*/
  flex-direction: column;
}
.outline_div input {
  border: none;
  outline: none;
}
.outline_div input:hover {
  border-width: 1px;
  border-style: inset;
  border-image: initial;
  border-color: light-dark(rgb(118, 118, 118), rgb(133, 133, 133));
}
.operate_div {
  display: none;
}
.subject_div {
  text-align: center;
  margin-bottom: 0.5em;
  color: #3c3c3c;
  font-weight: 600;
  font-size: 28px;
  line-height: 1.2;
}
.chapter_div {
  display: flex;
  align-items: center;
}
.chapter_div:hover > .operate_div {
  display: inline-block;
}
.chapter_title {
  padding-left: .5rem;
  padding-right: .5rem;
  border-radius: .375rem;
  margin-right: .8rem;
  border-style: dashed;
  width: max-content;
  display: inline-block;
  color: #624fc4;
  background-color: #e0dcf3;
}
.chapter_name, .page_name, .title_name {
  margin-right: 0.8rem;
  text-align: left;
  display: inline-block;
}
.chapter_name, .page_name {
  font-weight: bold;
}
.page_div {
  display: flex;
  align-items: center;
  padding-left: 1rem;
}
.page_div:hover > .operate_div {
  display: inline-block;
}
.title_div {
  display: flex;
  align-items: center;
  padding-left: 1rem;
}
.title_div:hover > .operate_div {
  display: inline-block;
}
.page_number {
  font-weight: 700;
}
.page_number, .title_number {
  width: 3rem;
  margin-right: .5rem;
  text-align: right;
}
.chapter_div:hover, .page_div:hover, .title_div:hover {
  background-color: #f3f3f3;
}
.operate_div span {
  padding: .2rem .4rem;
  margin-right: .5rem;
  cursor: pointer;
}
</style>
