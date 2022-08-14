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
  <main class="main">
    <div class="sidebar container">
      <button v-if="downloading" aria-busy="true">Please Wait...</button>
      <button v-else @click="downloadImage()">Download</button>
      <br />
      <article>
        <label for="blobBorderRadius"
          >Blob Border Radius
          <input
            type="range"
            min="0"
            max="1"
            step="0.01"
            v-model="settings.blobBorderRadius"
            id="blobBorderRadius"
          />
        </label>
        <label for="imageSize"
          >Image Size
          <input
            type="range"
            min="0.5"
            max="50"
            step="0.5"
            v-model="settings.imageSize"
            id="imageSize"
          />
        </label>
        <label for="showLineNumbers"
          >Show Line Numbers
          <input
            role="switch"
            type="checkbox"
            v-model="settings.showLineNumbers"
            id="showLineNumbers"
          />
        </label>
      </article>
    </div>
    <div class="edit container">
      <div class="codeEditWrapper">
        <CodeEditor
          v-model="input"
          :hide_header="true"
          :language="settings.language"
          width="100%"
          font_size="12px"
          :wrap_code="true"
        />
      </div>
      <br />
      <pre
        class="output-container"
      ><code class="hljs output" id="output"></code></pre>
    </div>
  </main>
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
      downloading: false,
      input: 'const enter = yourText("here")',
      settings: {
        theme: "atom-one-dark",
        language: "javascript",
        blobBorderRadius: 1,
        imageSize: 16,
        filterSize: 0,
        showLineNumbers: true,
      },
    };
  },
  watch: {
    input(newer) {
      if (newer.length > 2000) {
        this.input = newer.slice(0, 2000);
      }
      this.updatePreview();
    },
    settings: {
      handler() {
        this.updatePreview();
      },
      deep: true,
    },
  },
  mounted() {
    document.getElementById("theme").onload = async function () {
      this.updatePreview();
    }.bind(this);
  },
  methods: {
    highlight(code, language) {
      return hljs.highlight(code, { language: language }).value;
    },
    updatePreview() {
      const outputElement = document.getElementById("output");
      outputElement.innerHTML = this.highlight(
        this.input,
        this.settings.language
      );
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

      if (this.settings.showLineNumbers) {
        output =
          "<span class='hljs-comment line-number'>&#8203;</span>" +
          output
            .split("\n")
            .join("\n<span class='hljs-comment line-number'>&#8203;</span>");
      }

      outputElement.innerHTML = output;
      outputElement.style.fontSize = this.settings.imageSize + "px";
      outputElement.querySelectorAll("span").forEach((e) => {
        e.style.backgroundColor = window.getComputedStyle(e).color;
        e.style.borderRadius = this.settings.blobBorderRadius + "em";
      });
    },
    async downloadImage() {
      const element = document.getElementById("output");
      this.downloading = true;
      let canvas = await html2canvas(element, { backgroundColor: null });
      this.downloading = false;
      const a = document.createElement("a");
      a.href = canvas.toDataURL();
      a.download = "blobbed editor.png";
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
    },
  },
};
</script>

<style>
@import "@/assets/pico.css";
html {
  height: 100vh;
}
:root {
  --font-size: 12px;
}
.main {
  height: 100vh;
  display: flex;
  flex-wrap: nowrap;
}
.sidebar {
  height: 100vh;
  overflow-y: auto;
  width: 300px;
  min-width: auto;
  max-width: auto;
  border-right: 1px solid var(--muted-border-color);
}
.edit {
  width: calc(100vw - 300px);
  height: 100vh;
  display: flex;
  justify-content: space-around;
  flex-direction: column;
  min-width: auto;
  max-width: auto;
}
.codeEditWrapper {
  overflow: auto;
  height: 45%;
  border-radius: 12px;
}
.output-container {
  overflow: auto;
  width: 100%;
  height: 45%;
  background-color: transparent;
  display: flex;
  justify-content: center;
}
.container {
  padding: 1em;
}
@media (max-width: 576px) {
  .main {
    flex-wrap: wrap;
  }
  .sidebar {
    width: 100%;
    border: 0;
    height: auto;
  }
  .edit {
    width: 100%;
    height: auto;
  }
  .output-container {
    height: auto;
  }
}

label {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
[type="range"] {
  width: 50% !important;
}
[type="range"]::-ms-track {
  width: 50% !important;
}
[type="range"]::-moz-range-track {
  width: 50% !important;
}
[type="range"]::-webkit-slider-runnable-track {
  width: 50% !important;
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
  height: fit-content;
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  cursor: default;
}
</style>
