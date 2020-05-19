---
translateHelp: true
---

# How to speed up compilation


If you encounter slow compilation, slow incremental compilation, memory burst, OOM, etc., you can try the following methods.

## Configure `nodeModulesTransform` to` {type: 'none'} `

> 需要 Umi 3.1 。

Umi compiles the files under node \ _modules by default, which brings some benefits and also adds extra compilation time. If you don't want the files under node \ _modules to be compiled by babel, you can reduce the compilation time by 40% to 60% through the following configuration.

```js
export default {
  nodeModulesTransform: {
    type: 'none',
    exclude: [],
  },
}
```

## View package structure

When executing `umi dev` or` umi build`, increase the environment variable `ANALYZE = 1` to view the dependency ratio of the product.

<img src="https://img.alicdn.com/tfs/TB1P_iYDQL0gK0jSZFAXXcA9pXa-2432-1276.png" width="600" />

Note:

* `umi dev` can be modified and viewed in real time, but will introduce some development dependencies, pay attention to ignore

## Configure externals

For some large size dependencies, such as chart library, antd, etc., you can try to introduce related umd files through the configuration of externals to reduce compilation consumption.

For example react and react-dom:

```js
export default {
  // Configure external
  externals: {
    'react': 'window.React',
    'react-dom': 'window.ReactDOM',
  },

  // Import scripts from external libraries
  // Distinguish between development and production, use different products
  scripts: process.env.NODE_ENV === 'development' ? [
    'https://gw.alipayobjects.com/os/lib/react/16.13.1/umd/react.development.js',
    'https://gw.alipayobjects.com/os/lib/react-dom/16.13.1/umd/react-dom.development.js',
  ] : [
    'https://gw.alipayobjects.com/os/lib/react/16.13.1/umd/react.production.min.js',
    'https://gw.alipayobjects.com/os/lib/react-dom/16.13.1/umd/react-dom.production.min.js',
  ],
}
```

Note:

1. If you want to support IE10 or below, external react also needs to introduce additional patches, such as `https://gw.alipayobjects.com/os/lib/alipay/react16-map-set-polyfill/1.0.2/dist/react16-map -set-polyfill.min.js`
2. If external antd, external extra moment, react and react-dom need to be external at the same time and introduced before antd

## Reduce patch size

Umi will include patches for the following browsers and their versions by default,

```
chrome: 49,
firefox: 64,
safari: 10,
edge: 13,
ios: 10,
```

Choose the appropriate browser version, you can reduce a lot of size, such as the following, it is expected to reduce the size of 90K (compressed without gzip).

```js
export default {
  targets: {
    chrome: 79,
    firefox: false,
    safari: false,
    edge: false,
    ios: false,
  },
}
```

Note:

* Setting the browser to false will not include his patch

## Adjust splitChunks strategy to reduce overall size

If you open dynamicImport, then the product is particularly large, each export file contains the same dependencies, such as antd, you can try to adjust the extraction strategy of public dependencies through splitChunks configuration.

Such as:

```js
export default {
  chunks: ['vendors', 'umi'],
  chainWebpack: function (config, { webpack }) {
    config.merge({
      optimization: {
        minimize: true,
        splitChunks: {
          chunks: 'all',
          minSize: 30000,
          minChunks: 3,
          automaticNameDelimiter: '.',
          cacheGroups: {
            vendor: {
              name: 'vendors',
              test({ resource }) {
                return /[\\/]node_modules[\\/]/.test(resource);
              },
              priority: 10,
            },
          },
        },
      }
    });
  },
}
```

## Set memory limit via NODE \ _OPTIONS

If OOM appears, you can also try to solve it by increasing the memory limit. For example, `NODE_OPTIONS =-max_old_space_size = 4096` is set to 4G.

## Adjust SourceMap generation method

If dev is slow or incremental compilation is slow after modifying the code, you can try to disable SourceMap generation or adjust to a lower cost generation method,

```js
// Disable sourcemap
export default {
  devtool: false,
};
```

or，

```js
// Use the lowest cost sourcemap generation method, the default is cheap-module-source-map
export default {
  devtool: 'eval',
};
```

## monaco-editor packaging

The editor is packaged, it is recommended to use the following configuration to avoid construction errors:

```js
export default {
  chainWebpack: (config) => {
    config.plugin('monaco-editor-webpack-plugin').use(
      // More configuration https://github.com/Microsoft/monaco-editor-webpack-plugin#options
      new MonacoWebpackPlugin(),
    );
    config
    .plugin('d1-ignore')
      .use(
        // eslint-disable-next-line
        require('webpack/lib/IgnorePlugin'), [
          /^((fs)|(path)|(os)|(crypto)|(source-map-support))$/, /vs(\/|\\)language(\/|\\)typescript(\/|\\)lib/
        ]
      )
    .end()
    .plugin('d1-replace')
      .use(
        // eslint-disable-next-line
        require('webpack/lib/ContextReplacementPlugin'),
        [
          /monaco-editor(\\|\/)esm(\\|\/)vs(\\|\/)editor(\\|\/)common(\\|\/)services/,
          __dirname,
        ]
      )
    return config;
  }
}
```

## Replace the compressor with esbuild

> The experimental function may have pits, but the effect is outstanding.

Taking a project that relies on the full amount of antd and bizcharts as an example, the test was conducted on the basis of disabling the Babel cache and Terser cache, and the effect is shown in the figure:

![](https://cdn.nlark.com/yuque/0/2020/png/86025/1588847300475-07a8dcaa-c712-4e5b-b244-367b3e0d61ca.png)

Install dependencies first,

```bash
$ yarn add @umijs/plugin-esbuild
```

Then open it in the configuration,

```js
export default {
  esbuild: {},
}
```

## No compression

> Not recommended, use in emergencies.

The compression time accounts for most of the slow compilation, so if you do not compress when compiling, you can save a lot of time and memory consumption, but the size will increase a lot. The compression can be skipped by the environment variable `COMPRESS = none`.

```bash
$ COMPRESS=none umi build
```
