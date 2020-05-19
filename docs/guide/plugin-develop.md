---
translateHelp: true
---

# Plug-in development


In Umi, the plugin is actually a JS module, you need to define a plugin initialization method and export by default. The following example:

```js
export default (api) => {
  // your plugin code here
};
```

It should be noted that if your plug-in needs to be published as an npm package, then you need to compile before publishing to ensure that the published code is ES5 code.。

The initialization method will receive the `api` parameter, and the interfaces provided by Umi to the plug-in are exposed through it.

## Plugin example

Let's learn more about Umi's plug-in development by completing a simple requirement

### Demand

The performance of Umi's conventional routing is the main route, which corresponds to the `index` route, that is, the page accessed by` http: // localhost: 8000` is actually `src / pages / index`. You will encounter situations in which you want to modify the main route. For example, you want to route `/` to access `src / pages / home`.

### Initialize the plugin

You can directly create a Umi plugin scaffolding via [create-umi] (https://github.com/umijs/create-umi):

```shell
yarn create umi --plugin
? Select the boilerplate type plugin
? What's the plugin name? umi-plugin-main-path
? What's your plugin used for? config umi main path
? What's your email? 448627663@qq.com
? What's your name? xiaohuoni
? Which organization is your plugin stored under github? alitajs
? Select the development language TypeScript
? Does your plugin have ui interaction(umi ui)? No
   create package.json
   create .editorconfig
   create .fatherrc.ts
   create .gitignore
   create .prettierignore
   create .prettierrc
   create CONTRIBUTING.md
   create example/.gitignore
   create example/.umirc.ts
   create example/app.jsx
   create example/app.tsx
   create example/package.json
   create example/pages/index.css
   create example/pages/index.jsx
   create example/pages/index.tsx
   create example/tsconfig.json
   create example/typing.d.ts
   create README.md
   create src/index.ts
   create test/fixtures/normal/pages/index.css
   create tsconfig.json
✨ File Generate Done
```

### Install node module

```shell
$ yarn
```

> You can also use npm install, because there are written tests, so puppetee is installed, if you fail to install, you may need to go online scientifically, or use Taobao source.

### Umi@3 plugin naming feature

In Umi @ 3, when the plugin starts with `@ umijs` or` umi-plugin`, it will be used by default as long as it is installed, so if your plugin name is named after the above rules, you do n’t need to explicitly Use your plug-in. If your plug-in naming does not meet the above rules, then you only need to display it in config.

```ts
import { defineConfig } from 'umi';

export default defineConfig({
  plugins: ['you-plugin-name'],
});
```

> In this example, our plugin name is umi-plugin-main-path.

### Combat drill

First of all, let ’s take a look at the initialization code in the scaffolding. If this plugin is used, the log `use plugin` will be printed, and then` h1` will be added to the `body` using the` modifyHTML` api. For more plugin APIs, please refer to [Plugins Api] (/ plugins / api).

```ts
export default function (api: IApi) {
  api.logger.info('use plugin');

  api.modifyHTML(($) => {
    $('body').prepend(`<h1>hello umi plugin</h1>`);
    return $;
  });

}
```

To add a configuration for our plugin, use [describe] (/ plugins / api # describe-id-string-key-string-config--default-schema-onchange--) to register the configuration.

```ts
  api.describe({
    key: 'mainPath',
    config: {
      schema(joi) {
        return joi.string();
      },
    },
  });
```

Add the main logic of our plugin

```ts
  if (api.userConfig.mainPath) {
    api.modifyRoutes((routes: any[]) => {
      return resetMainPath(routes, api.config.mainPath);
    });
  }
```

> It should be noted here that we are taking api.userConfig in judgment, and api.config is used in the callback of api. You can understand that api.userConfig is the value in the configuration, and api.config is the modified plugin The value can be modified by any plugin.

Use our plugin in the demo:

Add configuration in `example / .umirc.ts`

```ts
import { defineConfig } from 'umi';

export default defineConfig({
  plugins: [require.resolve('../lib')],
  mainPath:'/home'
});
```

Create a new page, create `example / pages / home.tsx`

```tsx
import React from 'react';
import styles from './index.css';

export default () => (
  <div className={styles.normal}>
    <h2>Home Page!</h2>
  </div>
);
```

View the effect

```shell
yarn start
Starting the development server...

✔ Webpack
  Compiled successfully in 20.73s

  App running at:
  - Local:   http://localhost:8000 (copied to clipboard)
  - Network: http://192.168.50.236:8000
```

A browser can access `home` by visiting` http: // localhost: 8000`. To access the previous `index` page, use` http: // localhost: 8000 / index`.

### Write tests for plugins

In general Umi plug-in testing, we all use the result test solution, only looking at the final running effect. Here we use [`test-umi-plugin`] (https://github.com/umijs/test-umi-plugin), it also has certain conventions, after specifying` fixtures`, he will automatically execute the folder Under the test file.

Add configuration in `test / fixtures / normal / .umirc.ts`

```ts
export default {
  plugins: [require.resolve('../../../lib')],
  mainPath: '/home'
}
```

新建 page 页面，新建 `test/fixtures/normal/pages/home.tsx`

```tsx
import React from 'react';
import styles from './index.css';

export default () => (
  <div className={styles.normal}>
    <h2>Home Page!</h2>
  </div>
);
```

Modify test case `test / fixtures / normal / test.ts`

```diff
export default async function ({ page, host }) {
  await page.goto(`${host}/`, {
    waitUntil: 'networkidle2',
  });
  const text = await page.evaluate(
    () => document.querySelector('h1').innerHTML,
  );
  expect(text).toEqual('Home Page');
};
```

Perform test

```
$ yarn test
$ umi-test
  console.log
    [normal] Running at http://localhost:12401

 PASS  test/index.e2e.ts (10.219s)
  ✓ normal (1762ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        11.057s
Ran all test suites.
✨  Done in 14.01s.
```

The complete code for this example is at [umi-plugin-main-path] (https://github.com/alitajs/umi-plugin-main-path).
