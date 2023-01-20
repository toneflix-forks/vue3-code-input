# vue3-verification-code-input

> 🎉A verification code input for vue 3
> This is a fork of zlayine's vue3-verification-code-input with a lot more improvements on the way and is currently maintained.

[![NPM](https://img.shields.io/npm/v/vue3-verification-code-input.svg)](https://www.npmjs.com/package/vue3-verification-code-input) [![JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://standardjs.com)

[![NPM](https://nodei.co/npm/vue3-verification-code-input.png)](https://nodei.co/npm/vue3-verification-code-input/)

## Demo

![Vue 3 Verification Code Demo](https://github.com/zlayine/vue3-verification-code-input/blob/master/public/demo_image.png?raw=true)

## Install

```bash
npm install --save vue3-verification-code-input
```

## Usage

```jsx
<template>
  <code-input
    @complete="completed = true"
    :fields="3"
    :fieldWidth="56"
    :fieldHeight="56"
    :required="true"
    v-model="code"
  />
  <button class="btn" :disabled="!completed">
    Continue
  </button>
</template>

<script setup>
import CodeInput from "./components/CodeInput.vue";
import { ref } from "vue";

const completed = ref(false);
const code = ref('');
</script>
```

## PropTypes

|        Key       |  Type  |              Desc               |
| :--------------: | :----: | :-----------------------------: |
|      borders     | string | Borders to show (Default: btlr) |
|      fields      | number |    The count of characters      |
|     disabled     |  bool  |      Disable the inputs         |
|     required     |  bool  |      require the inputs         |
|    fieldWidth    | number |          input width            |
|    fieldHeight   | number |         input height            |
|       title      | string |       code input title          |
|     className    | string |          class name             |
|   primaryColor   | string |   The main component color      |
|  secondaryColor  | string | The alternate component color   |

## EmitTypes

|   Key    | Type |              Desc               |
| :------: | :--: | :-----------------------------: |
|  change  | func |   Trigger on character change   |
| complete | func | Trigger on all character inputs |

## License

MIT
