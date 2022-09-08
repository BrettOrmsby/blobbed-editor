<template>
  <main class="main">
    <div class="sidebar container">
      <hgroup class="title">
        <h1>Blobbed Editor</h1>
        <p style="margin-bottom: 0">
          Create blobbed, syntax-highlighted images of your code.
        </p>
      </hgroup>
      <div class="download-button">
        <button v-if="downloading" aria-busy="true">Please Wait...</button>
        <button v-else @click="downloadImage()">Download</button>
      </div>
      <strong class="subheading">Settings</strong>
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
        <label for="lineSpacing"
          >Line Spacing
          <input
            type="range"
            min="0"
            max="2"
            step="0.05"
            v-model="settings.lineSpacing"
            id="lineSpacing"
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
      <strong class="subheading">Highlighting</strong>
      <article>
        <SearchList
          @update="(lang) => (settings.language = lang)"
          label="Language"
          for="Languages"
          :list="highlightData.languages"
          :start="this.settings.language"
        />
        <SearchList
          @update="(theme) => (settings.theme = theme)"
          label="Theme"
          for="Themes"
          :list="highlightData.themes"
          :start="this.settings.theme"
        />
      </article>
      <strong class="subheading">Data</strong>
      <article>
        <span>Characters Remaining: {{ charLimit }}</span
        ><br />
        <span>Image Resolution: {{ resolution }}</span>
      </article>
      <details class="faq-button">
        <summary role="button">FAQs</summary>
        <article>
          <b>How do indents work?</b>
          <p>
            Indents blobs are made whenever there are 2 spaces or a tab in your
            code. Indents will be made even if they are within other blobs (like
            comments) and will break up the blob.
          </p>
          <b>How does filtering smaller blobs work?</b>
          <p>
            When you chose to filter smaller blobs, all blobs with a character
            length of 1 are removed. This will keep newline characters even if
            their full line is removed.
          </p>
          <b>Why is my image not downloaded properly?</b>
          <p>
            The blobbed images might be downloaded improperly if the resolution
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
          <p>
            There is a character limit because the blobbed image is much more
            likely to be rendered improperly if there are too many blobs.
          </p>
          <b>Where can I find the source for Blobbed Editor?</b>
          <p style="margin-bottom: 0">
            You can find the source on
            <a href="https://github.com/BrettOrmsby/blobbed-editor">GitHub</a>.
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
import SearchList from "./components/SearchList.vue";

export default {
  components: {
    CodeEditor,
    SearchList,
  },
  data() {
    return {
      downloading: false,
      input: 'const enter = yourCode("here")',
      resolution: "",
      highlightData: require("@/assets/highlight-data.json"),
      settings: {
        theme: "",
        language: "javascript",
        blobBorderRadius: 1,
        imageSize: 16,
        blobSpacing: 0.25,
        lineSpacing: 0.25,
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
    "settings.theme": async function () {
      let data = await fetch(
        "https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.6.0/styles/" +
          this.settings.theme +
          ".min.css"
      );
      let content = await data.text();
      document.getElementById("customStyle").innerHTML = content;
      this.updatePreview();
    },
  },
  created() {
    // Force load the first theme
    this.settings.theme = "panda-syntax-dark";
  },
  methods: {
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
            .join("\n<span class='indent new-line-spacer'></span>");
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

      //Filter and apply unique styles to each blob
      document.querySelectorAll("#output > span").forEach((e) => {
        //Filter non-indent and non-line-number smaller blobs if filterSmallerBlobs
        if (
          this.settings.filterSmallerBlobs &&
          e.innerText.length <= 1 &&
          !e.classList.contains("line-number") &&
          !e.classList.contains("indent")
        ) {
          e.style.display = "none";
          return;
        }
        //Set background color to the color provided by hljs
        e.style.backgroundColor = window.getComputedStyle(e).color;

        //Update width based on the amount of characters
        e.style.width =
          (e.innerText.length * this.settings.blobWidth).toString() + "em";

        //Add inner blank character so the span width is kept
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
        (outputElement.offsetWidth * 2).toString() +
        "x" +
        (outputElement.offsetHeight * 2).toString();
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
@import "@/assets/theme.css";
@import "@/assets/main.css";
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
  border-radius: v-bind('settings.blobBorderRadius + "em"');
  margin: v-bind("`${settings.lineSpacing}em ${settings.blobSpacing}em`");
}
span.indent {
  color: transparent;
}
span.new-line-spacer {
  padding: 0;
  margin: 0;
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
  padding: v-bind('settings.editorPadding + "em"');
  font-size: v-bind('settings.imageSize + "px"');
}
</style>
