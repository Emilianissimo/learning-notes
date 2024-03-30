# Adding SSR for Vue3 + Vite

First, we need to create:
- entry-client.js (ts)
- entry-server.js (ts)

Near the main.js file (and App.vue).

We have to remove all our content from main.js into the entry-client.js, it will handle all our client preparations and use functions (mounting and etc.).

Main.js, should be replaced with:
```js
import App from './App.vue';
import { createSSRApp } from 'vue';

export const createApp = () => {
    const app = createSSRApp(App);
    return {
        app,
    }
}
```

Here we are writing createApp function, instead of function from 'vue'.

We are running createSSRApp inside and returning this app.

After that, we have to replace:
```js
import { createApp } from 'vue'
const app = createApp(App)
```

To:

```js
import { createApp } from './main.js'
const { app } = createApp(App)
```

# Server side

Fill entry-server.js with it:
```js
import { createApp } from './main';
import { renderToString } from 'vue/server-renderer';

export const render = async () => {
    const { app } = createApp();
    const html = await renderToString(app);

    return {
        html,
    };
};
```

### Combine server and client

We have to download express
```
npm install express
```

Add to the root folder (near package.json) file ```server.js``` and fill it with:
```js
import fs from 'fs';
import path from 'path';
import express from 'express';
import { createServer } from 'vite';
import { fileURLToPath } from 'url';

const __filename = fileURLToPath(import.meta.url)
const __dirname = path.dirname(__filename)

async function initServer() {
    const app = express();

    const vite = await createServer({
        server: { middlewareMode: true },
        appType: 'custom',
    });

    app.use(vite.middlewares);
    app.use('*', async (req, res) => {
        let template = fs.readFileSync(
            path.resolve(__dirname, 'index.html'),
            'utf-8'
        );

        template = await vite.transformIndexHtml(req.originalUrl, template);

        const render = (await vite.ssrLoadModule('/src/entry-server.js')).render;

        const { html: appHtml } = await render();

        const html = template.replace('<!--main-app-->', appHtml);

        res.set({ 'Content-Type': 'text/html' }).end(html);
    });

    return app;
}

initServer().then((app) =>
    app.listen(3000, () => {
        console.log('Server is ready');
    })
);
```

### Important note: remember to add __dirname as here above in case of it would not be founded.

At lease change this line in the ```index.html```:
```html
<script type="module" src="/src/main.js"></script>
```

To:
```html
<script type="module" src="/src/entry-client.js"></script>
```

And add ```<!--main-app-->``` to the main content block:
```html
<div id="app">
    <!--main-app-->
</div>
```

And add new script to ```package.json```:
```json
  ...
  "scripts": {
    ...
    "ssr-dev": "node server.js",
    ...
  }
  ...
```

# Tips

### Custom directives
If you have custom directives, you'll not able to user them via ```app.directive('v-directive', directiveLib)```.

You'll need to transform your ```vite.config.js``` (directive-lib is an example name):

```js
import { directiveLib } from 'directive-lib'
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue({
      template: {
        ssr: true,
        compilerOptions: {
          directiveTransforms: {
            'v-directive': directiveLib
          }
        }
      }
    }),
  ],
  server: {
    host: '0.0.0.0'
  }
})
```

### Source: https://dev.to/akbarnafisa/my-first-time-implementing-ssr-using-vue-3-and-vite-e06

# More tips from me

- Remember, that you cannot access this.$prop in created! use mounted instead
- If you are using any 'document', delete it or change. For example cookies, should be replaced
- If you are using i18n and Vuex, you have to register them into entry-server.js:
  - ```js
    import { createApp } from './main';
    import { renderToString } from 'vue/server-renderer';
    import { createStore } from 'vuex'
    import { createI18n } from 'vue-i18n'
    import { messages, setI18nLanguage, getI18nLanguage } from './i18n' // may be plain data here in the file
    
    const i18n = createI18n({
        locale: 'en',
        fallbackLocale: 'en',
        messages
    })
    
    export const store = createStore()
    
    
    export const render = async () => {
        const { app } = createApp();
        
        app.use(i18n)
        app.use(store)
    
        const html = await renderToString(app);
    
        return {
            html,
        };
    };
    
    ```

### DO NOT USE VUE-ROUTER WITH SSR, IT IS ONLY FOR SPA, USE VITE_SSR_PLUGIN INSTEAD
