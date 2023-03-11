<template>
  <div
    :class="{
      'code-input-container': true,
      [className]: !!className,
    }"
    :style="`--ci-color-secondary: ${secondaryColor}; --ci-color-primary: ${primaryColor};`"
  >
    <div class="code-input font-alatsi" :class="parseBorders()">
      <p class="title" v-if="title">{{ title }}</p>
      <template v-for="(v, index) in values" :key="index">
        <input
          class="
            text-center
            transition-all
            border-none
            rounded-lg
            outline-none
            w-14
            h-14
            focus:outline-none focus:ring-0
          "
          type="number"
          pattern="[0-9]"
          :style="{
            width: `${fieldWidth}px`,
            height: `${fieldHeight}px`,
          }"
          :autoFocus="autoFocus && index === autoFocusIndex"
          :data-id="index"
          :value="v"
          :ref="
            (el) => {
              if (el) inputs[index + 1] = el;
            }
          "
          v-on:input="onValueChange"
          v-on:focus="(e) => onFocus(e, index)"
          v-on:keydown="onKeyDown"
          :required="required"
          :readonly="index-1 > 0 && values[index - 1] === ''"
          :disabled="disabled"
          maxlength="1"
        />
      </template>
    </div>
    <!-- Show error message below here -->
    <div class="error-message text-center" v-if="hasError && errorMessage">
      {{ errorMessage }}
    </div>
  </div>
</template>

<script>
import { defineComponent } from "vue";
import { ref, onBeforeUpdate } from "vue";

export default defineComponent({
  emits: ["change", "complete", "update:modelValue"],
  props: {
    className: String,
    primaryColor: {
      type: String,
      default: "#3880ff",
    },
    secondaryColor: {
      type: String,
      default: "#3dc2ff",
    },
    borders: {
      type: String,
      default: "btlr",
    },
    fields: {
      type: Number,
      default: 3,
    },
    fieldWidth: {
      type: Number,
      default: 56,
    },
    fieldHeight: {
      type: Number,
      default: 56,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    required: {
      type: Boolean,
      default: true,
    },
    hasError: {
      type: Boolean,
      default: true,
    },
    errorMessage: {
      type: String,
      default: "",
    },
    modelValue: {
      type: String,
      default: "",
    },
    title: String,
  },
  setup(props, { emit }) {
    const KEY_CODE = {
      backspace: 8,
      delete: 46,
      left: 37,
      up: 38,
      right: 39,
      down: 40,
    };

    const values = ref([]);
    const iRefs = ref([]);
    const inputs = ref([]);
    const fields = ref(props.fields);
    const autoFocusIndex = ref(0);
    const autoFocus = true;

    const initVals = () => {
      let vals;
      if (values.value && values.value.length) {
        vals = [];
        for (let i = 0; i < fields.value; i++) {
          vals.push(values.value[i] || "");
        }
        autoFocusIndex.value =
          values.value.length >= fields.value ? 0 : values.value.length;
      } else {
        vals = Array(fields.value).fill("");
      }
      iRefs.value = [];
      for (let i = 0; i < fields.value; i++) {
        iRefs.value.push(i + 1);
      }
      values.value = vals;
    };

    const onFocus = (e, prevIndex) => {
      const prev = inputs.value[prevIndex];
      if (prev && prev.value === "") {
        prev.focus();
      } else {
        e.target.select(e);
      }
    };

    const onValueChange = (e) => {
      const index = parseInt(e.target.dataset.id);
      e.target.value = e.target.value.replace(/[^\d]/gi, "");
      // this.handleKeys[index] = false;
      if (e.target.value === "" || !e.target.validity.valid) {
        return;
      }
      let next;
      const value = e.target.value;
      values.value = Object.assign([], values.value);
      if (value.length > 1) {
        let nextIndex = value.length + index - 1;
        if (nextIndex >= fields.value) {
          nextIndex = fields.value - 1;
        }
        next = iRefs.value[nextIndex];
        const split = value.split("");
        split.forEach((item, i) => {
          const cursor = index + i;
          if (cursor < fields.value) {
            values.value[cursor] = item;
          }
        });
      } else {
        next = iRefs.value[index + 1];
        values.value[index] = value;
      }
      if (next) {
        const element = inputs.value[next];
        element.focus();
        element.select();
      }
      triggerChange(values.value);
    };

    const onKeyDown = (e) => {
      const index = parseInt(e.target.dataset.id);
      const prevIndex = index - 1;
      const nextIndex = index + 1;
      const prev = iRefs.value[prevIndex];
      const next = iRefs.value[nextIndex];
      switch (e.keyCode) {
        case KEY_CODE.backspace: {
          e.preventDefault();
          const vals = [...values.value];
          if (values.value[index]) {
            vals[index] = "";
            values.value = vals;
            triggerChange(vals);
          } else if (prev) {
            vals[prevIndex] = "";
            inputs.value[prev].focus();
            values.value = vals;
            triggerChange(vals);
          }
          break;
        }
        case KEY_CODE.delete: {
          e.preventDefault();
          const vals = [...values.value];
          if (values.value[index]) {
            vals[index] = "";
            values.value = vals;
            triggerChange(vals);
          } else if (next) {
            vals[nextIndex] = "";
            inputs.value[next].focus();
            values.value = vals;
            triggerChange(vals);
          }
          break;
        }
        case KEY_CODE.left:
          e.preventDefault();
          if (prev) {
            inputs.value[prev].focus();
          }
          break;
        case KEY_CODE.right:
          e.preventDefault();
          if (next) {
            inputs.value[next].focus();
          }
          break;
        case KEY_CODE.up:
        case KEY_CODE.down:
          e.preventDefault();
          break;
        default:
          // this.handleKeys[index] = true;
          break;
      }
    };

    const triggerChange = (values = values.value) => {
      const val = values.join("");
      emit("change", val);
      emit("update:modelValue", val);
      emit("complete", val.length >= fields.value);
    };

    initVals();

    onBeforeUpdate(() => {
      inputs.value = [];
    });

    return {
      inputs,
      values,
      onFocus,
      autoFocus,
      onKeyDown,
      onValueChange,
      autoFocusIndex,
      parseBorders() {
        const actualBorders = {
          b: "border-b",
          t: "border-t",
          l: "border-l",
          r: "border-r",
        };
        // Split each the letters in the string
        const letters = props.borders.split("");
        // Map the letters to the actual borders
        const mapped = letters.map((letter) => actualBorders[letter]);
        // Join the mapped borders with a space
        return mapped.join(" ");
      },
    };
  },
});
</script>

<style scoped lang="scss">
@import url(https://fonts.bunny.net/css?family=anton:400);

.code-input-container {
  --ci-color-primary: #3880ff;
  --ci-color-secondary: #3dc2ff;
}

// Tailwind text-center
.text-center {
  text-align: center;
}
// Tailwind transition-all
.transition-all {
  transition-property: all;
}
// Tailwind border-none
.border-none {
  border-width: 0;
}
// Tailwind rounded-lg
.rounded-lg {
  border-radius: 0.5rem;
}
// Tailwind outline-none
.outline-none {
  outline: 0;
}
// Tailwind w-14
.w-14 {
  width: 3.5rem;
}
// Tailwind h-14
.h-14 {
  height: 3.5rem;
}
// Tailwind focus:outline-none focus:ring-0
.focus\:outline-none.focus\:ring-0:focus {
  outline: 0;
  box-shadow: none;
}

.code-input-container {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 5px;
}
.code-input {
  display: flex;
  flex-direction: row;
  justify-content: center;
  gap: 10px;
  & > input {
    font-family: "Anton", sans-serif;
    font-size: 30px;
    border-radius: 0;
    text-align: center;
    transition: 0.2s all ease-in-out;
    color: var(--ci-color-primary);
    box-sizing: border-box;
    appearance: initial;
    -webkit-appearance: initial;
    &:focus {
      outline: none;
      caret-color: var(--ci-color-secondary);
    }
  }
  &.border-b {
    & > input {
      border-bottom: solid 2px var(--ci-color-primary);
      &:focus {
        border-bottom: 2px solid var(--ci-color-secondary);
      }
    }
  }
  &.border-t {
    & > input {
      border-top: solid 2px var(--ci-color-primary);
      &:focus {
        border-top: 2px solid var(--ci-color-secondary);
      }
    }
  }
  &.border-l {
    & > input {
      border-left: solid 2px var(--ci-color-primary);
      &:focus {
        border-left: 2px solid var(--ci-color-secondary);
      }
    }
  }
  &.border-r {
    & > input {
      border-right: solid 2px var(--ci-color-primary);
      &:focus {
        border-right: 2px solid var(--ci-color-secondary);
      }
    }
  }
}
.title {
  margin: 0;
  height: 20px;
  padding-bottom: 10px;
}
.error-message {
  color: red;
  font-size: 12px;
  margin: 0;
  padding-top: 5px;
}
/* Chrome, Safari, Edge, Opera */
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* Firefox */
input[type="number"] {
  -moz-appearance: textfield;
}
</style>
