<template>
  <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
    <h3 class="text-sm font-bold text-slate-400 mb-3">正则表达式输入</h3>
    <div class="relative">
      <span class="absolute left-3 top-2 text-cyan-500 font-bold text-lg">/</span>
      <input
        v-model="localPattern"
        @input="onInput"
        @keyup.enter="execute"
        type="text"
        placeholder="输入正则表达式..."
        class="w-full bg-slate-900 border border-slate-600 rounded-lg pl-8 pr-12 py-2 text-cyan-400 font-mono text-sm focus:outline-none focus:border-cyan-500"
      />
      <span class="absolute right-3 top-2 text-cyan-500 font-bold text-lg">/g</span>
    </div>
    <div v-if="store.error" class="mt-2 text-red-400 text-sm">⚠ {{ store.error }}</div>
    <textarea
      v-model="localTestString"
      @input="onTestInput"
      placeholder="输入测试字符串..."
      rows="3"
      class="w-full mt-3 bg-slate-900 border border-slate-600 rounded-lg px-3 py-2 text-slate-200 font-mono text-sm focus:outline-none focus:border-cyan-500 resize-none"
    ></textarea>
    <button @click="execute" class="w-full mt-3 px-4 py-2 bg-cyan-600 hover:bg-cyan-500 rounded-lg text-white font-bold text-sm">执行匹配</button>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, onBeforeUnmount } from 'vue'
import { useRegexStore } from '../store/regex'

const store = useRegexStore()
// 本地仅作为输入防抖缓冲，真实状态以 store 为唯一数据源
const localPattern = ref(store.pattern)
const localTestString = ref(store.testString)

let patternTimer: ReturnType<typeof setTimeout> | undefined
let testTimer: ReturnType<typeof setTimeout> | undefined

function cancelPatternTimer() {
  if (patternTimer) { clearTimeout(patternTimer); patternTimer = undefined }
}
function cancelTestTimer() {
  if (testTimer) { clearTimeout(testTimer); testTimer = undefined }
}

// store 被外部修改（如套用模板、状态回退）时同步到编辑区，
// 并取消未执行的防抖，防止旧值随后覆盖新状态造成乱序
watch(() => store.pattern, (v) => {
  cancelPatternTimer()
  localPattern.value = v
}, { flush: 'sync' })
watch(() => store.testString, (v) => {
  cancelTestTimer()
  localTestString.value = v
}, { flush: 'sync' })

function onInput() {
  cancelPatternTimer()
  patternTimer = setTimeout(() => { store.setPattern(localPattern.value) }, 300)
}
function onTestInput() {
  cancelTestTimer()
  testTimer = setTimeout(() => { store.setTestString(localTestString.value) }, 300)
}
function execute() {
  cancelPatternTimer()
  cancelTestTimer()
  store.setPattern(localPattern.value)
  store.setTestString(localTestString.value)
}
onBeforeUnmount(() => { cancelPatternTimer(); cancelTestTimer() })
</script>
