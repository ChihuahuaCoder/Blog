---
title: "Implementation of i18n in the Vue 3 Composition API"
datePublished: Tue Apr 16 2024 12:15:21 GMT+0000 (Coordinated Universal Time)
cuid: clv2ck1pq000508id1dwr988c
slug: implementation-of-i18n-in-the-vue-3-composition-api
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1713270045792/d2cce44e-d258-4941-ad1c-1eae78ed182b.png
tags: vuejs, i18n

---

i18n stands for internationalisation, or internationalisation. It is a term used to describe processes and techniques for adapting software to different languages and cultures, without changing the source code. i18n enables not only the translation of texts but also the formatting of dates, times, currencies, and numbers.

There are many tools and libraries available to facilitate the implementation of i18n. For the Vue 3 environment, the most popular library is vue-i18n. In this article, I will show you how to get started with i18n simply.

## Installation

NPM:

```bash
npm install vue-i18n@9
```

Yarn:

```bash
yarn add vue-i18n@9
```

[Full documentation](https://vue-i18n.intlify.dev/guide/installation)

## File **Preparation**

In the src directory, create a new folder called locales. This is where you will store all your translation files. Create a new file — *en.json*. You can choose another format (e.g. YAML), but as JSON is more common in this case, it will serve as the template.

```json
"common": {
 "next": "Next",
 "cancel": "Cancel"
},
"home": {
 "logOut": {
   "logIn": "Log in",
   "register": "Register"
 },
  "logIn": {
   "welcome": "Welcome!",
 }
}
```

The keys must be the same in all translation files. If you want to add another language, you must create a new file and assign appropriate values for the existing keys. For example, for the Polish language, this would be the file *pl.json*.

```json
"common": {
 "next": "Dalej",
 "cancel": "Anuluj"
},
"home": {
 "logOut": {
   "logIn": "Zaloguj się",
   "register": "Załóż konto"
 },
  "logIn": {
   "welcome": "Witaj!",
 }
}
```

### Configuration

Navigate to the main.ts file. Import the translation files you just created. Next, you need to configure i18n.

```typescript
import pl from "./locales/pl.json" 
import en from "./locales/en.json" 

const i18n = createI18n({ 
  locale: navigator.language, 
  fallbackLocale: "en", 
  messages: { pl, en }, 
  legacy: false 
})
```

Locale allows you to specify what you want the start-up language of your application to be. You can specify a specific one here, e.g. “en”, or use the `navigator.language` property, which corresponds to the language of the user’s browser.

`FallbackLocale` points to a translation file that will be used when the user’s browser is in a language that your application does not support, or when the corresponding key is missing from another JSON file.

You declare the translation files in the messages property. You must set the legacy field to false if you are using the Composition API.

Once the application instance has been created, it is necessary to register the i18n plugin.

```typescript
const app = createApp(App) 
app.use(i18n) 
app.mount('#app')
```

## Component usage

To use i18n in a component you must import it and then destructure it to access the function t. This will be used to perform translations.

```xml
<script setup lang='ts'>
  import {useI18n} from 'vue-i18n' 

  const {t} = useI18n();
</script>
```

In the HTML part, instead of strings, use the corresponding keys, with the help of the `t` function.

```xml
<span>{{t('home.logIn.welcome')}</span>
```

### Language switcher

It is useful to give the user the option of adjusting the language to their preference. To do this, you can use the locale property.

```xml
<script setup lang="ts">
import {useI18n} from 'vue-i18n'

const {locale} = useI18n();
const currentLanguage = locale.value;

</script>
```

To change the active language, simply assign a new value to `locale.value`, e.g. “en”. I will present my implementation of the language switcher below.

I first added the `SelectOption` interface. The `label` will correspond to the text displayed in Select and the `value` will point to the file name with the corresponding translation.

```typescript
export interface SelectOption {
   label: string,
   value: string
}
```

I then created the `LanguageSwitcher` component. It stores information about the available languages, handles their switching, and communicates with the child component `BaseSelect`. It accepts an options property of type array `SecetOption` and supports a `modelValue` of type string.

```xml
<script setup lang="ts">
import {useI18n} from 'vue-i18n'
import BaseSelect from "@/components/common/baseSelect/BaseSelect.vue";
import type {SelectOption} from "@/components/common/baseSelect/selectOption";
import {ref, watchEffect} from "vue";

const {locale} = useI18n();

const options: SelectOption[] = [
 {
   label: 'English',
   value: 'en'
 },
 {
   label: 'Polski',
   value: 'pl'
 }
];
const currentLanguage = options.find((x) => x.value === locale.value)?.label ?? "English";
const selectedOption = ref(currentLanguage);

watchEffect(() => {
 locale.value = options.find((x) => x.label === selectedOption.value)!.value;
})

</script>

<template>
 <BaseSelect v-model="selectedOption" :options="options"/>
</template>
```

`CurrentLanguage` checks what language is currently set as the leading language. If the user is using a language not supported by the application, English will be selected.

Based on the change in value in `BaseSelect`, `WatchEffect` searches for the relevant value and assigns it to `locale.value`.

## Next steps

The Vue-i18n offers enormous possibilities. The complete [documentation](https://vue-i18n.intlify.dev/guide/essentials/syntax) is well worth reading.