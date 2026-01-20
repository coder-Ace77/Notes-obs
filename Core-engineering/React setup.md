
---

Using vite+react

```bash
npm create vite@latest my-app -- --template react-ts
```

Installing tailwind postcss and autoprefixer

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

Edit tailwind.config.js
```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

Add this to src/index.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

CSS must be imported in main file

Run project

```bash
npm run dev
```

Set up vitest

```shell
npm install -D vitest @vitest/ui jsdom
npm install -D @testing-library/react @testing-library/jest-dom
```

Update vite.config.js

```shell
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/setupTests.ts',
  },
})
```

Create setup test
```
import '@testing-library/jest-dom'
```

Adding test scripts

```json
"scripts": {
  "test": "vitest",
  "test:ui": "vitest --ui",
  "test:watch": "vitest --watch"
}
```

