### Summary

*This is the second part of the article. Read the first part [here](https://github.com/Rolanddoda/articles/blob/main/Unified%20SVG%20icons%20with%20Vite%2C%20Vue%203%2C%20Quasar%20and%20Pinia.md).*

In this article, we will:

- use built-in component: [Suspense](https://vuejs.org/guide/built-ins/suspense.html#suspense)
- create an interactive playground to play with the SvgIcon component
- highlight and generate the code using `Highlight.js`
- add responsive design with CSS Grid & Quasar
- add CSS Gradient rounded borders
- More usage of Quasar, Pinia and Composition API with script setup

### What are we going to build:

- [Click here to see the app what we are going to build](https://stupefied-montalcini-342150.netlify.app)
- [Play with the code online on Stackblitz](https://stackblitz.com/github/Rolanddoda/unified-way-of-using-svg-icons?file=src%2FApp.vue&terminal=dev) (you might have to run `vite` in the terminal to run the app)
- [Repo with each lesson in a branch](https://github.com/Rolanddoda/svg-icons-interactive-playground)

### Create controls and result section 

The `SvgIcon` component is customizable by props:

![image](https://user-images.githubusercontent.com/18482346/153892452-a24fa0f2-0ea3-4494-ae93-45f0a9f65aba.png)

Wouldn't be awesome to dynamically change the props of the component? Guess what? We are going to do just that! 🕺

Before we start, we need to create 2 simple global components:

AppSelect.vue
```vue
<template>
  <q-select dense dark outlined />
</template>
```
and AppInput.vue
```vue
<template>
  <q-input dense dark outlined />
</template>
```

We just need to put those 2 components inside `src/components/global` folder and our "auto-registering" we wrote in Part 1 will take care of making them global components 🪄

Now let's create the `src/components/ControlsSection.vue` component by using Composition API with script setup:

```html
<script setup>
import { useGlobalStore } from '@/stores/global-store'

const globalStore = useGlobalStore()
</script>

<template>
  <div class="controls relative-position q-pa-xl">
    <h4 class="h4 text-indigo-3 text-center">Controls</h4>

    <AppSelect label="Icon" v-model="globalStore.selectedIcon" :options="globalStore.availableIcons" />
    <AppSelect label="Size" v-model="globalStore.selectedSize" :options="globalStore.availableSizes" />
    <AppInput label="Color:" v-model="globalStore.color" hint="default value: var:primary" />

    <section class="section">
      <q-checkbox label="Enable hover color" dark dense v-model="globalStore.hasHoverColor" class="q-mb-sm" />
      <AppInput
        label="Hover color"
        v-model="globalStore.hoverColor"
        :disable="!globalStore.hasHoverColor"
        hint="default value: white"
      />
    </section>
  </div>
</template>

<style lang="scss" scoped>
.controls {
  display: grid;
  align-items: start;
  gap: 16px;
}
</style>
```

As you can see, we have connected our fields with the global Pinia store.
Now in order for the ControlsSection component to be able to change the props of SvgIcon, we need to bind the global store to its props. Since we used a component for the controls section, let's use a component for the usage of SvgIcon component with props bound to the global store:

*src/components/ResultSection.vue:*

```html
<script setup>
import { useGlobalStore } from '@/stores/global-store'

const globalStore = useGlobalStore()
</script>

<template>
  <div class="result-area">
    <div class="icon-container">
      <div class="bg"></div>
      <SvgIcon
        :name="globalStore.selectedIcon"
        :color="globalStore.color"
        :hover-color="globalStore.hasHoverColor ? globalStore.hoverColor : false"
        :size="globalStore.selectedSize"
      />
    </div>
  </div>
</template>

<style lang="scss" scoped>
.result-area {
  display: grid;
  gap: 16px;
  flex: 1;

  .icon-container {
    position: relative;
    display: grid;
    place-items: center;
    place-content: center;
    border-radius: 12px;
    padding: 32px;
    box-shadow: 0 0 15px black;

    .bg {
      position: absolute;
      inset: 0;
      border-radius: inherit;
      background: linear-gradient(135deg, rgba(66, 211, 146) 25%, #647eff);
      filter: brightness(0.5);
      opacity: 0.6;
      z-index: -1;
    }
  }
}
</style>
```
Great! Now when we change the fields in the Controls section, the props of SvgIcon reactively change. 🪄
In order to try it out, let's import and use the components we created in `App.vue`:

```html
<script setup>
import { useGlobalStore } from '@/stores/global-store'
// Components
import ResultSection from '@/components/ResultSection.vue' // ++
import ControlsSection from '@/components/ControlsSection.vue' // ++

const globalStore = useGlobalStore()
</script>

<template>
  <header>
    <div class="gradient-font q-my-sm">Unified way of using SVG Icons</div>
  </header>

  <main class="">
    <ResultSection /> <!-- ++ -->
    <ControlsSection /> <!-- ++ -->
  </main>
</template>

<style lang="scss">
@import 'css/base';

.main {
  --q-primary: v-bind('globalStore.cssVarColors.primary');
  --q-secondary: v-bind('globalStore.cssVarColors.secondary');
  --q-positive: v-bind('globalStore.cssVarColors.positive');
  --q-negative: v-bind('globalStore.cssVarColors.negative');
  --q-info: v-bind('globalStore.cssVarColors.info');
  --q-warning: v-bind('globalStore.cssVarColors.warning');

  width: 100%;
}
</style>
```

The app now should look like this:

![image](https://user-images.githubusercontent.com/18482346/154858045-edeea3bf-54bf-4de3-83af-16b53ca85e2c.png)

and be fully functional. Try to change icon, size, color, hover color and see the result.

*See the working code [here on Github](https://github.com/Rolanddoda/svg-icons-interactive-playground/tree/lesson-5) or [online on Stackblitz](https://stackblitz.com/github/Rolanddoda/svg-icons-interactive-playground/tree/lesson-5?terminal=dev).*

### Show the generated code

Now that we have the controls section, we can change how the icons looks. Let's show the generated code as well. In order to do that, let's create a computed property in 'global-store.js' before the return statement:

```js
  /* Example Output
      <svg-icon
        name="user
        color="var:primary"
        has-hover-color
      />
  */
  const generatedCode = computed(() => {
    let code = '<svg-icon'
    code += `\n name="${selectedIcon.value}"`

    if (selectedSize.value !== 'xl') {
      code += `\n size="${selectedSize.value}"`
    }

    if (color.value) {
      code += `\n color="${color.value}"`
    }

    if (hasHoverColor.value) {
      if (!hoverColor.value) {
        code += `\n hover-color`
      } else {
        code += `\n hover-color="${hoverColor.value}"`
      }
    }

    code += `\n/>`

    return code
  })
```

And return it together with the other properties:

```js
  return {
    // ...
    generatedCode
}
```

Now that we have the code, we can use Highlight.js to show it highlighted:

```html
<script setup>
import { useGlobalStore } from '@/stores/global-store'
</script>

<template>
  <highlightjs language="html" :code="globalStore.generatedCode" />
</template>
```

Here is the full code:
*src/components/CodeSnippet.vue:*

```html
<script setup>
// eslint-disable-next-line no-unused-vars
import hljs from 'highlight.js/lib/common'
import hljsVuePlugin from '@highlightjs/vue-plugin'
import { useGlobalStore } from '@/stores/global-store'

const highlightjs = hljsVuePlugin.component
const globalStore = useGlobalStore()
</script>

<template>
  <div class="container">
    <div class="code-snippet">
      <div class="shadow">
        <div class="shadow-background"></div>
      </div>

      <div class="highlightjs-container">
        <div class="snippet-header">
          <div v-for="i in 3" :key="i" class="circle"></div>
        </div>

        <highlightjs language="html" :code="globalStore.generatedCode" class="overflow-auto hide-scrollbar" />
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
// Stolen design from https://ray.so
.container {
  padding: 24px 16px;
  display: grid;
  place-items: center;
  border-radius: 12px;
  background: linear-gradient(140deg, rgb(207, 47, 152), rgb(106, 61, 236));
}

.code-snippet {
  position: relative;
  border-radius: 12px;
  min-width: 250px;
  width: 100%;
  font-size: clamp(1.1rem, 9vw - 2rem, 1.7rem);

  .shadow,
  .shadow-background {
    position: absolute;
    top: 0;
    left: 0;
    border-radius: 12px;
    height: 100%;
    width: 100%;
  }

  .shadow:after {
    position: absolute;
    content: '';
    left: 0;
    top: 24px;
    width: 100%;
    height: 100%;
    border-radius: 12px;
    background-color: rgba(0, 0, 0, 0.6);
    transform: translateZ(-1px);
    filter: blur(30px);
    z-index: -1;
  }

  .shadow-background {
    background: linear-gradient(140deg, rgb(207, 47, 152), rgb(106, 61, 236));
  }

  .highlightjs-container {
    position: relative;
    height: 100%;
    width: 100%;
    background-color: rgba(0, 0, 0, 0.75);
    border-radius: 12px;
    padding: 16px;
    transform-style: preserve-3d;
  }
}

.snippet-header {
  display: grid;
  grid-auto-flow: column;
  justify-content: start;
  gap: 8px;
  margin-bottom: 16px;

  .circle {
    width: 12px;
    height: 12px;
    border-radius: 6px;
    background-color: #fff3;
  }
}
</style>

<style lang="scss">
.hljs-tag {
  color: #6599ff;
  .hljs-name {
    color: #6599ff;
  }
}
.hljs-attr {
  color: #f8518d;
}
.hljs-string {
  color: #e9aefe;
}
</style>

```

Awesome! Now we only have to install highlight.js and the vue plugin:

```bash
npm install highlight.js
npm install @highlightjs/vue-plugin
```

Finally, we can import the `CodeSnippet` component in `App.vue` and see our code dynamically generated.

*See the working code [here on Github](https://github.com/Rolanddoda/svg-icons-interactive-playground/tree/lesson-6) or [online on Stackblitz](https://stackblitz.com/github/Rolanddoda/svg-icons-interactive-playground/tree/lesson-6?terminal=dev).*

### CSS Variables section && responsive design

We use css variables to define the color of the icon and the hover color of it. But wouldn't be nice if we could change the css variable colors? 

Since we already have `cssVarColors` in `globalStore.js` as a reactive property that's really easy to implement. We can simply create a component where we loop over the properties of `cssVarColors` and bind each property to an input "type='color'" field.

Since we use Quasar, out input can be beautiful with a built-in color picker. Let's see the code:

*src/components/VariablesSection.vue*

```html
<script setup>
import { useGlobalStore } from '@/stores/global-store'

const globalStore = useGlobalStore()
</script>

<template>
  <div class="css-vars-area relative-position q-pa-xl">
    <h4 class="h4 text-indigo-3 text-center q-mb-md">CSS Variables</h4>

    <q-input
      v-for="(colorValue, colorKey) in globalStore.cssVarColors"
      v-model="globalStore.cssVarColors[colorKey]"
      :key="colorKey"
      filled
      dark
      dense
      class="my-input q-mb-sm"
    >
      <template #prepend>
        <q-icon name="circle" :color="colorKey"></q-icon>
        <small> {{ colorKey }}:</small>
      </template>
      <template #append>
        <q-icon name="colorize" class="cursor-pointer">
          <q-popup-proxy cover transition-show="scale" transition-hide="scale">
            <q-color dark v-model="globalStore.cssVarColors[colorKey]" />
          </q-popup-proxy>
        </q-icon>
      </template>
    </q-input>
  </div>
</template>
```

Great! Now we have to import that component and use it in `App.vue`. But alongside that let's add responsive design by using CSS Grid and some help from Quasar:

*App.vue:*

```html
<script setup>
import { useGlobalStore } from '@/stores/global-store'
// Components
import ControlsSection from '@/components/ControlsSection.vue'
import CodeSnippet from '@/components/CodeSnippet.vue'
import ResultSection from '@/components/ResultSection.vue'
import VariablesSection from '@/components/VariablesSection.vue'

const globalStore = useGlobalStore()
</script>

<template>
  <header>
    <div class="gradient-font q-my-sm">Unified way of using SVG Icons</div>
  </header>

  <main class="main" :class="`screen-${$q.screen.name}`">
    <ResultSection class="result-section" style="grid-area: result" />
    <CodeSnippet class="code-snippet" style="grid-area: code" />
    <ControlsSection class="controls-section" style="grid-area: controls" />
    <VariablesSection class="variables-section" style="grid-area: variables" />
  </main>
</template>

<style lang="scss">
@import 'css/base';

.main {
  --q-primary: v-bind('globalStore.cssVarColors.primary');
  --q-secondary: v-bind('globalStore.cssVarColors.secondary');
  --q-positive: v-bind('globalStore.cssVarColors.positive');
  --q-negative: v-bind('globalStore.cssVarColors.negative');
  --q-info: v-bind('globalStore.cssVarColors.info');
  --q-warning: v-bind('globalStore.cssVarColors.warning');

  width: 100%;
  display: grid;
  grid-template-areas:
    'code'
    'result'
    'controls'
    'variables';
  gap: 12px;

  &.screen-xs,
  &.screen-sm {
    .code-snippet {
      padding: 0 4px;
    }
  }

  &.screen-md {
    display: grid;
    grid-template-columns: auto 1fr auto;
    grid-template-areas:
      'result code'
      'controls variables';
  }

  &.screen-lg,
  &.screen-xl {
    display: grid;
    grid-template-columns: 1fr minmax(500px, 1fr) 1fr;
    grid-template-areas:
      'controls code variables'
      'controls result variables';
  }
}
</style>
```

[`$q.screen`](https://quasar.dev/options/screen-plugin#introduction) plugin is used to detect screen size (`sm`, `md`, `lg` or `xl`). We use it to add classes to the `main` element, and we use CSS grid with `grid-template-columns` and `grid-template-areas` to make the grid responsive.

Simple, right?


*See the working code [here on Github](https://github.com/Rolanddoda/svg-icons-interactive-playground/tree/lesson-7) or [online on Stackblitz](https://stackblitz.com/github/Rolanddoda/svg-icons-interactive-playground/tree/lesson-7?terminal=dev).*

### Suspense and gradient borders

Resize the window of your browser to see the mobile version of the playground.
You will see that the app is responsive. In the mobile view try to change the icon and see the result area change. You will notice that the content is "jumping" when we change the icon.

In order to fix that, we will use Suspense and show the icon only when it's loaded.
To do that, open `SvgIcon.vue` and change the html code to this:

```html
  <Suspense>
    <component
      :is="dynamicComponent"
      class="svg-icon"
      :width="svgSize"
      :height="svgSize"
      :stroke-width="strokeWidth"
      :class="{ 'add-hover': !!hoverColorBind }"
    />

    <template #fallback> <q-spinner :size="svgSize" /> </template>
  </Suspense>
```

So we have wrapped the component with Suspense. We also have a fallback component, which is a spinner, and it will be shown when the icon is loading.

Awesome! 😎😎😎

Now the last things we need to do, is to add gradient borders to the "Controls" and "CSS Variables" sections.
First, go to `src/css/base.css` and add the following class:

```css
.gradient-border {
  border-radius: 12px;
  box-shadow: 0 0 5px;
  padding: 32px;

  &::before {
    content: '';
    position: absolute;
    inset: 0;
    border-radius: inherit;
    padding: 3px;
    background: linear-gradient(
                    45deg,
                    var(--q-secondary),
                    var(--q-positive),
                    var(--q-negative),
                    var(--q-info),
                    var(--q-warning)
    );
    -webkit-mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0);
    -webkit-mask-composite: xor;
    mask-composite: exclude;
  }
}
```

Now to the root element in `ControlsSection.vue` and `VariablesSection.vue` add the class `gradient-border`.

And that's it! 🥳🥳 Now we have gradient borders and the app should look like this:

![image](https://user-images.githubusercontent.com/18482346/163685992-40b53927-84db-4615-b55f-11cf2600d48b.png)


*See the working code [here on Github](https://github.com/Rolanddoda/svg-icons-interactive-playground/tree/lesson-8) or [online on Stackblitz](https://stackblitz.com/github/Rolanddoda/svg-icons-interactive-playground/tree/lesson-8?terminal=dev).*

If you liked this article, you can show your support by buying me a  coffee. It would motivate me a lot.

<a href="https://www.buymeacoffee.com/rolanddoda" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

Thanks for reading, I hope you enjoyed it!