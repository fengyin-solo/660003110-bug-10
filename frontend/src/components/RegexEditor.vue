<template>
  <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
    <h3 class="text-sm font-bold text-slate-400 mb-3">正则表达式输入</h3>
    <div class="relative">
      <span class="absolute left-3 top-2 text-cyan-500 font-bold text-lg">/</span>
      <input
        v-model="patternInput"
        @keyup.enter="execute"
        type="text"
        placeholder="输入正则表达式..."
        class="w-full bg-slate-900 border border-slate-600 rounded-lg pl-8 pr-12 py-2 text-cyan-400 font-mono text-sm focus:outline-none focus:border-cyan-500"
      />
      <span class="absolute right-3 top-2 text-cyan-500 font-bold text-lg">/g</span>
    </div>
    <div v-if="store.error" class="mt-2 text-red-400 text-sm">⚠ {{ store.error }}</div>
    <textarea
      v-model="testStringInput"
      placeholder="输入测试字符串..."
      rows="3"
      class="w-full mt-3 bg-slate-900 border border-slate-600 rounded-lg px-3 py-2 text-slate-200 font-mono text-sm focus:outline-none focus:border-cyan-500 resize-none"
    ></textarea>
    <button @click="execute" class="w-full mt-3 px-4 py-2 bg-cyan-600 hover:bg-cyan-500 rounded-lg text-white font-bold text-sm">执行匹配</button>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useRegexStore } from '../store/regex'

const store = useRegexStore()

// 直接读写 store，不保留本地副本，保证卡片选中、输入内容与匹配结果同步；
// 防抖只延迟执行，不会把旧文本回写到 store，避免快速切换模板时结果乱序
let debounceTimer: ReturnType<typeof setTimeout> | undefined
function scheduleExecute() {
  clearTimeout(debounceTimer)
  debounceTimer = setTimeout(() => store.execute(), 300)
}

const patternInput = computed({
  get: () => store.pattern,
  set: (v: string) => { store.pattern = v; scheduleExecute() }
})
const testStringInput = computed({
  get: () => store.testString,
  set: (v: string) => { store.testString = v; scheduleExecute() }
})

function execute() {
  clearTimeout(debounceTimer)
  store.execute()
}
</script>
