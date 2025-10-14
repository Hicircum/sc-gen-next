<template>
  <div class="p-4">
    <h2>上传 XML 文件并提取 &lt;sc&gt; 标签</h2>
    <div class="upload-area">
      <input type="file" accept=".xml" @change="handleFileChange" :disabled="isLoading" />
      <div v-if="isLoading" class="loading">处理中...</div>
    </div>
    
    <div v-if="resultMessage" class="result-message" :class="{ error: isError }">
      {{ resultMessage }}
    </div>
    
    <div v-if="scItems.length > 0" class="results mt-4">
      <h3>提取结果 ({{ scItems.length }}条)</h3>
      <div v-for="(item, index) in scItems" :key="index" class="result-item">
        <div><strong>文本:</strong> {{ item.text }}</div>
        <div class="attributes">
          <div v-for="(value, key) in getAttributesOnly(item)" :key="key">
            <strong>{{ key }}:</strong> {{ value }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'

interface ScItem {
  text: string;
  [key: string]: string;
}

const isLoading = ref(false)
const resultMessage = ref('')
const isError = ref(false)
const scItems = ref<ScItem[]>([])

const handleFileChange = (event: Event) => {
  const input = event.target as HTMLInputElement
  const file = input.files?.[0]
  if (!file) return
  
  // 重置状态
  isLoading.value = true
  resultMessage.value = ''
  isError.value = false
  scItems.value = []

  const reader = new FileReader()
  reader.onload = (e) => {
    const text = e.target?.result as string
    parseScTags(text)
  }
  reader.onerror = () => {
    isLoading.value = false
    resultMessage.value = '读取文件失败，请重试。'
    isError.value = true
  }
  reader.readAsText(file, 'utf-8')
}

/**
 * 提取所有 <sc> 的内容
 */
const parseScTags = (xmlText: string) => {
  const regex = /<sc\s+([^>]*)>(.*?)<\/sc>/gs
  const matches = Array.from(xmlText.matchAll(regex))
  
  if (matches.length === 0) {
    isLoading.value = false
    resultMessage.value = '未在文件中找到任何 <sc> 标签。'
    isError.value = true
    return
  }

  for (const match of matches) {
    const attrs = match[1]
    const innerText = match[2]

    // 提取属性键值对
    const attrRegex = /(\w+)="(.*?)"/g
    const attrObj: Record<string, string> = {}
    if (typeof attrs === 'string') {
      for (const a of attrs.matchAll(attrRegex)) {
        if (a[1] !== undefined && a[2] !== undefined) {
          attrObj[a[1]] = a[2]
        }
      }
    }

    scItems.value.push({
      ...attrObj,
      text: (innerText ?? '').trim(),
    })
  }
  
  isLoading.value = false
  resultMessage.value = `成功提取 ${scItems.value.length} 个 <sc> 标签内容。`
}

/**
 * 从项目中提取属性，排除text字段
 */
const getAttributesOnly = (item: ScItem) => {
  const { text, ...attributes } = item
  return attributes
}
</script>

<style scoped>
h2 {
  margin-bottom: 12px;
}

.upload-area {
  display: flex;
  align-items: center;
  gap: 12px;
}

.loading {
  display: inline-flex;
  align-items: center;
  color: #666;
}

.loading::after {
  content: "";
  width: 16px;
  height: 16px;
  margin-left: 8px;
  border: 2px solid #ccc;
  border-top-color: #007bff;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.result-message {
  margin-top: 16px;
  padding: 8px 12px;
  border-radius: 4px;
  background-color: #e6f7ff;
  color: #0070cc;
}

.result-message.error {
  background-color: #fff2f0;
  color: #ff4d4f;
}

.results {
  border-top: 1px solid #eee;
  padding-top: 16px;
}

.result-item {
  margin-bottom: 16px;
  padding: 12px;
  border: 1px solid #eaeaea;
  border-radius: 4px;
  background-color: #fafafa;
}

.attributes {
  margin-top: 8px;
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.attributes > div {
  background: #f0f0f0;
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 0.9em;
}
</style>
