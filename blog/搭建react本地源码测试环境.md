### 搭建react本地源码测试环境

>本文讲解了如何下载最新的react源代码、以及对其测试使用、更改调试等，从而方便react源代码阅读者边测试边阅读代码。

*注意：react 源代码变动频繁，因此本文内容可能不会完全适用，因此欢迎在[这里](https://github.com/aircloud/react-source-learn/issues)提问进行交流，一般我会在 36 小时内回复*

我们在阅读框架源代码的过程中，如果源代码比较复杂，特别是逻辑分支较多，最好的办法就是边阅读代码边通过一些demo尝试，这就要求我们在本地搭建react本地源码测试环境，本文即会对该过程进行一个详细的介绍。

首先需要确保本地已经全局安装了如下内容

* yarn
* create-react-app

首先，我们需要先从[项目官方地址](https://github.com/facebook/react)clone下源代码，或者先fork到自己的账户下然后下载到本地。

然后，我们使用 `create-react-app` 来创建一个 DEMO 程序，用于测试我们的 react 源代码：

```
create-react-app react-source-learn-demo
```

然后，我们把之前下载好的源代码整个目录拷贝到我们新建的 DEMO 的目录下，这个时候，我们的 DEMO 目录结构应该是如下的：

```
|- README.md
|- node_modules
|- public
|- react
|- src
|- package.json
```

接下来，我们创建 `symbol link`，首先我们需要删除之前已经下载好的`react` 和 `react-dom`:

```
yarn remove react react-dom
```

然后在 `package.json` 粘贴这几行：

```
"react": "link:./react/packages/react/",
"react-dom": "link:./react/packages/react-dom/",
"react-dom": "link:./react/packages/react-dom/",
"shared": "link:./react/packages/shared/",
"events": "link:./react/packages/events/",
"react-reconciler": "link:./react/packages/react-reconciler/",
"react-scheduler": "link:./react/packages/react-scheduler/",
"react-is": "link:./react/packages/react-is/",
```

然后我们再运行一遍 `yarn`：

```
yarn
```

然后我们需要修改 `webpack` 配置，所以我们需要先运行：

```
yarn run eject
```

首先的一件事情是，由于 react 源码中采用了一种 flow 类型检查（可以理解为一种阉割版本的 Typescript），直接引入 js 会报错，因此我们需要把 flow 注释掉，需要安装一个 babel 插件：

```
yarn add babel-plugin-transform-flow-comments
```

之后新建一个 `.babelrc` 并且写入如下内容：

```
{
    "presets": ["react-app"],
    "plugins": ["transform-flow-comments"]
}
```

然后，由于在当前的 webpack 配置中，默认跳过 node_modules 的处理，对我们来说这样显然是不行的，所以我们需要修改一下配置：

```
{
        test: /\.(js|jsx|mjs)$/,
        include: [
          paths.appSrc,
          path.resolve(__dirname, '../react'),
        ],
        loader: require.resolve('babel-loader'),
        options: {
          
          // This is a feature of `babel-loader` for webpack (not Babel itself).
          // It enables caching results in ./node_modules/.cache/babel-loader/
          // directory for faster rebuilds.
          cacheDirectory: true,
        },
},
```
另外，由于 react 中有设置一些全局变量，我们需要在 webpack 中对其进行转换，所以我们可以在 `config/env.js` 中加入一些全局变量，对`stringified` 定义的地方进行修改：

```
const stringified = {
    'process.env': Object.keys(raw).reduce((env, key) => {
      env[key] = JSON.stringify(raw[key]);
      return env;
    }, {}),
    __DEV__: true,
    __PROFILE__: true,
};
```

最后一点，如果这个时候我们直接运行 `yarn start`，大概率会报错，经过尝试，我发现还需要做下面一件事情：

将 `react/packages/react-reconciler/src/forks/ReactFiberHostConfig.dom.js` 中的内容复制到 `/Users/hh/LTS/react-source-learn-demo/react/packages/react-reconciler/src/ReactFiberHostConfig.js` 中去，并删掉后者之前的内容。


这个时候实际上我们已经改好了，为了测试，我们可以改动一下 `/Users/hh/LTS/react-source-learn-demo/react/packages/react/src/ReactBaseClasses.js` 中的内容:

```
function Component(props, context, updater) {
  console.log("Component Init")
  this.props = props;
  this.context = context;
  this.refs = emptyObject;
  // We initialize the default updater but the real one gets injected by the
  // renderer.
  this.updater = updater || ReactNoopUpdateQueue;
}
```

改动并保存，这个时候我们发现会触发热更新，并且控制台会打出相应的内容。

至此，我们的 react 本次源代码测试环境已经搭建完成，另外，这个环境实际上只是用于学习、调试 react 源代码，至于为其贡献内容，则有其他的工作流，这方面我也在研究，也欢迎和我交流。

完。

---

> 系列文章地址：[react-source-learn](https://github.com/aircloud/react-source-learn)

