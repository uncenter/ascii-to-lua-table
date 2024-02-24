<script setup lang="ts">
import type { HighlighterCore } from 'shiki/core'
import { getHighlighterCore } from 'shiki/core'
import githubLightTheme from 'shiki/themes/github-light.mjs'
import githubDarkTheme from 'shiki/themes/github-dark.mjs'
import luaLang from 'shiki/langs/lua.mjs'

const isDark = useDark()

const input = useStorage(
  'input',
  `  ／l、
（ﾟ､ ｡ ７
  l  ~ヽ
  じしf_,)ノ`,
)
const converted = ref('')
const output = ref('')

const copied = ref(false)
const clipboard = useClipboard()

function asciiToLuaTable(ascii: string) {
  function escape(str: string) {
    return str.replace(/\\/g, '\\\\').replace(/"/g, '\\"')
  }

  let result = ''
  let first = true

  const lines = ascii.split('\n').map(line => escape(line))
  const longestLine = lines.reduce((a, b) => (a.length > b.length ? a : b))

  for (const line of lines) {
    if (first)
      first = false
    else result += ', \n  '

    const padding = ' '.repeat(longestLine.length - line.length)
    const wrapper = ['"', '"']

    result += `${wrapper[0]}${line.length > 0 && line.length < longestLine.length ? line + padding : line}${wrapper[1]}`
  }

  return ascii.length > 0
    ? `local table = {
  ${result}
}`
    : ''
}

async function run() {
  converted.value = asciiToLuaTable(input.value)
  const highlighter = await getHighlighterCore({
    themes: [githubLightTheme, githubDarkTheme],
    langs: [luaLang],
    loadWasm: () => import('shiki/wasm'),
  })

  output.value = highlighter.codeToHtml(converted.value, {
    lang: 'lua',
    theme: isDark.value ? 'github-dark' : 'github-light',
  })
}

function copy() {
  if (converted.value.length === 0)
    return
  clipboard.copy(converted.value)
  copied.value = true
  setTimeout(() => {
    copied.value = false
  }, 1000)
}

watch(
  [input, isDark],
  (n, o) => {
    const changed = o !== n
    if (changed)
      run()
  },
  { immediate: true },
)

if (import.meta.hot) {
  import.meta.hot.accept(() => {
    run()
  })
}
</script>

<template>
  <div h-100vh w-full px4 pt-4>
    <header flex="~ justify-between">
      <h1>ASCII to Lua Table Converter</h1>

      <div flex="~ row gap-2">
        <a
          border="~ base rounded" p2 hover="bg-active" aria-label="GitHub repository"
          href="https://github.com/uncenter/ascii-to-lua-table" target="_blank"
        >
          <div i-carbon-logo-github />
        </a>
        <button
          border="~ base rounded" p2 hover="bg-active"
          :aria-label="`Toggle theme to ${isDark ? 'light' : 'dark'}`" @click="isDark = !isDark"
        >
          <div dark:i-carbon-moon i-carbon-sun />
        </button>
      </div>
    </header>
    <div flex="~ col md:row gap-4" font-mono p4>
      <textarea
        v-model="input" border="~ base rounded" placeholder="ASCII..." px3 py1 :class="`${isDark ? 'bg-[#24292F] ' : ''
        }w-full md:w-[50vw] h-[40vh] md:h-[70vh]`" spellcheck="false" autocorrect="off" autocapitalize="off"
      />
      <div
        border="~ base rounded" px3 py1 relative of-auto :class="`${isDark ? 'bg-[#24292F] ' : ''
        }w-full md:w-[50vw] h-[40vh] md:h-[70vh]`"
      >
        <button
          border="~ base rounded" p2 absolute top-2 right-2 :class="converted.length > 0
            ? 'hover:bg-active'
            : 'op-[0.5] cursor-not-allowed'
          " :aria-disabled="converted.length === 0" title="Copy" @click="copy()"
        >
          <div v-if="copied" i-carbon-checkmark />
          <div v-else i-carbon-copy />
        </button>
        <div v-html="output" />
      </div>
    </div>
  </div>
</template>

<style>
:root {
  color-scheme: light;
}

:root.dark {
  color-scheme: dark;
}
</style>
