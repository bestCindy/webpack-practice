# webpack-practice
Some practices of webpack

## tree-shaking

在 webpack 中打包模式分两种，一个是开发模式，一个是生产模式

我们可以设置用开发模式打包，也可以设置用生产模式打包，只需要在 `package.json` 文件里面设置即可

```
"scripts": {
    "dev": "webpack --mode development",
    "prod": "webpack --mode production"
  },
```
然后使用 `npm run dev` 或者 `npm run prod` 就可以用开发模式或者生产模式打包

现在我们新建两个文件 `a.js` 和 `b.js`,在 `a.js` 里面定义 `a` 函数并输出 `aaaaaaaaaa`,在 `b.js` 里面定义 `b` 函数并输出 `bbbbbbbbbb`。然后在 `index.js` 文件里面将 `a` 和 `b` 引入，但是只执行 `a` 函数

- 执行 `npm run dev`

我们会发现，即使没有执行 `b` 函数，但是 `a` 和 `b` 都被完整的打包到 `main.js` 里面

- 现在在执行 `npm run prod`

这个时候 `main.js` 文件里面只把 `a` 函数打包了进去，并没有把 `b` 打包进去

所以，在生产环境打包的时候，会把没用的文件过滤掉

把不用的文件过滤掉，只打包项目中用到的模块，这个就是 js 的 tree shaking

tree shaking 可以有效减少代码体积，提高页面加载时间

## tree-shaking 原理
tree-shaking 是 DCE 的一种新的实现

**DCE(dead code elimination)**

编译器会判断**哪些代码不影响输出**，然后消除这些代码

**注意：** tree-shaking 和传统的 DCE 方法不太一样，传统的 DCE 消灭**不可能执行**的代码，而 tree-shaking 更关注消除**没有用到**的代码

**js 的 DCE**

在编译型语言中，都是由编译器将 dead code 从 AST 中删除

关于 js 的 DCE，在 js 代码被送到浏览器之前已经做好了 DCE，这个工作是由代码压缩优化工具 uglify 完成的

## tree-shaking 的基础

tree-shaking 的消除原理是依赖于 ES6 的模块特性：
ES6 模块的依赖关系是确定的，和运行时的状态无关，可以不执行代码，从字面量上对代码进行分析

## sideEffects

`sideEffects` 是副作用的意思，默认值是 `true`,在 `package.json` 里面把 `sideEffects` 设置为 `false`，可以告诉 webpack，当前的包**期望**是无副作用的，让 webpack 在 tree shaking 的时候当成无副作用的包就好

举个例子：

现在新增一个 `collect.js`，在这个文件中把 `a.js` 和 `b.js` 里面的内容全部 export 出去，此时 `b.js` 里面不只有一个 `b` 函数，还包括一个**副作用语句** `console.log("======b.js======")`

```
export * from './a';
export * from './b';
```

然后在 `index.js` 里面引入 `a` 函数并执行

此时执行 `npm run prod`，虽然 `b` 函数被 tree shaking 掉了，但是 `console.log("======b.js======")`，仍然存在 `main.js` 中

现在把 `sideEffects` 设置为 `false`，执行 `npm run prod`，此时 `console.log("======b.js======")` 和 `b` 函数都已经被 tree shaking 掉了

可以看出 `sideEffects` 进一步优化了打包体积

**注意**：
`sideEffects` 除了能设置 `boolean` 值, 还可以设置为数组, 传递需要保留副作用的代码文件(例如: "./src/polyfill.js") 或者传递模糊匹配符(例如: "src/**/*.css")

```
sideEffects: boolean | string[]
```

>参考：
https://github.com/yinxin630/blog/issues/23




