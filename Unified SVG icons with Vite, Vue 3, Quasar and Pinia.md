### Summary

In this article, we will:

- use custom SVG icons from Figma
- build a flexible and customizable reusable component to use SVG icons:
  ![Image from ray.so https://ray.so/?title=html&theme=breeze&spacing=64&background=true&darkMode=true&code=PFN2Z0ljb24gCiAgbmFtZT0iYXJyb3ciCiAgY29sb3I9ImJsdWUiCiAgc2l6ZT0ibWQiCiAgaG92ZXItY29sb3I9IndoaXRlIgogIHRyYW5zZm9ybT0icm90YXRlKDQ1KSIKLz4&language=xml](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/id83fdn6y98xy2cbseuj.png)
- create an interactive playground to play with the SvgIcon component and generate the code
- use [Vite](https://vitejs.dev), [Vue 3](https://vuejs.org), [Quasar](https://quasar.dev) and [Pinia](https://pinia.vuejs.org)
- use script setup and options API with Vue 3
- auto-register global components using `import.meta.glob`
- add responsive design with CSS Grid
- add CSS Gradient rounded borders
- link CSS values to dynamic component state using `v-bind` CSS function

> Wait😲! All of these in a single article ?

YES! Let's do it! 🤹‍♂️

First, let's see what we are going to build:

- [See the app live]([Click here to see live what we are going to build](https://stupefied-montalcini-342150.netlify.app))
- [Play with the code online](https://stackblitz.com/github/Rolanddoda/unified-way-of-using-svg-icons?file=src%2FApp.vue) (you might have to run `vite` in the terminal to run the app)

### Requirements

_Let's add a designer into the game who defines how we should build that component._

Our designer 👨‍🎨/👩‍🎨 is tired of using icon sets and has decided that the app needs its own unique SVG icons. Here are the specifications he/she gave us:

- Most icons fall in 16, 24, 32, 48 sizes. However, a few icons, some special ones, have different sizes. The most used size is 48px.

- SVG icons are scalable and their strokes too, but our smart designer wants to manually control the stroke width in different sizes:
  - 16px and 24px: 1px stroke width
  - 32px: 2px stroke width
  - 48px: 2.5px stroke width

- All icons have outlined style.

- All icons should have a default color which is called 'primary' and a default hover color `white`, however, it must be easy to overwrite those defaults.

- The app has colors defined in CSS variables, but it should be possible to set the color with a color value (hex, rgb etc)

  Here are the CSS Variables colors:

  ```css
  --primary: #007bff;
  --secondary: #6c757d;
  --positive: #28a745;
  --negative: #dc3545;
  --info: #17a2b8;
  --warning: #ffc107;
  ```

Alright! As you can see, we need a flexible and customizable reusable component. 🏯

### Let's start coding! ⌨️🔥 

We could start by creating a new Vite + Vue 3 project which you can do by running `npm init vue@latest` in the terminal, but to speed things up, I have already done that, cleaned up the project and added some SVG icons.

So, [clone or download the repo](https://github.com/Rolanddoda/svg-icons-interactive-playground) or play directly with the code online [on Stackblitz](https://stackblitz.com/github/Rolanddoda/svg-icons-interactive-playground).

As you can see, we have a clean Vite + Vue 3 app and some SVG icons in `src/components/icons` folder.
The next step, is to install Quasar and Pinia. Before doing so, I loved how in Vue 2 we could keep `main.js` file clean and simple, so we are going to do exactly that!

First, let's create a plugins folder (`src/plugins`) and inside a `main-app.js` file:

```js
import { createApp } from 'vue'
import App from '../App.vue'

export const app = createApp(App)
```
Then, our `main.js` should look like this:

```js
import { app } from './plugins/main-app'

app.mount('#app')
```

Clean and simple right?

### Install Quasar and Pinia

First run the command:

```bash
npm install quasar @quasar/extras pinia
```
In order to make Quasar work in Vite, we need to install the appropriate plugin:

```bash
 npm install -D @quasar/vite-plugin
```
Now that we installed them, let's register them in the `plugins` folder:

*pinia.js*
```js
import { app } from './main-app'
import { createPinia } from 'pinia'

app.use(createPinia())
```

*quasar.js*
```js
import { Quasar } from 'quasar'
import { app } from './main-app'

// Import icon libraries
import '@quasar/extras/material-icons/material-icons.css'
// Import Quasar css
import 'quasar/src/css/index.sass'

app.use(Quasar, {
  plugins: {} // import Quasar plugins and add here
})
```

Finally, let's import the Quasar and Pinia plugins in the `main.js` file:

```js
import { app } from './plugins/main-app'
import './plugins/quasar' // +
import './plugins/pinia' // +

app.mount('#app')
```

*If something doesn't work on your end, see the working code [here on Github](https://github.com/Rolanddoda/svg-icons-interactive-playground/tree/lesson-1) or [online on Stackblitz](https://stackblitz.com/github/Rolanddoda/svg-icons-interactive-playground/tree/lesson-1)*

### Create reusable component for SVG icons

Now, we need to create a reusable component for SVG icons. Let's do it. 👷

We will call it `SvgIcon` and make it a global component so we can use it everywhere without importing it.

Let's create a `SvgIcon.vue` and a `contants.js` in `src/components/global/svg-icon` folder. *Inside `components/global` we will keep all of our global components*

Remember our requirements?
- our common icon sizes are 16, 24, 32 and 48. So we will call them `sm, md, lg, xl` respectively.
- Default one is 48, so that means `xl`.
- 16 and 24 have 1px stroke width, 32 has 2px, 48 has 2.5px.
- Default color is `primary`, and default hover color is `white`.

Let's define those in `contants.js` file:

```js
export const sizes = {
  sm: {
    size: 16,
    strokeWidth: 1
  },
  md: {
    size: 24,
    strokeWidth: 1
  },
  lg: {
    size: 32,
    strokeWidth: 2
  },
  xl: {
    size: 48,
    strokeWidth: 2.5
  }
}
export const defaults = {
  color: 'var:primary',
  varPrefix: 'q-',
  hoverColor: 'white',
  size: 'xl'
}
```
Quasar variables are prefixed with `q-` by default. e.g. `--q-primary`.
In order to account for that we define a `varPrefix` property in `defaults` object.

`var:primary`: `color` and `hoverColor` can either be a color value e.g. `yellow` or a variable e.g. `var:primary`. `var:primary` targets the `--q-primary` variable.

Next, let's write some code in `SvgIcon.vue` file. We will use Options API 😎:

```html
<script>
import { defineAsyncComponent } from 'vue'
import { sizes, defaults } from './constants'

export default {
  props: {
    name: {
      type: String,
      required: true
    },

    color: {
      type: String,
      default: defaults.color
    },

    size: {
      type: String,
      default: defaults.size,
      validator: (val) => Object.keys(sizes).includes(val)
    },

    hoverColor: [Boolean, String]
  },

  computed: {
    dynamicComponent() {
      const name = this.name.charAt(0).toUpperCase() + this.name.slice(1) + 'Icon'

      return defineAsyncComponent(() => import(`../../icons/${name}.vue`))
    },

    colorBind() {
      const color = this.color ? this.color : defaults.color

      return this.getVarOrColorValue(color)
    },

    hoverColorBind() {
      if (this.hoverColor === false) return

      if (this.hoverColor === true || !this.hoverColor) return defaults.hoverColor
      return this.getVarOrColorValue(this.hoverColor)
    },

    svgSize() {
      return sizes[this.size].size
    },

    strokeWidth() {
      return sizes[this.size].strokeWidth
    }
  },

  methods: {
    getVarOrColorValue(str) {
      return str.startsWith('var:') ? str.replace(/^var:/, `var(--${defaults.varPrefix}`) + ')' : str
    }
  }
}
</script>
```
*I think the component's code is straightforward, but here is some explanations:*

- `dynamicComponent`: Based on the `name` prop, we import the corresponding icon component.
- `colorBind`: if `color` prop is `falsy` use `defaults.color`, otherwise use `color`. Call `getVarOrColorValue` to return the color or the variable.
- `hoverColorBind`: if `hoverColor` prop is `false` no hover is enabled. If it's `falsy`(e.g. `undefined`) we will use `defaults.hoverColor`. Call `getVarOrColorValue` to return the color or the variable.
- `getVarOrColorValue`: if `str` is a color value it returns it as is. Otherwise, if it starts with `var:` then it returns the CSS variable. e.g. str === `var:primary` will return `var(--q-primary)` taking into account `defaults.varPrefix`.

Next, let's add the `<template>` and `<style>` tags:

```html
<template>
  <component
    :is="dynamicComponent"
    class="svg-icon"
    :width="svgSize"
    :height="svgSize"
    :stroke-width="strokeWidth"
    :class="{ 'add-hover': !!hoverColorBind }"
  />
</template>

<style lang="scss" scoped>
.svg-icon {
  color: v-bind(colorBind);
  transition: color 0.2s ease-in-out;

  &.add-hover:hover {
    color: v-bind(hoverColorBind);
  }
}
</style>
```

Not much to explain about the template but in the style we use `v-bind` to link `colorBind` and `hoverColorBind` computed properties to the CSS color property. Whenever these computed properties change, the color property will be updated. In fact, the actual value will be compiled into a hashed CSS variable. [Learn more in the docs](https://vuejs.org/api/sfc-css-features.html#v-bind-in-css).

Greatness! Here are some simple examples of using the component we just created:

```
<svg-icon name="home" />

<svg-icon name="home" size="sm" color="var:primary" hoverColor />

<svg-icon name="home" size="sm" color="var:primary" hoverColor="blue" />

<svg-icon name="home" size="sm" color="blue" hoverColor="var:primary" />
```

*See the working code [here on Github](https://github.com/Rolanddoda/svg-icons-interactive-playground/tree/lesson-2)* or [online on Stackblitz](https://stackblitz.com/github/Rolanddoda/svg-icons-interactive-playground/tree/lesson-2).*

### Auto-register components

We haven't made our `SvgIcon.vue` component global yet, so let's register all components in the `components/global` folder.
In Vite, we can do this by using [Glob import](https://vitejs.dev/guide/features.html#glob-import).

First create `plugins/global-components.js` file:

##### `import.meta.glob`

By using `import.meta.glob`, matched files are lazy loaded via dynamic import and will be split into separate chunks during build:

```js
import { defineAsyncComponent } from 'vue'
import { app } from './main-app'

const globalComponentsPaths = import.meta.glob('/src/components/global/**/*.vue')

Object.entries(globalComponentsPaths).forEach(([path, module]) => {
  // "./components/SvgIcon.vue" -> "SvgIcon"
  const componentName = path
    .split('/')
    .pop()
    .replace(/\.vue$/, '')

  app.component(componentName, defineAsyncComponent(module))
})
```

##### `import.meta.globEager`

If you want to load all matched files eagerly, you can use `import.meta.globEager`:

```js
import { app } from './main-app'

const globalComponentsPaths = import.meta.globEager('/src/components/global/**/*.vue')

Object.entries(globalComponentsPaths).forEach(([path, module]) => {
  // "./components/SvgIcon.vue" -> "SvgIcon"
  const componentName = path
    .split('/')
    .pop()
    .replace(/\.vue$/, '')

  app.component(componentName, module.default)
})
```

In our case, we don't want separate chunks since we will only have a single page, so we will use `import.meta.globEager`. This will load all components eagerly and will be included in the main bundle.

The last step is to import the `global-components.js` file in `main.js`:

```js
import { app } from './plugins/main-app'
import './plugins/quasar'
import './plugins/pinia'
import './plugins/global-components' // +

app.mount('#app')
```

