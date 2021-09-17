<template>
  <section class="w-4/5 py-8 mx-auto text-gray-600 dark:text-gray-300">
    <section>
      <header>
        <h2 class="text-xl">BP-ColorParser</h2>
        <p>Parse your color codes here, and render BP color codes &amp; unity rich text codes.</p>
      </header>
      <aside class="pt-4 text-gray-400">
        <h3 class="text-lg">Regex matcher</h3>
        <p>Match $1 must be the hex color value and $2 the content.</p>
        <input type="text" class="w-full px-3 py-2 mt-2 bg-gray-200 rounded-md shadow-inner dark:bg-gray-700 md:w-auto" v-model="regexInput" />
      </aside>
    </section>
    <section class="flex flex-col py-8 md:flex-row">
      <main class="relative pb-4 md:pr-4 md:w-1/2 md:pb-0">
        <span class="absolute right-10 top-2 dark:text-gray-500">edit</span>
        <textarea class="w-full h-full px-3 py-2 font-mono bg-gray-200 rounded-md shadow-inner dark:bg-gray-700" v-model="input"></textarea>
      </main>
      <aside class="relative pt-2 pl-4 font-mono whitespace-pre-line bg-gray-200 rounded-md shadow-inner md:w-1/2 dark:bg-gray-700">
        <span class="absolute right-6 top-2 dark:text-gray-500">preview</span>
        <p v-html="renderedHtml"></p>
      </aside>
    </section>
    <LayoutFooter />
  </section>
</template>

<script lang="ts">
import Vue from 'vue';

export default Vue.extend({
  data() {
    return {
      regexInput: '<color=(.*?)>((.|\\n)*?)</color>',
      input: '',
      colorCodes: {
        '0': '#000000',
        '1': '#0000aa',
        '2': '#00aa00',
        '3': '#00aaaa',
        '4': '#aa0000',
        '5': '#aa00aa',
        '6': '#ffaa00',
        '7': '#aaaaaa',
        '8': '#555555',
        '9': '#5555ff',
        a: '#55ff55',
        b: '#55ffff',
        c: '#ff5555',
        d: '#ff55ff',
        e: '#ffff55',
        f: '#ffffff',
      },
    };
  },
  methods: {
    parseColors(str: string) {
      let result = '';

      for (const line of str.split('&')) {
        if (!line) {
          continue;
        }
        // @ts-ignore
        const hex = this.colorCodes[line[0]];
        if (!hex) {
          result += line;
          continue;
        }
        result += `<color=${hex}>${line.slice(1)}</color>`;
      }
      return result;
    },
  },
  computed: {
    regex(): RegExp {
      return new RegExp(this.regexInput as string, 'gm');
    },
    renderedHtml() {
      let value = this.input as string;

      value = this.parseColors(value);
      value = value.replaceAll(this.regex, '<span style="color: $1">$2</span>');

      return value;
    },
  },
  watch: {
    input(newVal) {
      history.replaceState({}, '', `${location.origin}${location.pathname}#${Buffer.from(newVal, 'utf-8').toString('base64')}`);
    },
  },
  mounted() {
    if (!this.$route.hash) {
      this.input = Object.entries(this.colorCodes)
        .map(([key, value]) => `&${key}^${key} @ ${value} - hello world`)
        .join('\n');
      return;
    }
    this.input = Buffer.from(this.$route.hash, 'base64').toString();
  },
});
</script>