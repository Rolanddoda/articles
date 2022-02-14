


### Interactive demo

For this article, I have made a project using [Vue.js 3](https://vuejs.org), [Vite](https://vitejs.dev), [Quasar](https://quasar.dev) and connected the repo to [Stackblitz](https://stackblitz.com) where you can play with Controls to generate the code for the `svg-icon` component!

- See it live: [here](https://vbwippnrg--github--3000.local.webcontainer.io)
- Play with the code: [here](https://stackblitz.com/github/Rolanddoda/unified-way-of-using-svg-icons?file=src%2Fcomponents%2FSvgIcon.vue)
- Star the repo on Github: [here](https://github.com/Rolanddoda/unified-way-of-using-svg-icons)

### Introduction

Nowadays, in most cases, we end up using icon sets like Material design icons, FontAwesome, etc. Even though there are a lot of icon sets, and in most cases they cover all of our needs, in this article we are going to use pure SVG icons.

##### Requirements

Our designer has decided that the app needs its own unique icons.

- Most icons fall in 16, 24, 32, 48 sizes. However, a few icons, some special ones, have different sizes. The most used size is 48.

- SVG icons are scalable, but our smart designer noticed that the SVG doesn't look that sharp in different sizes. So the SVG, on each size, has a different stroke-width: 16 and 24 has a stroke width of 1px. For 32 is 2 and for 48 is 2.5.

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

  ####

Alright! As you can see, we need a highly customizable way to define and use SVG icons. And it got to be simple of course!

### Add SVG icons

We will put all SVG icons inside the `components/icons` folder and convert them to a Vue.js component for better management.
Let's see an example of an SVG icon component (user icon):

```vue
<template>
  <svg
    width="24"
    height="24"
    viewBox="0 0 24 24"
    fill="none"
    xmlns="http://www.w3.org/2000/svg"
  >
    <path
      d="M5 20C5 17.5 7 15.6 9.4 15.6H14.5C17 15.6 18.9 17.6 18.9 20M15 5.2C16.7 6.9 16.7 9.6 15 11.2C13.3 12.8 10.6 12.9 8.99998 11.2C7.39998 9.5 7.29998 6.8 8.99998 5.2C10.7 3.6 13.3 3.6 15 5.2Z"
      stroke="currentColor"
      stroke-linecap="round"
      stroke-linejoin="round"
    />
  </svg>
</template>
```

The process is simple:

- Export the icon from Figma as an SVG
- Create a new Vue file with the name of the icon
- Paste the copied SVG and wrap it with the `template` tag.

### Start coding 🧑‍💻

Let's start by mapping the defined sizes with the stroke-width:

```javascript
const sizes = {
  sm: {
    size: 16,
    strokeWidth: 1,
  },
  md: {
    size: 24,
    strokeWidth: 1,
  },
  lg: {
    size: 32,
    strokeWidth: 2,
  },
  xl: {
    size: 48,
    strokeWidth: 2.5,
  },
};
```

Next, let's define the defaults:

```javascript
const defaults = {
  color: "var:primary",
  hoverColor: "white",
  size: "xl",
};
```

With `var: primary` we target the CSS variable `--primary`.

`hoverColor` doesn't target any CSS variable that's why it doesn't start with `var:`.

For the default size, we choose the `xl` one.

### Init SvgIcon.vue component

Now that we have the mapped sizes and the defaults, let's create the component and add the `props`:

```vue
<script>
const sizes = {
  sm: {
    size: 16,
    strokeWidth: 1,
  },
  md: {
    size: 24,
    strokeWidth: 1,
  },
  lg: {
    size: 32,
    strokeWidth: 2,
  },
  xl: {
    size: 48,
    strokeWidth: 2.5,
  },
};
const defaults = {
  color: "var:primary",
  varPrefix: "q-",
  hoverColor: "white",
  size: "xl",
};

export default {
  props: {
    color: {
      type: String,
      default: defaults.color,
    },

    size: {
      type: String,
      default: defaults.size,
      validator: (val) => Object.keys(sizes).includes(val),
    },

    hoverColor: [Boolean, String],
  },
};
</script>

<template>
  <div class="svg-icon"></div>
</template>
```

Props:

`color`: The color of the icon, defaults to `var:primary`
`size`: The size of the icons, defaults to `xl`
`hoverColor`: to-be-explained

It might be obvious but we haven't explained why `hoverColor` color needs to be Boolean or String.
Let's explain that in the next section.

### Implement `color` and `hoverColor` bindings

Our `color` prop needs to support both a pure color value and one with a leading `var:` indicating that it is a CSS variable.
For that, we need a `computed` property and a method to transform the `var:[color]` string to a CSS variable:

```javascript
{
    //...
    computed: {
        colorBind() {
          const color = this.color ? this.color : defaults.color

          return this.getVarOrColorValue(color)
        }
    },

    methods: {
        getVarOrColorValue(str) {
          return str.startsWith("var:")
            ? str.replace(/^var:/, 'var(--') + ")"
            : str;
        },
    }
}
```

_In `colorBind()` we say: Use the color prop, if it is falsy, then use the default one. Finally, add support for CSS variable color by returning the result of `getVarOrColorValue`._
_`getVarOrColorValue`: if it starts with `var:` then convert it to a CSS variable. e.g. `var:primary` will return `var(--primary)`. Otherwise will return the string as-is._

Awesome 🎉. Now we need to do pretty much the same for `hoverColor`. The reason why this prop is a `Boolean` or a `String` is simply because:

- `hoverColor` is `false` by default. No hover is enabled in that case.
- If it's `true` or a falsy value other than `false`, then the default color gets applied on hover.
- If it's a string and non-falsy, then that color gets applied on hover.

Let's see the logic above, implemented in code:

```javascript
{
    // ...
    computed: {
        hoverColorBind() {
          if (this.hoverColor === false) return;
          if (this.hoverColor === true || !this.hoverColor) return defaults.hoverColor;

          return this.getVarOrColorValue(this.hoverColor);
        }
    }
}
```

Since we now have the `colorBind` and `hoverColorBind` we can use the awesome [v-bind in CSS](https://vuejs.org/api/sfc-css-features.html#v-bind-in-css) feature 😎 to dynamically and reactively set the color of the SVG icon:

```css
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

Also, let's change the template a little bit to make sure hover color is enabled only when `hoverColorBind` has a truthy value:

```html
<div class="svg-icon" :class="{ 'add-hover': !!hoverColorBind }" />
```

### Connect component with SVG icons

We are almost done! We took care of many requirements and now we just need to connect the SvgIcon.vue to the icon components.
To do that, we need to add a new required prop for defining the icon's name and use that to import the icon:

```vue
import { defineAsyncComponent } from 'vue' export default { //... props: { name:
{ type: String, required: true }, //... }, computed: { dynamicComponent() {
const name = this.name return defineAsyncComponent(() =>
import(`./icons/${name.toLowerCase()}.vue`)) }, //... } }
```

Look at this magic😄😄! We can use a name prop and our component will dynamically import the corresponding svg icon component.
So with the code:

```html
<svg-icon name="user" />
```

Our component will load `./icons/user.vue`.

In order to render the dynamic component we will use the [Special <component> element](https://vuejs.org/api/built-in-special-elements.html#component):

```html
<component :is="dynamicComponent" />
```

Before we see what our component looks like, let's add the `size` and `stroke` computed properties:

```js
{
    //...
    svgSize() {
      return sizes[this.size].size
    },

    strokeWidth() {
      return sizes[this.size].strokeWidth
    }
}
```

✨ Great! Now, our `SvgIcon.vue` component should look like this: ✨

```vue
<script>
import { defineAsyncComponent } from "vue";

const sizes = {
  sm: {
    size: 16,
    strokeWidth: 1,
  },
  md: {
    size: 24,
    strokeWidth: 1,
  },
  lg: {
    size: 32,
    strokeWidth: 2,
  },
  xl: {
    size: 48,
    strokeWidth: 2.5,
  },
};
const defaults = {
  color: "var:primary",
  hoverColor: "white",
  size: "xl",
};

export default {
  props: {
    name: {
      type: String,
      required: true,
    },

    color: {
      type: String,
      default: defaults.color,
    },

    size: {
      type: String,
      default: defaults.size,
      validator: (val) => Object.keys(sizes).includes(val),
    },

    hoverColor: [Boolean, String],
  },

  computed: {
    dynamicComponent() {
      const name = this.name;

      return defineAsyncComponent(() =>
        import(`./icons/${name.toLowerCase()}.vue`)
      );
    },

    colorBind() {
      const color = this.color ? this.color : defaults.color;

      return this.getVarOrColorValue(color);
    },

    hoverColorBind() {
      if (this.hoverColor === false) return;

      if (this.hoverColor === true || !this.hoverColor)
        return defaults.hoverColor;
      return this.getVarOrColorValue(this.hoverColor);
    },

    svgSize() {
      return sizes[this.size].size;
    },

    strokeWidth() {
      return sizes[this.size].strokeWidth;
    },
  },

  methods: {
    getVarOrColorValue(str) {
      return str.startsWith("var:")
        ? str.replace(/^var:/, "var(--") + ")"
        : str;
    },
  },
};
</script>

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

Simple, yet powerful! 💪

### How to use the component

If you use a lot of icons in many places, `SvgIcon.vue` can be a global component so you don't have to import it every time you want to use it.
Otherwise, you can import it as a local component.

Now we can easily specify the icon, size, color and hover color:

```html
<svg-icon name="user" size="lg" color="var:warning" hover-color />
```

Seeing the requirements, the SvgIcon component meets them all, but we can't specify a different size other than the common ones (sm, md, lg, xl).

It's true that the `size` prop is bounded to only the common sizes (intentionally) but you can still provide different sizes.
Our component is directly using the `<svg>` tag meaning that every attribute you would put in an `<svg>` in valid in `<svg-icon` component.

So let's go crazy and try a size of 50, a stroke of 3, and rotate the icon 45 degrees:

```html
<svg-icon
  name="user"
  width="50"
  height="50"
  stroke-width="3"
  transform="rotate(45)"
/>
```

The result is:

![image](https://user-images.githubusercontent.com/18482346/153548293-d3becb8c-b863-4769-87c4-a73e24db1132.png)

Amazing, isn't it? 😎😎

**Now with a single component, we can easily and in a powerful way use and customize SVG icons!**

**Thank you for reading, I hope you learned something from this article.**

Follow me on [Linkedin](https://www.linkedin.com/in/roland-doda/)
Follow me on [Github](https://github.com/Rolanddoda)
