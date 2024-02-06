<script setup lang="ts">
import type { HighlighterCore } from "shiki/core";
import { getHighlighterCore } from "shiki/core";
import githubLightTheme from "shiki/themes/github-light.mjs";
import githubDarkTheme from "shiki/themes/github-dark.mjs";
import luaLang from "shiki/langs/lua.mjs";

const isDark = useDark();

const input = useStorage(
  "input",
  `  ／l、
（ﾟ､ ｡ ７
  l  ~ヽ
  じしf_,)ノ`,
);
const output = ref("");

const copied = ref(false);
const clipboard = useClipboard();

function asciiToLuaTable(ascii) {
  let result = "";
  let first = true;

  const lines = ascii.split("\n");
  const longestLine = lines.reduce((a, b) => (a.length > b.length ? a : b));

  for (const line of lines) {
    if (first) {
      first = false;
    } else {
      result += ", \n  ";
    }

    const padding = " ".repeat(longestLine.length - line.length);
    const wrapper = ["[[", "]]"];

    result += `${wrapper[0]}${line.length < longestLine.length ? line + padding : line}${wrapper[1]}`;
  }

  return ascii.length > 0
    ? `local table = {
  ${result}
}`
    : "";
}

async function run() {
  const highlighter = await getHighlighterCore({
    themes: [githubLightTheme, githubDarkTheme],
    langs: [luaLang],
    loadWasm: () => import("shiki/wasm"),
  });

  const result = highlighter.codeToHtml(asciiToLuaTable(input.value), {
    lang: "lua",
    theme: isDark.value ? "github-dark" : "github-light",
  });
  console.log({ isDark });
  output.value = result;
}

function clear() {
  input.value = "";
}

function copy() {
  clipboard.copy(output.value);
  copied.value = true;
  setTimeout(() => {
    copied.value = false;
  }, 1000);
}

watch(
  [input, isDark],
  (n, o) => {
    const changed = o !== n;
    if (changed) run();
  },
  { immediate: true },
);

if (import.meta.hot) {
  import.meta.hot.accept(() => {
    run();
  });
}
</script>

<template>
  <div h-100vh w-full px4 pt-4>
    <header flex="~ justify-between">
      <h1>ASCII to Lua Table Converter</h1>

      <div flex="~ row gap-2">
        <a
          border="~ base rounded"
          p2
          hover="bg-active"
          href="https://github.com/uncenter/ascii-to-lua-table"
          target="_blank"
        >
          <div i-carbon-logo-github />
        </a>
        <button
          border="~ base rounded"
          p2
          hover="bg-active"
          title="Clear"
          @click="clear()"
        >
          <div i-carbon-clean />
        </button>
        <button
          border="~ base rounded"
          p2
          hover="bg-active"
          @click="isDark = !isDark"
        >
          <div dark:i-carbon-moon i-carbon-sun />
        </button>
      </div>
    </header>
    <div flex="~ col md:row gap-4" font-mono p4>
      <textarea
        border="~ base rounded"
        v-model="input"
        placeholder="ASCII..."
        px3
        py1
        :class="
          (isDark ? 'bg-[#24292F] ' : '') +
          'w-full md:w-[50vw] h-[40vh] md:h-[70vh]'
        "
      />
      <div
        border="~ base rounded"
        v-html="output"
        px3
        py1
        :class="
          (isDark ? 'bg-[#24292F] ' : '') +
          'w-full md:w-[50vw] h-[40vh] md:h-[70vh]'
        "
      />
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
