<template>
  <label for="langSelect">Language</label>
  <input
    type="text"
    id="langSelect"
    style="margin-bottom: calc(var(--spacing) * 0.25)"
    placeholder="Search Languages"
    v-model="enteredText"
    @focus="revealSearch()"
    @focusout="hideSearch()"
  />
  <ul id="langSearch" class="listPopUp" role="listbox">
    <li
      v-for="(item, index) in filteredLanguages"
      :key="index"
      @mousedown="selectLanguage(item)"
      :class="{ selected: item == language }"
    >
      {{ item }}
    </li>
  </ul>
</template>

<script>
export default {
  data() {
    return {
      language: "javascript",
      languages: [
        "actionscript",
        "ada",
        "apache",
        "applescript",
        "arduino",
        "asciidoc",
        "bash",
        "basic",
        "brainfuck",
        "clojure",
        "clojure-repl",
        "cmake",
        "coffeescript",
        "cpp",
        "cs",
        "csp",
        "css",
        "d",
        "dart",
        "diff",
        "django",
        "dockerfile",
        "erlang",
        "erlang-repl",
        "excel",
        "fortran",
        "go",
        "gradle",
        "handlebars",
        "haskell",
        "htmlbars",
        "java",
        "javascript",
        "json",
        "julia",
        "julia-repl",
        "kotlin",
        "less",
        "lisp",
        "lua",
        "makefile",
        "markdown",
        "mathematica",
        "matlab",
        "moonscript",
        "nginx",
        "objectivec",
        "ocaml",
        "perl",
        "php",
        "plaintext",
        "powershell",
        "processing",
        "prolog",
        "purebasic",
        "python",
        "r",
        "ruby",
        "rust",
        "scala",
        "scheme",
        "scss",
        "shell",
        "sql",
        "swift",
        "tex",
        "typescript",
        "vbscript",
        "vim",
        "xml",
        "xquery",
        "yaml",
      ],
      enteredText: "javascript",
    };
  },
  computed: {
    filteredLanguages() {
      return this.languages.filter((e) => e.includes(this.enteredText));
    },
  },
  emits: ["changeLang"],
  methods: {
    selectLanguage(lang) {
      this.language = lang;
      document.querySelector("#langSearch").blur();
    },
    revealSearch() {
      document.querySelector("#langSearch").style.display = "flex";
      this.enteredText = "";
    },
    hideSearch() {
      document.querySelector("#langSearch").style.display = "none";
      this.enteredText = this.language;
    },
  },
  watch: {
    language(newer) {
      this.$emit("changeLang", newer);
    },
  },
};
</script>

<style scoped>
.listPopUp {
  width: auto;
  max-height: calc(3 * ((var(--form-element-spacing-vertical) * 1.75) + 1em));
  overflow-y: auto;
  display: none;
  align-items: center;
  z-index: 1;
  position: relative;
  top: 0.5em;
  margin: 0 auto;
  bottom: var(--spacing);
  flex-direction: column;
  padding: 0;
  border: var(--border-width) solid var(--dropdown-border-color);
  border-radius: var(--border-radius);
  background-color: var(--dropdown-background-color);
  color: var(--dropdown-color);
  white-space: nowrap;
}
.listPopUp li {
  cursor: pointer;
  width: 100%;
  display: block;
  margin-bottom: 0;
  padding: calc(var(--form-element-spacing-vertical) * 0.5)
    var(--form-element-spacing-horizontal);
  list-style: none;
}
.listPopUp li:hover {
  background-color: var(--dropdown-hover-background-color);
}
.listPopUp li.selected {
  background-color: var(--primary);
  color: var(--primary-inverse);
}
.listPopUp li.selected:hover {
  background-color: var(--primary-hover);
}
</style>
