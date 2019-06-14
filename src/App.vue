<template>
  <div id="app" class="w-full h-full bg-gray-100">
    <div class="container mx-auto pt-16">
      <div class="flex -mx-2 mb-4">
        <div class="w-1/2 px-2">
          <card title="Configuration">
            <p class="mb-2">
              1. Let's enter the data you want to describe.
            </p>

            <input class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" type="text" v-model="text" @select="updateSelection" />

            <p class="mt-6 mb-2">
              2. Add a section by selecting the part you want to describe from the input field.
            </p>

            <div class="flex -mx-2">
              <div class="w-1/2 px-2">
                <input class="appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" type="text" v-model="selection.text" disabled placeholder="Select text to describe" />
              </div>
              <div class="w-1/3 px-2">
                <input class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" type="text" v-model="label" placeholder="Description" />
              </div>
              <div class="w-1/6 px-2">
                <button v-if="!selection.text || !label"
                        class="bg-blue-500 text-white font-bold py-2 px-4 rounded opacity-50 cursor-not-allowed w-full"
                        type="button" >Add</button>

                <button v-if="selection.text && label"
                        class="w-full text-center bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 rounded focus:outline-none focus:shadow-outline"
                        type="button"
                        @click="addSection">Add</button>
              </div>
            </div>

            <p class="mt-6 mb-2">
              3. Select the character set and indentation type.
            </p>

            <div class="flex -mx-2">
              <div class="relative w-1/2 mx-2">
                <select class="block appearance-none w-full bg-white border border-gray-400 hover:border-gray-500 px-4 py-2 pr-8 rounded shadow leading-tight focus:outline-none focus:shadow-outline" v-model="characterSet">
                  <option value="0">Thin (default)</option>
                  <option value="1">Thick</option>
                  <option value="2">Mixed</option>
                  <option value="3">Mixed alt</option>
                </select>
                <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700">
                  <svg class="fill-current h-4 w-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><path d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z"/></svg>
                </div>
              </div>

              <div class="relative w-1/2 mx-2">
                <select class="block appearance-none w-full bg-white border border-gray-400 hover:border-gray-500 px-4 py-2 pr-8 rounded shadow leading-tight focus:outline-none focus:shadow-outline" v-model="indentation">
                  <option value="1">Indent Labels (default)</option>
                  <option value="0">Inline Labels</option>
                </select>
                <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700">
                  <svg class="fill-current h-4 w-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><path d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z"/></svg>
                </div>
              </div>
            </div>
          </card>
        </div>

        <div class="w-1/2 px-2">
          <card title="Result">
            <pre class="bg-gray-100 rounded w-full px-6 py-4 font-mono" @mouseup="updateSelection" ref="result">{{ text + '\n' + description }}</pre>
          </card>
          <card title="Sections">
            <div v-for="(section, index) in sections" v-bind:key="'section_' + index">
              <span>{{ section }}</span>
              <div class="float-right text-gray-300 hover:text-red-600 cursor-pointer font-bold" @click="sections.splice(index, 1)">Delete</div>
            </div>

            <p v-if="sections.length === 0" class="text-gray-500">No sections created yet.</p>
          </card>
        </div>
      </div>

      <div class="m-auto text-center mt-16">
        <a href="https://github.com/Bw2801/unicode-descriptor" class="text-gray-400 hover:text-gray-800">
          <svg class="m-auto fill-current" height="32" viewBox="0 0 16 16" version="1.1" width="32" aria-hidden="true"><path fill-rule="evenodd" d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"></path></svg>
        </a>

        <p class="text-gray-500 mt-2 text-sm">&copy; 2019 Benedikt Wüller</p>
      </div>
    </div>
  </div>
</template>

<script>
import Card from "./components/Card";
import {UnicodeDescriptor} from "unicode-descriptor/src/descriptor";

const characterSets = [
  { hl: '─', vl: '│', ctl: '┘', ctr: '└', cross: '┼', tt: '┴', tb: '┬', tr: '├', tl: '┤' },
  { hl: '═', vl: '║', ctl: '╝', ctr: '╚', cross: '╬', tt: '╩', tb: '╦', tr: '╠', tl: '╣' },
  { hl: '═', vl: '│', ctl: '╛', ctr: '╘', cross: '╪', tt: '╧', tb: '╤', tr: '╞', tl: '╡' },
  { hl: '─', vl: '║', ctl: '╜', ctr: '╙', cross: '╫', tt: '╨', tb: '╥', tr: '╟', tl: '╢' },
];

export default {
  name: 'app',

  components: {Card},

  data() {
    return {
      text: 'Lorem ipsum dolor sit amet',
      selection: {
        from: 0,
        to: 0,
        text: null,
      },
      label: null,
      sections: [],
      description: '',
      characterSet: 0,
      indentation: '1',
    };
  },

  watch: {
    sections () { this.refreshDescriptor() },
    characterSet () { this.refreshDescriptor() },
    indentation () { this.refreshDescriptor() },
  },

  methods: {
    updateSelection () {
      let text = window.getSelection().toString();
      const from = this.text.indexOf(text);
      const to = from + text.length;

      if (from < 0) text = null;
      this.selection = { text, from, to };
    },

    addSection () {
      this.sections.push({
        from: this.selection.from,
        to: this.selection.to,
        label: this.label,
      });


      this.label = null;
      this.selection = {
        from: 0,
        to: 0,
        text: null,
      };
    },

    refreshDescriptor () {
      const config = {
        characters: characterSets[this.characterSet],
        indentLabels: this.indentation === '1',
      };

      const descriptor = new UnicodeDescriptor(config);
      this.sections.forEach((section) => {
        descriptor.addSection(section.from, section.to - section.from, section.label);
      });
      this.description = descriptor.toString();
    }
  }
}
</script>

<style lang="scss">
  @import "~tailwindcss/base";
  @import "~tailwindcss/components";
  @import "~tailwindcss/utilities";
</style>
