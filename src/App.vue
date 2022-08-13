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
  <main class="container">
    <div class="grid">
      <div>
        <CodeEditor
          v-model="input"
          :hide_header="true"
          :language="settings.language"
          width="100%"
        />
        <br />
        <a
          :href="mainSrc"
          download="Blobbed Editor"
          role="button"
          style="display: block"
          >Download</a
        >
        <br />
        <details>
          <summary role="button">Settings</summary>
          <article>
            <label for="blobBorderRadius"
              >Blob Border Radius:
              <code>{{ Math.round(settings.blobBorderRadius * 100) }}%</code>
              <input
                type="range"
                min="0"
                max="1"
                step="0.01"
                v-model="settings.blobBorderRadius"
                id="blobBorderRadius"
              />
            </label>
            <label for="imageQuality"
              >Image Quality:
              <code>{{ Math.round(settings.imageQuality * 2) }}</code>
              <input
                type="range"
                min="1"
                max="50"
                step="0.5"
                v-model="settings.imageQuality"
                id="imageQuality"
              />
            </label>
            <label for="showLineNumbers">
              <input
                type="checkbox"
                v-model="settings.showLineNumbers"
                id="showLineNumbers"
              />
              Show Line Numbers
            </label>
          </article>
        </details>
      </div>
      <div>
        <h2>Image</h2>
        <img :src="mainSrc" style="width: 100%" />
      </div>
    </div>
  </main>
  <pre
    style="overflow: hidden; height: 0"
  ><code class="hljs output" id="output"></code></pre>
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
      input: 'const enter = yourText("here")',
      mainSrc: "",
      settings: {
        theme: "atom-one-dark",
        language: "javascript",
        blobBorderRadius: 1,
        imageQuality: 16,
        filterSize: 0,
        showLineNumbers: true,
      },
    };
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

      if (settings.showLineNumbers) {
        output =
          "<span class='hljs-comment line-number'>&#8203;</span>" +
          output
            .split("\n")
            .join("\n<span class='hljs-comment line-number'>&#8203;</span>");
      }

      outputElement.innerHTML = output;
      outputElement.style.fontSize = settings.imageQuality + "px";
      outputElement.querySelectorAll("span").forEach((e) => {
        e.style.backgroundColor = window.getComputedStyle(e).color;
        console.log(settings.blobBorderRadius + "em");
        e.style.borderRadius = settings.blobBorderRadius + "em";
      });

      let canvas = await html2canvas(outputElement, { backgroundColor: null });
      return canvas.toDataURL();
    },
  },
};
</script>

<style>
@import "@/assets/pico.css";
h2 {
  text-align: center;
  margin-bottom: 0;
}
.output span {
  padding: 0 1em;
  margin: 0.25em;
  display: inline-block;
}
span.line-number {
  margin-right: 0.75em;
}
.output .indent {
  color: transparent;
}
pre code.output.hljs {
  display: inline-block;
  padding: 2em;
  font-size: 16px;
  border-radius: 1em;
}
</style>
