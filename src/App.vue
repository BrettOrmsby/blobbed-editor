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
      <hgroup>
        <h1>Blobbed Editor</h1>
        <em>Create blobbed, syntax highlighted, images of your code</em>
      </hgroup>
      <button v-if="downloading" aria-busy="true">Please Wait...</button>
      <button v-else @click="downloadImage()">Download</button>
      <h6 class="subheading">Settings</h6>
      <article>
        <select
          v-model="settings.type"
          style="margin-bottom: calc(var(--spacing) * 0.25)"
        >
          <option value="regular" selected>Regular View</option>
          <option value="app">App Icon View</option>
          <option value="window">Window View</option>
        </select>
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
        <label for="blobWidth"
          >Blob Width
          <input
            type="range"
            min="0.1"
            max="2"
            step="0.1"
            v-model="settings.blobWidth"
            id="blobWidth"
          />
        </label>
        <label for="blobSpacing"
          >Blob Spacing
          <input
            type="range"
            min="0"
            max="2"
            step="0.05"
            v-model="settings.blobSpacing"
            id="blobSpacing"
          />
        </label>
        <label for="editorPadding"
          >Editor Padding
          <input
            type="range"
            min="0"
            max="5"
            step="0.5"
            v-model="settings.editorPadding"
            id="editorPadding"
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
        <label for="filterSmallerBlobs">
          Filter Smaller Blobs
          <input
            role="switch"
            type="checkbox"
            v-model="settings.filterSmallerBlobs"
            id="filterSmallerBlobs"
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
        <SearchLang @changeLang="changeLang" />
      </article>
      <h6 class="subheading">Data</h6>
      <article>
        <span>Characters Remaining: {{ charLimit }}</span
        ><br />
        <span>Image Resolution: {{ resolution }}</span>
      </article>
      <details>
        <summary role="button">FAQs</summary>
        <article>
          <b>How do indents work?</b>
          <p>
            Indents blobs are made whenever there are 2 spaces or a tab in your
            code. Indents will be made even if they are within other blobs (like
            comments) and will break up the blob.
          </p>
          <b>Why is my image not rendered properly?</b>
          <p>
            The blobbed images might be rendered improperly if the resolution
            size is to big.
          </p>
          <b>What can I use the blobbed images for?</b>
          <ul>
            <li>Favicons</li>
            <li>Logos</li>
            <li>Profile Pictures</li>
            <li>App Icons</li>
            <li>Replacement For Stock Photos</li>
          </ul>
          <b>Why is there a character limit?</b>
          <p style="margin-bottom: 0">
            There is a character limit because the blobbed image is much more
            likely to be rendered improperly if there are too many blobs.
          </p>
        </article>
      </details>
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
//import VueFeather from "vue-feather";

export default {
  components: {
    CodeEditor,
    SearchLang,
    //VueFeather,
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
        blobSpacing: 0.25,
        blobWidth: 1,
        editorPadding: 1,
        filterSmallerBlobs: false,
        showLineNumbers: true,
        type: "regular",
      },
    };
  },
  computed: {
    charLimit() {
      return 2000 - this.input.length;
    },
  },
  watch: {
    //Update the preview on changes to input and settings
    input(newer) {
      //Enforce character limit
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
    // Update the preview when the theme loads to update the blob colours
    document.getElementById("theme").onload = async function () {
      this.updatePreview();
    }.bind(this);
  },
  methods: {
    //Update the language from the search component
    changeLang(lang) {
      this.settings.language = lang;
    },

    //Simple highlight method
    highlight(code, language) {
      return hljs.highlight(code, { language: language }).value;
    },

    //Updates the preview element to configured settings and input
    updatePreview() {
      //Store main element
      const outputElement = document.getElementById("output");
      //Highlight the text
      outputElement.innerHTML = this.highlight(
        this.input,
        this.settings.language
      );

      //Store the output
      let output = "";
      //Repeat through each node to accommodate for nestled spans, indents and newlines
      traverse(outputElement, "");

      function traverse(node, type) {
        if (node.nodeType === Node.TEXT_NODE) {
          //Get the text and split it by lines and indents to format it for the output
          const text = node.data;
          output += text
            .split("\n")
            .map((line) => {
              if (!line) {
                return "";
              }
              //Split line by indents, remove blank elements and combine back with a indent span
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
          //Repeat for each child and allow them to inherit its class if it is a text node
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

      //If showLineNumbers prepend all lines with a line number blob
      if (this.settings.showLineNumbers) {
        output = output.replace(
          /^/gm,
          "<span class='hljs-comment line-number'> </span>"
        );
      }

      //If window type add the window bar
      if (this.settings.type === "window") {
        output =
          '<div class="window-bar"><span></span><span></span><span></span></div>' +
          output;
      }

      //Update the output element
      outputElement.innerHTML = output;
      outputElement.style.fontSize = this.settings.imageSize + "px";
      outputElement.style.padding = this.settings.editorPadding + "em";

      //For window type style the dots and bar
      if (this.settings.type === "window") {
        document.querySelectorAll(".window-bar > span").forEach((e) => {
          e.classList.toggle("hljs-comment");
          e.style.backgroundColor = window.getComputedStyle(e).color;
          e.classList.toggle("hljs-comment");
        });
        const windowBar = document.querySelector(".window-bar");
        windowBar.style.bottom = this.settings.editorPadding + "em";
        windowBar.style.right = this.settings.editorPadding + "em";
      }

      //Style all blobs
      document.querySelectorAll("#output > span").forEach((e) => {
        //Filter non-indent smaller blobs if filterSmallerBlobs
        if (
          this.settings.filterSmallerBlobs &&
          e.innerText.length <= 1 &&
          !e.classList.contains("line-number") &&
          !e.classList.contains("indent")
        ) {
          e.style.display = "none";
          return;
        }
        //Other styles
        e.style.backgroundColor = window.getComputedStyle(e).color;
        e.style.borderRadius = this.settings.blobBorderRadius + "em";
        //Style all non-blank blobs that keep the lines at the same height on new lines with margin
        if (!(e.innerText.length === 0 && e.classList.contains("indent"))) {
          e.style.margin = "0.25em " + this.settings.blobSpacing + "em";
        }
        //Update width based on the amount of characters
        e.style.width =
          (e.innerText.length * this.settings.blobWidth).toString() + "em";
        e.innerHTML = "&#8203;";
      });
      //Update visibility for getting the resolution
      document.querySelector(".output-container").style.overflow = "visible";
      outputElement.style.overflow = "visible";
      //Reset the styles that were changed with the app type
      outputElement.style.width = "fit-content";
      outputElement.style.height = "fit-content";
      outputElement.style.borderRadius = "0px";

      //If app type set image size to a square with padding
      if (this.settings.type === "app") {
        const w = outputElement.offsetWidth;
        const h = outputElement.offsetHeight;
        const bigger = w > h ? w : h;
        outputElement.style.width = bigger.toString() + "px";
        outputElement.style.height = bigger.toString() + "px";
        outputElement.style.borderRadius =
          ((10 / 57) * bigger).toString() + "px";
      }

      //Update resolution and set overflow back
      this.resolution =
        outputElement.offsetWidth.toString() +
        "x" +
        outputElement.offsetHeight.toString();
      document.querySelector(".output-container").style.overflow = "auto";
      outputElement.style.overflow = "auto";
    },

    //Download the image by creating a <a> element and clicking it
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
  background-color: transparent;
  margin: 0 auto;
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
    overflow-x: hidden;
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
  text-align: center;
}
.subheading {
  margin-bottom: 2px;
}
article {
  margin-bottom: var(--spacing);
  --block-spacing-horizontal: var(--spacing);
}
details {
  border-bottom: 0;
  margin-bottom: 0;
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
.window-bar {
  position: relative;
  padding-top: 1em;
  padding-left: 0.5em;
}
.window-bar span {
  border-radius: 100%;
  width: 1em;
  height: 1em;
  margin-left: 0.5em;
  display: inline-block;
}
.output > span {
  padding: 0 1em;
  display: inline-block;
}
span.indent {
  color: transparent;
}
pre code.output.hljs {
  overflow-x: visible;
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
