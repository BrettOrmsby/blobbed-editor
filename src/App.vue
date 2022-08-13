<template>
  <link
    id="theme"
    rel="stylesheet"
    :href="
      'https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.6.0/styles/' +
      settings.theme +
      '.min.css'
    "
  />
  <a :href="mainSrc" download="Blobbed Editor">Download</a>
  <CodeEditor
    v-model="input"
    :hide_header="true"
    :language="settings.language"
  />
  <label for="blobBorderRadius"
    >Blob Border Radius
    <input
      type="range"
      min="0"
      max="0.5"
      step="0.01"
      v-model="settings.blobBorderRadius"
      id="blobBorderRadius"
    />{{ settings.blobBorderRadius }}
  </label>
  <label for="blobSize"
    >Blob Size
    <input
      type="range"
      min="1"
      max="50"
      step="1"
      v-model="settings.blobSize"
      id="blobSize"
    />{{ settings.blobSize }}
  </label>
  <pre><code class="hljs" v-html="highlightedCode"></code></pre>
  <img :src="mainSrc" style="width: 100%" />

  <pre
    style="overflow: hidden; height: 0"
  ><code class="hljs" id="output" style="display:inline-block"></code></pre>
</template>

<script>
import hljs from "highlight.js";
import html2canvas from "html2canvas";
import CodeEditor from "./components/CodeEditor.vue";

export default {
  components: {
    CodeEditor,
  },
  data() {
    return {
      input: "",
      mainSrc: "",
      settings: {
        theme: "github-dark",
        language: "javascript",
        blobBorderRadius: 0.5,
        blobSize: 16,
        filterSize: 0,
      },
    };
  },
  computed: {
    highlightedCode() {
      return this.highlight(this.input, this.settings.language);
    },
  },
  watch: {
    async input(newer) {
      if (newer.length > 2000) {
        this.input = newer.slice(0, 2000);
      }
      this.mainSrc = await this.toBlobbedImage(this.input, this.settings);
    },
    settings: {
      async handler(newer) {
        this.mainSrc = await this.toBlobbedImage(this.input, newer);
      },
      deep: true,
    },
  },
  mounted() {
    this.input = "var blob = new Blob()";
    document.getElementById("theme").onload = async function () {
      this.mainSrc = await this.toBlobbedImage(this.input, this.settings);
    }.bind(this);
  },
  methods: {
    highlight(code, language) {
      return hljs.highlight(code, { language: language }).value;
    },
    async toBlobbedImage(code, settings) {
      const outputElement = document.getElementById("output");
      outputElement.innerHTML = this.highlight(code, settings.language);
      let output = "";
      traverse(outputElement, "");

      function traverse(node, type) {
        if (node.nodeType === Node.TEXT_NODE) {
          const text = node.data;
          output += text
            .split("\n")
            .map((line) => {
              if (!line) {
                return "";
              }
              return line
                .split(/ {2}|\t/g)
                .map((section) => {
                  section = section.trim();
                  if (!section) {
                    return "";
                  }
                  return `<span class="${type}">${section.replace(
                    /./g,
                    " "
                  )}</span>`;
                })
                .join("<span class='indent'>  </span>");
            })
            .join("\n");
        } else {
          for (let child of node.childNodes) {
            traverse(
              child,
              [...node.classList]
                .filter((e) => (e == "hljs" ? false : true))
                .join(" ")
            );
          }
        }
      }
      outputElement.innerHTML = output;

      outputElement.querySelectorAll("span").forEach((e) => {
        e.style.backgroundColor = window.getComputedStyle(e).color;
        console.log(settings.blobBorderRadius + "em");
        e.style.borderRadius = settings.blobBorderRadius + "em";
        e.style.fontSize = settings.blobSize + "px";
      });

      let canvas = await html2canvas(outputElement, { backgroundColor: null });
      return canvas.toDataURL();
    },
  },
};
</script>

<style>
#output span {
  font-size: 16px;
  padding: 0 0.5em;
  margin: 0.25em;
  display: inline-block;
}
#output .indent {
  color: transparent;
}
</style>
