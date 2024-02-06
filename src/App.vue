<script setup lang="ts">
const isDark = useDark();

const input = useStorage("input", "");
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

    const diff = longestLine.length - line.length;

    let toPush;

    const newline = line.replace(/\\/g, "\\\\").replace(/"/g, '\\"');

    if (line.length < longestLine.length) {
      const padding = " ".repeat(diff);
      toPush = `"${newline}${padding}"`;
    } else {
      toPush = `"${newline}"`;
    }

    result += toPush;
  }

  return `{
  ${result}
}`;
}

async function run() {
  output.value = asciiToLuaTable(input.value);
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
  input,
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
    <div flex="~ row gap-4" p4>
      <textarea
        border="~ base rounded"
        v-model="input"
        placeholder="ASCII..."
        px3
        py1
        class="w-[50vw] h-[70vh]"
      />
      <textarea
        border="~ base rounded"
        v-model="output"
        px3
        py1
        class="w-[50vw] h-[70vh]"
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
