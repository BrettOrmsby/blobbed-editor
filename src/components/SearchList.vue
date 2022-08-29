<template>
  <label :for="`${this.for}Search`">{{ label }}</label>
  <input
    type="text"
    autocomplete="false"
    :id="`${this.for}Search`"
    style="margin-bottom: calc(var(--spacing) * 0.25)"
    :placeholder="`Search ${this.for}`"
    @keyup.enter="selectTop()"
    v-model="enteredText"
    @focus="revealSelect()"
    @focusout="hideSelect()"
  />
  <ul :id="`${this.for}Select`" class="listPopUp" role="listbox">
    <li
      v-for="(item, index) in filteredList"
      :key="index"
      @mousedown.prevent="selectItem(item)"
      :class="{ selected: item == output }"
      v-html="bold(item)"
    >
    </li>
  </ul>
</template>

<script>
export default {
  props: ["for", "list", "label", "start"],
  data() {
    return {
      output: "",
      enteredText: "",
    };
  },
  created() {
    this.output = this.start;
    this.enteredText = this.start;
  },
  computed: {
    filteredList() {
      return this.list.filter((e) => e.includes(this.enteredText));
    },
  },
  emits: ["update"],
  methods: {
    selectTop() {
      this.output = this.filteredList[0] || this.output;
      document.querySelector(`#${this.for}Search`).blur();
    },
    selectItem(item) {
      this.output = item;
      document.querySelector(`#${this.for}Search`).blur();
    },
    revealSelect() {
      document.querySelector(`#${this.for}Select`).style.display = "flex";
      this.enteredText = "";
    },
    hideSelect() {
      document.querySelector(`#${this.for}Select`).style.display = "none";
      this.enteredText = this.output;
    },
    bold(item) {
      return item.replace(this.enteredText, `<b>${this.enteredText}</b>`)
    }
  },
  watch: {
    output(newer) {
      this.$emit("update", newer);
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
