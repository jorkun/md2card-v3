<template>
  <el-container class="min-h-screen p-4 bg-gray-50">
    <el-main>
      <div v-if="!started" class="flex flex-col items-center justify-center min-h-screen text-center">
        <h1 class="text-4xl font-bold mb-4">📄 Markdown to Card</h1>
        <p class="text-lg text-gray-600 mb-6 max-w-xl">
          Create beautiful card-style previews from your Markdown text.
        </p>
        <el-button type="primary" size="large" @click="started = true">Start Writing</el-button>
      </div>

      <el-row v-else :gutter="20">
        <el-col :span="12">
          <div class="mb-2 flex flex-wrap gap-2">
            <el-button @click="insert('bold')">Bold</el-button>
            <el-button @click="insert('italic')">Italic</el-button>
            <el-button @click="insert('code')">Code</el-button>
            <el-button @click="insert('h1')">H1</el-button>
            <el-button @click="insert('ul')">UL</el-button>
            <el-button @click="insert('ol')">OL</el-button>
          </div>
          <div id="monaco" class="h-96 border rounded" />
          <div class="mt-2 flex gap-2 flex-wrap">
            <el-button @click="preview = !preview">{{ preview ? '隐藏预览' : '显示预览' }}</el-button>
            <el-button type="success" @click="copyMarkdown">复制 Markdown</el-button>
            <el-button @click="downloadMarkdown">下载 .md</el-button>
            <el-button type="warning" @click="exportImage">导出图片</el-button>
          </div>
        </el-col>
        <el-col :span="12" v-if="preview">
          <div ref="previewRef" class="prose p-4 bg-white rounded shadow" v-html="renderedMarkdown" />
        </el-col>
      </el-row>
    </el-main>
  </el-container>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'
import * as monaco from 'monaco-editor'
import { marked } from 'marked'
import html2canvas from 'html2canvas'

const markdown = ref("# MD2Card" +
  "\n" +
  "## 介绍\n" +
  "\n" +
  "MD2Card 是一个 markdown 转知识卡片工具，可以让你用 Markdown 制作优雅的图文海报。 🌟\n" +
  "\n" +
  "![](https://picsum.photos/600/300)\n" +
  "\n" +
  "## 它的主要功能：\n" +
  "\n" +
  "1. 将 Markdown 转化为**知识卡片**\n" +
  "2. 多种主题风格任你选择\n" +
  "3. 长文自动拆分，或者根据 markdown `---` 横线拆分\n" +
  "4. 可以复制图片到`剪贴板`，或者下载为`PNG`、`SVG`图片\n" +
  "5. 所见即所得\n" +
  "6. 免费\n" +
  "\n")
const preview = ref(true)
const started = ref(false)
const previewRef = ref<HTMLElement | null>(null)
let editor: monaco.editor.IStandaloneCodeEditor

watch(started, (v) => {
  console.log('started', v)
  requestAnimationFrame(() => {
    if (v) {
      editor = monaco.editor.create(document.getElementById('monaco')!, {
      value: markdown.value,
      language: 'markdown',
      theme: 'vs-dark',
      automaticLayout: true
    })

    editor.onDidChangeModelContent(() => {
      markdown.value = editor.getValue()
    })    }
  })
})

const renderedMarkdown = computed(() => marked(markdown.value))

function insert(type: string) {
  const insertMap: Record<string, string> = {
    bold: '**bold**',
    italic: '*',
    code: '`code`',
    h1: '# Heading',
    ul: '- List item',
    ol: '1. List item'
  }
  const text = insertMap[type] || ''
  const selection = editor?.getSelection?.()
  const id = { major: 1, minor: 1 }
  const op = selection ? {
    identifier: id,
    range: selection!,
    text: text,
    forceMoveMarkers: true
  } : {
    identifier: id,
    range: new monaco.Range(1, 1, 1, 1),
    text: text,
    forceMoveMarkers: true
  }
  editor.executeEdits('insert-text', [op])
  editor.focus()
}

function copyMarkdown() {
  navigator.clipboard.writeText(markdown.value)
}

function downloadMarkdown() {
  const blob = new Blob([markdown.value], { type: 'text/markdown' })
  const link = document.createElement('a')
  link.download = 'markdown.md'
  link.href = URL.createObjectURL(blob)
  link.click()
}

async function exportImage() {
  if (!previewRef.value) return
  const canvas = await html2canvas(previewRef.value)
  const link = document.createElement('a')
  link.download = 'preview.png'
  link.href = canvas.toDataURL()
  link.click()
}
</script>

<style scoped>
#monaco {
  height: 400px;
}
</style>
