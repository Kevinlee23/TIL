### Quick setup

#### 1. install

```cmd
pnpm add --save-dev @nuxtjs/tailwindcss

yarn add --dev @nuxtjs/tailwindcss

npm install --save-dev @nuxtjs/tailwindcss
```

#### 2. edit tailwind.config.{js, ts} and import tailwind { base, components and utilities }

`tailwind.config`

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./components/**/*.{js,vue,ts}",
    "./layouts/**/*.vue",
    "./pages/**/*.vue",
    "./plugins/**/*.{js,ts}",
    "./app.vue",
    "./error.vue",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

`main.css`

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

#### 3. edit nuxt.config.{js, ts}

```json
{
  "modules": ["@nuxtjs/tailwindcss"],
  "css": ["~/assets/css/main.css"],
  "postcss": {
    "plugins": {
      "tailwindcss": {},
      "autoprefixer": {}
    }
  }
}
```

#### run your project and use tailwindcss!
