<template>
  <div class="dropdown b-dropdown" :class="dropdownClasses" :id="safeId()">
    <b-form-input
      :id="safeId('_BV_input_')"
      ref="toggle"
      @click="show"
      @keydown="onKeydownLocal"
      v-model="inputValue"
    />
    <b-button
      v-if="showButton"
      class="dropdown-toggle"
      :class="toggleClasses"
      :props="{ ...commonProps, tag: toggleTag }"
      @mousedown="onMousedown"
      @click="toggle"
    >
      <span class="sr-only">{{ toggleText }}</span>
    </b-button>
    <ul
      class="dropdown-menu"
      :class="menuClasses"
      :role="role"
      tabindex="-1"
      :aria-labelledby="safeId(showButton ? '_BV_toggle_' : '_BV_input_')"
      @keydown="onKeydown"
      ref="menu"
    >
      <b-dropdown-item
        v-for="option in filteredOptions"
        :key="option"
        @click="onItemClick(option)"
      >
        {{ option }}
      </b-dropdown-item>

      <b-dropdown-item
        v-if="!filteredOptions.length"
        @click="onItemClick(null)"
      >
        No matching items...
      </b-dropdown-item>
    </ul>
  </div>
</template>

<script>
import idMixin from "bootstrap-vue/src/mixins/id";
import dropdownMixin, {
  props as dropdownProps,
} from "bootstrap-vue/src/mixins/dropdown";
import { makePropsConfigurable } from "bootstrap-vue/src/utils/config";
import normalizeSlotMixin from "bootstrap-vue/src/mixins/normalize-slot";
import { requestAF } from "bootstrap-vue/src/utils/dom";
import {
  CODE_ENTER,
  CODE_ESC,
} from "bootstrap-vue/src/constants/key-codes";

import { stopEvent } from "bootstrap-vue/src/utils/events";

const CODE_TAB = 9;

const NAME_DROPDOWN = "DsAutoComplete";

export const props = makePropsConfigurable(
  {
    ...dropdownProps,

    variant: {
      type: String,
      default: "secondary",
    },
    size: {
      type: String,
      // default: null
    },
    block: {
      type: Boolean,
      default: false,
    },
    menuClass: {
      type: [String, Array, Object],
      // default: null
    },
    toggleTag: {
      type: String,
      default: "button",
    },
    toggleText: {
      // TODO: This really should be `toggleLabel`
      type: String,
      default: "Toggle dropdown",
    },
    toggleClass: {
      type: [String, Array, Object],
      // default: null
    },

    lazy: {
      // If true, only render menu contents when open
      type: Boolean,
      default: false,
    },
    role: {
      type: String,
      default: "menu",
    },
    showButton: {
      // If true, only render menu contents when open
      type: Boolean,
      default: false,
    },
    options: {
      type: Array,
      required: true,
    },
    value: {},
    optionString: {
      type: Function,
      default: (o) => o,
    },
    filter: {
      type: Function,
      // todo
      default: () => [],
    },
  },
  NAME_DROPDOWN
);

export default {
  name: NAME_DROPDOWN,
  mixins: [idMixin, dropdownMixin, normalizeSlotMixin],
  props,
  mounted() {
    this.$root.$on("bv::dropdown::hidden", () => { this.closeMenuHandler(); });
  },

  methods: {
    focusMenu() {}, // override dropdownMixin to not change focus
    onItemClick(option) {
      this.inputValue = option;
      this.$emit("input", option);
    },

    onKeydownLocal(evt) {
      const { keyCode } = evt;

      if ([CODE_ESC, CODE_TAB].indexOf(keyCode) === -1) {
        // Close on ESC
        this.show();
      }
      if (keyCode === CODE_TAB && this.visible && this.showButton) {
        this.focusNext(evt, false);
        stopEvent(evt);
      }

      if (keyCode === CODE_ENTER) {
        this.attemptSelect(evt);
      }

      this.onKeydown(evt);
      //attemptFocus(this.$refs.menu);
    },
    clickOutHandler(evt) {
      this.closeMenuHandler(evt)
    },

    closeMenuHandler() {
      // override bootstrapjs
      this.hide();
      requestAF(() => {
        if (this.inputValue === "") this.$emit("input", "");
        else this.inputValue = this.value;
      });
    },

    attemptSelect() {
      if (this.filteredOptions.length === 1) {
        this.onItemClick(this.filteredOptions[0]);
        requestAF(() => {
          this.visible = false;
        });
      }
    },
  },

  data() {
    // add to dropdownMixin data() to include local data
    return {
      visible: false,
      visibleChangePrevented: false,
      inputValue: "",
    };
  },
  computed: {
    split() {
      return true;
    },

    // take 2 inputs

    filteredOptions() {
      return this.options.filter(
        (o) =>
          o.toLowerCase().indexOf((this.inputValue || "").toLowerCase()) > -1
      );
    },

    commonProps() {
      return {
        variant: this.variant,
        size: this.size,
        block: this.block,
        disabled: this.disabled,
      };
    },
    dropdownClasses() {
      const { block } = this;
      return [
        this.directionClass,
        this.boundaryClass,
        {
          show: this.visible,
          "btn-group": !block || true,
        },
      ];
    },
    menuClasses() {
      return [
        this.menuClass,
        {
          "dropdown-menu-right": this.right,
          show: this.visible,
        },
      ];
    },
    toggleClasses() {
      return [
        this.toggleClass,
        {
          "dropdown-toggle-split": true,
        },
      ];
    },
  },
};
</script>
