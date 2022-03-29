---
title: "一些有关 NormalModuleReplacementPlugin 的分析"
date: 2022-01-26
tags: ['webpack', 'node']
---
[](https://github.com/webpack/webpack/blob/master/lib/NormalModuleReplacementPlugin.js)

在源码中可以看到，这个插件对输入的正则表达式进行两次的匹配。

第一次是 `beforeResolve` 我的理解是在用户的程序向 webpack 请求资源时的事件。在匹配成功后会根据 `newResource` 的类型来进行不同的行为，如果 `newResource` 不是函数，则直接把 `newResource` 的值赋到 `result.request`。这个 `result.request` 暂时还不知道有什么用。如果是函数就直接把 `result` 当作参数传入函数 `newResource(result);` 。

第二次是 `afterResove` 我认为是 webpack 在请求需要的文件。同样在成功后会因 `newResource` 类型的不同有不同的行为。如果是函数则直接把 `result` 传入。而如果不是函数，会对 `newResource` 路径进行处理。

为了了解这个 `result` 的结构，我向 `newResource` 传入了一个函数在 log 中输出 `result`。

```jsx
new webpack.NormalModuleReplacementPlugin(
    /e/, // 尽可能匹配所有东西
    result=>{
        console.log(result);
    }
)
```

在控制台的输出中最终可以看到两类 `result` 的结构

```jsx
{
  contextInfo: {
    issuer: 'C:\\Users\\liuzh\\Documents\\dev\\js-test\\src\\test.js',
    issuerLayer: null,
    compiler: undefined
  },
  resolveOptions: {},
  context: 'C:\\Users\\liuzh\\Documents\\dev\\js-test\\src',
  request: '../envirments/envirment', // 正则匹配

	...
  ...

  createData: {},
  cacheable: true
}
```

```jsx
{
  contextInfo: {
    issuer: 'C:\\Users\\liuzh\\Documents\\dev\\js-test\\src\\test.js',
    issuerLayer: null,
    compiler: undefined
  },
  resolveOptions: {},
  context: 'C:\\Users\\liuzh\\Documents\\dev\\js-test\\src',
  request: '../envirments/envirment',
  
  ...
  ...

  createData: {
    layer: null,
    request: 'C:\\Users\\liuzh\\Documents\\dev\\js-test\\envirments\\envirment.js',
    userRequest: 'C:\\Users\\liuzh\\Documents\\dev\\js-test\\envirments\\envirment.js',
    rawRequest: '../envirments/envirment',
    loaders: [],
    resource: 'C:\\Users\\liuzh\\Documents\\dev\\js-test\\envirments\\envirment.js', // 正则匹配
    matchResource: undefined,
    resourceResolveData: {
      context: [Object],
      path: 'C:\\Users\\liuzh\\Documents\\dev\\js-test\\envirments\\envirment.js',
      request: undefined,
      query: '',
      fragment: '',
      module: false,
      directory: false,
      file: false,
      internal: false,
      fullySpecified: false,
      descriptionFilePath: 'C:\\Users\\liuzh\\Documents\\dev\\js-test\\package.json',
      descriptionFileData: [Object],
      descriptionFileRoot: 'C:\\Users\\liuzh\\Documents\\dev\\js-test',
      relativePath: './envirments/envirment.js',
      __innerRequest_request: undefined,
      __innerRequest_relativePath: './envirments/envirment.js',
      __innerRequest: './envirments/envirment.js'
    },
    settings: { type: 'javascript/auto' },
    type: 'javascript/auto',
    parser: JavascriptParser {
      hooks: [Object],
      sourceType: 'auto',
      scope: undefined,
      state: undefined,
      comments: undefined,
      semicolons: undefined,
      statementPath: undefined,
      prevStatement: undefined,
      currentTagData: undefined
    },
    generator: JavascriptGenerator {},
    resolveOptions: undefined
  },
  cacheable: true
}
```

可以看出第一种结构的 `result` 中没有 `createData` 中的数据，而 `createData` 中的数据主要是本地文件的绝对路径，所以可以推测后一种 `result` 是在 webpack 向系统请求文件时产生的，而前一种是用户代码在向 webpack 请求资源时产生的。

> 💡 在 plugin 的源代码中用来和用户设置的正则表达式匹配的两个变量是 `createData.resource` 和 `createData.result.request`。而从result的数据中可以看到前者使用的是本地计算机的绝对路径，使用的路径分隔符也是更具本地的系统来决定的，所以在 windows 上使用的是 `\\`  ，而后者使用的是在用户代码中引入文件时的路径，这是一般是 linux 的分隔符 `/` 。在设置正则表达式时要注意这一点

## 结论

最后我是用的代码

```jsx
new webpack.NormalModuleReplacementPlugin(
            /envirment\.js/,
            'envirment.prod.js'
        )
```

正则表达式匹配的是 webpack 向系统请求资源时的 `result` 因为用户代码在请求资源时不会有 `.js` 后缀。此时如果后面参数是一个相对路径，插件会自动把这个路径转换为以匹配到的资源的文件夹的绝对路径，也就是说这个 envirment.prod.js 文件和 envirment.js 文件在同一个文件夹中。

[NormalModuleReplacementPlugin does not work when modifying resourceRegExp slightly](https://stackoverflow.com/a/56751687/11160506)