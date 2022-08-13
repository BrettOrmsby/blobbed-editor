<!--
modified code from https://github.com/justcaliturner/simple-code-editor
-->
<template>
  <div
    class="code_editor hljs"
    :class="{
      hide_header: withoutHeader,
      scroll: canScroll,
      read_only: read_only,
      wrap_code: wrap_code,
    }"
    :style="{
      width: width,
      height: height,
      borderRadius: border_radius,
      zIndex: z_index,
      maxWidth: max_width,
      minWidth: min_width,
      maxHeight: max_height,
      minHeight: min_height,
    }"
  >
    <div class="header" v-if="withoutHeader == true ? false : true"></div>
    <div
      class="code_area"
      :style="{
        borderBottomLeftRadius: border_radius,
        borderBottomRightRadius: border_radius,
        borderTopLeftRadius: withoutHeader == true ? border_radius : 0,
        borderTopRightRadius: withoutHeader == true ? border_radius : 0,
      }"
    >
      <textarea
        v-if="
          read_only == true ? false : modelValue === undefined ? true : false
        "
        ref="textarea"
        :autofocus="autofocus"
        @input="calcContainerWidth"
        @keydown.tab.prevent.stop="tab"
        v-on:scroll="scroll"
        v-model="staticValue"
        :style="{ fontSize: font_size }"
      ></textarea>
      <textarea
        v-if="
          read_only == true ? false : modelValue === undefined ? false : true
        "
        ref="textarea"
        :autofocus="autofocus"
        @keydown.tab.prevent.stop="tab"
        v-on:scroll="scroll"
        :value="modelValue"
        @input="
          $emit('update:modelValue', $event.target.value),
            calcContainerWidth($event)
        "
        :style="{ fontSize: font_size }"
      ></textarea>
      <pre
        :style="{ width: containerWidth === 0 ? '' : containerWidth + 'px' }"
      >
        <code
            v-highlight="contentValue"
            :class="languageClass"
            :style="{ top: top + 'px', left: left + 'px', fontSize: font_size, borderBottomLeftRadius: read_only == true ? border_radius : 0, borderBottomRightRadius: read_only == true ? border_radius : 0 }"
        ></code>
      </pre>
    </div>
  </div>
</template>

<script>
import hljs from "highlight.js";

export default {
  name: "CodeEditor",
  props: {
    modelValue: {
      type: String,
    },
    wrap_code: {
      type: Boolean,
      default: false,
    },
    read_only: {
      type: Boolean,
      default: false,
    },
    autofocus: {
      type: Boolean,
      default: false,
    },
    hide_header: {
      type: Boolean,
      default: false,
    },
    value: {
      type: String,
      default: "",
    },
    width: {
      type: String,
      default: "540px",
    },
    height: {
      type: String,
      default: "auto",
    },
    max_width: {
      type: String,
    },
    min_width: {
      type: String,
    },
    max_height: {
      type: String,
    },
    min_height: {
      type: String,
    },
    border_radius: {
      type: String,
      default: "12px",
    },
    selector_width: {
      type: String,
      default: "110px",
    },
    selector_height: {
      type: String,
      default: "auto",
    },
    selector_displayed_by_default: {
      type: Boolean,
      default: false,
    },
    copy_code: {
      type: Boolean,
      default: true,
    },
    z_index: {
      type: String,
    },
    font_size: {
      type: String,
      default: "17px",
    },
    theme: {
      type: String,
      default: "dark",
    },
    language: {
      type: String,
      default: "javascript",
    },
  },
  directives: {
    highlight: {
      //vue2
      bind(el, binding) {
        el.textContent = binding.value;
        hljs.highlightElement(el);
      },
      componentUpdated(el, binding) {
        el.textContent = binding.value;
        hljs.highlightElement(el);
      },
      //vue3
      created(el, binding) {
        el.textContent = binding.value;
        hljs.highlightElement(el);
      },
      updated(el, binding) {
        el.textContent = binding.value;
        hljs.highlightElement(el);
      },
    },
  },
  data() {
    return {
      containerWidth: 0,
      staticValue: this.value,
      top: 0,
      left: 0,
      content:
        this.modelValue === undefined ? this.staticValue : this.modelValue,
    };
  },
  watch: {
    value(value) {
      this.staticValue = value;
    },
  },
  computed: {
    languageClass() {
      return "hljs language-" + this.language;
    },
    contentValue() {
      return this.read_only
        ? this.value
        : this.modelValue === undefined
        ? this.staticValue + "\n"
        : this.modelValue + "\n";
    },
    canScroll() {
      return this.height == "auto" ? false : true;
    },
    withoutHeader() {
      return true;
    },
  },
  methods: {
    calcContainerWidth(event) {
      //  calculating the textarea's width while typing for syncing the width between textarea and highlight area
      this.containerWidth = event.target.clientWidth;
    },
    tab() {
      document.execCommand("insertText", false, "    ");
    },
    scroll(event) {
      this.top = -event.target.scrollTop;
      this.left = -event.target.scrollLeft;
    },
    resize() {
      // listen to the change of the textarea's width to resize the highlight area
      const resize = new ResizeObserver((entries) => {
        for (let entry of entries) {
          const obj = entry.contentRect;
          this.containerWidth = obj.width + 40; // 40 is the padding
        }
      });
      // only the textarea is rendered the listener will run
      if (this.$refs.textarea) {
        resize.observe(this.$refs.textarea);
      }
    },
  },
  mounted() {
    this.$nextTick(function () {
      this.content =
        this.modelValue === undefined ? this.staticValue : this.modelValue;
    });
    this.resize();
  },
  updated() {
    this.$emit("input", this.staticValue);
    this.$nextTick(function () {
      this.content =
        this.modelValue === undefined ? this.staticValue : this.modelValue;
    });
  },
};
</script>

<style scoped>
/* header */
.header {
  position: relative;
  z-index: 2;
  height: 34px;
  box-sizing: border-box;
}
.header > .dropdown {
  position: absolute;
  top: 12px;
  left: 18px;
}
.header > .copy_code {
  position: absolute;
  top: 10px;
  right: 12px;
}

/* code_area */
.code_editor {
  display: flex;
  flex-direction: column;
  font-size: 0;
  position: relative;
  text-align: left;
}
.code_editor > .code_area {
  position: relative;
  overflow: hidden;
}
.code_editor > .code_area > textarea,
.code_editor > .code_area > pre > code {
  padding: 0px 20px 20px 20px;
  font-family: Consolas, Monaco, monospace;
  line-height: 1.5;
  font-size: 16px;
}
.code_editor > .code_area > textarea {
  overflow-y: hidden;
  box-sizing: border-box;
  caret-color: rgba(127, 127, 127);
  -webkit-text-fill-color: transparent;
  white-space: pre;
  word-wrap: normal;
  border: 0;
  position: absolute;
  z-index: 1;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: none;
  resize: none;
}
.code_editor > .code_area > textarea:hover,
.code_editor > .code_area > textarea:focus-visible {
  outline: none;
}
.code_editor > .code_area > pre {
  position: relative;
  margin: 0;
}
.code_editor > .code_area > pre > code {
  position: relative;
  overflow-x: visible;
  border-radius: 0;
  box-sizing: border-box;
  display: block;
  border: none;
  margin: 0;
}

/* hide_header */
.hide_header > .code_area > textarea,
.hide_header > .code_area > pre > code {
  padding: 20px;
}
.hide_header.scroll > .code_area {
  height: 100%;
}

/* read_only */
.read_only > .code_area > pre > code {
  width: 100%;
  height: 100%;
  overflow: auto !important;
}

/* wrap code */
.wrap_code > .code_area > textarea,
.wrap_code > .code_area > pre > code {
  white-space: pre-wrap;
  word-wrap: break-word;
}

/* scroll */
.scroll > .code_area {
  height: calc(100% - 34px);
}
.scroll > .code_area > textarea {
  overflow: auto;
}
.scroll > .code_area > pre {
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.scroll,
.code_area .code_editor,
textarea {
  -ms-overflow-style: none; /* Internet Explorer 10+ */
  scrollbar-width: none; /* Firefox */
}
textarea::-webkit-scrollbar,
.scroll::-webkit-scrollbar,
.code_area::-webkit-scrollbar,
.code_editor::-webkit-scrollbar {
  display: none; /* Safari and Chrome */
}
</style>
