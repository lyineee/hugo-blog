---
title: "webpack 配置文件"
date: 2022-01-26
tags: ['webpack', 'node']
---
## 基础配置

```jsx
const path = require('path');

module.exports = {
    entry: './src/index.js', //程序的入口文件
    devtool: 'inline-source-map', //使用sourcs map进行调试
    output: {
        filename: 'main.js', //打包文件输出文件名
        path: path.resolve(__dirname, 'dist'), //打包文件输出文件路径
    },
};
```

## 使用 `webpack-dev-server`

使用 `webpack-dev-server` 需要安装相关依赖 `yarn add webpack-dev-server` 并添加如下配置

```jsx
devServer: {
        contentBase: './dist', //指定服务器根路径
    }
```

## 使用`typescript`

首先要要安装webpack的扩展 `yarn add ts-loader` 和 `typescript`[TypeScript](https://www.notion.so/TypeScript-a16e95729b7b4ccd869cee89b53c9dd4) 

创建 `typescript` 的配置文件 [tsconfig文件](https://www.notion.so/tsconfig-042480867d614da8a964bffb60c8a4e3) 

在配置文件中添加下面的代码

```jsx
module: {
        rules: [
            {
                test: /\.tsx?$/,
                loader: 'ts-loader',
                exclude: /node_modules/, //排除node_modules文件夹来减少处理时间
                options: { appendTsSuffixTo: [/\.vue$/] }, //.vue文件也是用这个规则来处理
            }
        ]
  }
```

## 使用CSS

首先安装依赖 `yarn add style-loader css-loader`

在配置文件中添加下面的代码

```jsx
module: {
    rules: [
      {
        test: /\.css$/i,
        use: ['style-loader', 'css-loader'],
      },
    ],
  }
```

## 使用 SCSS

处了一般的 css 的依赖以外，使用 SCSS 还需要 scss 和 scss-loader `yarn add sass sass-loader`

💡 Sass 是使用缩进的语法，而 SCSS 是后来官方推出的全新语法，意思是 Sassy CSS，所以 SCSS 有着与 CSS 更相似的语法

与CSS类似，但处理文件的后缀 `test` 字段不同，处理的管线 `ues` 字段也不同

```jsx
module: {
    rules: [
      {
        test: /\.s[ac]ss$/i,
        use:[
          'style-loader',
          'css-loader',
          'sass-loader',
        ],
      },
    ],
  }
```

## 多配置文件

多个配置文件可以根据需要

## 文件替换

有时可能需要在编译生产环境的代码前替换环境配置文件，webpack 提供了内置的模块来实现这个需求 `NormalModuleReplacementPlugin` 。

[NormalModuleReplacementPlugin | webpack](https://webpack.js.org/plugins/normal-module-replacement-plugin/)

使用的一般形式：

```jsx
new webpack.NormalModuleReplacementPlugin(
  resourceRegExp,
  newResource
);
```

Stackoverflow 上的参考：

[How can I replace files at compile time using webpack?](https://stackoverflow.com/questions/51338574/how-can-i-replace-files-at-compile-time-using-webpack)

⚠️ 如果报错 `TypeError: webpack.NormalModuleReplacementPlugin is not a constructor` 可能是导入 webpack 包的方法出现了问题
正确：`var webpack = require('webpack');`
错误：`var { webpack } = require('webpack');`
导入内部的包时不能使用大括号？


另一个方法是 `file-replacement-loader` 这个包使用了 webpack 的 loader 来进行文件的的替换

[vyushin/file-replace-loader](https://github.com/vyushin/file-replace-loader)

以下是我的使用方法

```jsx
module.exports = merge(common, {
    mode: 'production',
    module: {
        rules: [{
            test: /envirments\/envirmnet\.js/,
            loader: 'file-replace-loader',
            options: {
                condition: 'if-replacement-exists',
                replacement: resolve('./envirment.prod.js'),
            }
        }]
    }
})
```

[一些有关 NormalModuleReplacementPlugin 的分析](../一些有关-normalmodulereplacementplugin-的分析/)
