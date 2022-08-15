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
      <h1>Blobbed Editor</h1>
      <button v-if="downloading" aria-busy="true">Please Wait...</button>
      <button v-else @click="downloadImage()">Download</button>
      <h6 class="subheading">Settings</h6>
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
      <h6 class="subheading">Highlighting</h6>
      <article>
        <SearchLang />
      </article>
      <h6 class="subheading">Data</h6>
      <article>
        <span>Characters Remaining: {{ charLimit }}</span
        ><br />
        <span>Image Resolution: {{ resolution }}</span>
      </article>
    </div>
    <div class="edit container">
      <div class="codeEditWrapper">
        <CodeEditor
          v-model="input"
          :hide_header="true"
          :language="settings.language"
          width="100%"
          font_size="14px"
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
import SearchLang from "./components/SearchLang.vue";

export default {
  components: {
    CodeEditor,
    SearchLang,
  },
  data() {
    return {
      downloading: false,
      input: 'const enter = yourText("here")',
      resolution: "",
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
  computed: {
    charLimit() {
      return 2000 - this.input.length;
    },
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
            .join(
              "\n<span class='indent' style='padding:0;margin-left:0;margin-right:0'></span>"
            );
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
          "<span class='hljs-comment line-number'> </span>" +
          output
            .split("\n")
            .join("\n<span class='hljs-comment line-number'> </span>");
      }
      outputElement.innerHTML = output;
      outputElement.style.fontSize = this.settings.imageSize + "px";
      outputElement.querySelectorAll("span").forEach((e) => {
        e.style.backgroundColor = window.getComputedStyle(e).color;
        e.style.borderRadius = this.settings.blobBorderRadius + "em";
        e.style.width = (e.innerText.length * 1).toString() + "em";
        e.innerHTML = "&#8203;";
      });
      document.querySelector(".output-container").style.overflow = "visible";
      outputElement.style.overflow = "visible";
      this.resolution =
        outputElement.offsetWidth.toString() +
        "x" +
        outputElement.offsetHeight.toString();
      document.querySelector(".output-container").style.overflow = "auto";
      outputElement.style.overflow = "auto";
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
  --font-size: 14px;
}
.main {
  display: flex;
  flex-wrap: wrap;
}
.sidebar {
  margin-bottom: var(--spacing);
}
.output-container {
  margin-bottom: var(--spacing);
  overflow: auto;
  width: 100%;
  background-color: transparent;
  display: flex;
  justify-content: center;
}
article {
  margin: 0;
}
@media (min-width: 576px) {
  .main {
    height: 100vh;
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
    height: 45%;
  }
  .container {
    padding: 1em;
  }
}
h1 {
  margin-bottom: var(--spacing);
  text-align: center;
}
.subheading {
  margin-bottom: 2px;
}
article {
  margin-bottom: var(--spacing);
  --block-spacing-horizontal: var(--spacing);
}
label {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
[type="range"] {
  width: 50% !important;
  margin: 0 !important;
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
span.indent {
  color: transparent;
}
pre code.output.hljs {
  overflow-x: auto;
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
