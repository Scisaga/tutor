# unocss
UnoCSS 是一个现代化、极其灵活的原子 CSS 引擎。它与传统的 CSS 框架不同，UnoCSS 采取了按需生成的方式，只有在使用时才生成相应的 CSS 样式。这使得其非常轻量级，并且具有极高的性能和灵活性。
- https://unocss.dev/
- [GitHub项目地址](https://github.com/unocss/unocss)

## uno.config.ts
```jshelllanguage
import { defineConfig } from 'unocss'

export default defineConfig({
  rules: [
    [/^m-([\.\d]+)$/, ([_, num]) => ({ margin: `${num}px` })],
  ],
  shortcuts: {
    // shortcuts to multiple utilities
    'btn': 'py-2 px-4 font-semibold rounded-lg shadow-md',
    'btn-green': 'text-white bg-green-500 hover:bg-green-700',
    // single utility alias
    'red': 'text-red-100',
  },
  theme: {
    // ...
    colors: {
      'veryCool':'#0000ff', // class="text-very-cool"
      'brand':{
        'primary':'hsl(var(--hue, 217) 78% 51%)', // class="bg-brand-primary"
      }
    }
  }
})
```

## 预设样式
- [presetAttributify](https://unocss.dev/presets/attributify) 将class标签属性化，减少前缀 class="text-sm text-white"  text="sm white"
   ```html
   <button
     bg="blue-400 hover:blue-500 dark:blue-500
     dark:hover:blue-600"
     text="sm white"
     font="mono light"
     p="y-2 x-4"
     border="2 rounded blue-200">
     Button
    </button>
    ```
   - 相同前缀进一步简化 `class="border border-red" --> border="~ red"`
- [presetIcons](https://unocss.dev/presets/icons)  https://icones.js.org/
   ```jshelllanguage
   pnpm add -D @unocss/preset-icons @iconify-json/[the-collection-you-want]
   pnpm add -D @iconify-json/mdi
   ```
- [presetTypography](https://unocss.dev/presets/typography) 预设字体
- [presetUno](https://unocss.dev/presets/uno) 预设样式，等价@unocss/preset-wind
- [presetWebFonts](https://unocss.dev/presets/web-fonts) 预设字体，支持本体保存

## transformers
- [transformerDirectives](https://unocss.dev/transformers/directives) @apply @screen theme()
- [transformerVariantGroup](https://unocss.dev/transformers/variant-group)
   ```
   class="hover:(bg-gray-400 font-medium) font-(light mono)"
   class="hover:bg-gray-400 hover:font-medium font-light font-mono"
   ```

<style>
body { counter-reset: h1counter h2counter h3counter h4counter h5counter h6counter; }

h1 { counter-reset: h2counter; }
h2 { counter-reset: h3counter; }
h3 { counter-reset: h4counter; }
h4 { counter-reset: h5counter; }
h5 { counter-reset: h6counter; }
h6 {}

h2:before {
    counter-increment: h2counter;
    content: counter(h2counter) "\0000a0";
}

h3:before {
    counter-increment: h3counter;
    content: counter(h2counter) "." counter(h3counter) "\0000a0";
}

h4:before {
    counter-increment: h4counter;
    content: counter(h2counter) "." counter(h3counter) "." counter(h4counter) "\0000a0";
}

h5:before {
    counter-increment: h5counter;
    content: counter(h2counter) "." counter(h3counter) "." counter(h4counter) "." counter(h5counter) "\0000a0";
}

h6:before {
    counter-increment: h6counter;
    content: counter(h2counter) "." counter(h3counter) "." counter(h4counter) "." counter(h5counter) "." counter(h6counter) "\0000a0";
}

pre {
    overflow: auto;
    white-space: pre-wrap !important;
    word-wrap: break-word !important;
    
    margin: .75rem 0;
    padding: .5rem;

    font-size: .875em;
    
    border: 1px solid #666;
    border-radius: 3px;
}
</style>