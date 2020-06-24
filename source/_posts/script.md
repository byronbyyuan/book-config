---
title: npm script自动化工作流（基础篇）
date: 2019-12-30 20:22:41
tags: 前端自动化
categories: 
- 前端
- npm
top_img: http://www.ruanyifeng.com/blogimg/asset/2016/bg2016101101.jpg
cover: http://www.ruanyifeng.com/blogimg/asset/2016/bg2016101101.jpg
description: 1231231231312321123123
---

# npm script自动化工作流（基础篇）

## 前言

首先作为一个前端工程师，对npm 肯定不陌生，平常工作中用到的一些工具都会在npm上下载，而所有的npm 插件都会有 `package.json` 这个文件，文件里会记录整个包的信息，依赖，配置等，而其中script配置项是可以配置一些自动化命令，例如我们常用的在项目中 可能为用到像 `npm run dev｜build`这种命令，然后会自动化的去实现一些操作，记得初次接触到这个的时候，觉得很强大，经过去仔细了解了下,npm run 实际上是 npm run-script 命令的简写，那么当我们运行 `npm run xxx`，内部运行流程是这样的：

>
1. 从 package.json 文件中读取 scripts 对象里面的全部配置;
2. 以传给 npm run 的第一个参数作为键，比如 dev，在 scripts 对象里面获取对应的值作为接下来要执行的命令，如果没找到直接报错;
3. 在系统默认的 shell 中执行上述命令。


## 基础配置

---

一般我们在取配置一些script 命令的时候，例如给项目增加 eslint 自动检查的配置：

```
"scripts": {
  "lint": "eslint *.js"
}
```

随后运行 `npm run lint` 之后就会取 lint 这句命令自动执行 eslint 进行规则匹配操作。

有没有好奇过上面的 eslint 命令是从哪里来的？其实，npm 在执行指定 script 之前会把 node_modules/.bin 加到环境变量 $PATH 的前面，这意味着任何内含可执行文件的 npm 依赖都可以在 npm script 中直接调用，换句话说，你不需要在 npm script 中加上可执行文件的完整路径，比如 `./node_modules/.bin/eslint **.js`。


## 项目多个命令如何同时启动？

---

对于一个完整的项目来说 一般 script 中命令都是多个的, 比如还有对 css json 单元测试 运行 打包等命令，那比如下面配置对这么多命令难道要一个个去执行吗？ 那也太不自动化了。

"scripts": {
  "lint:js": "eslint *.js",
  "lint:css": "stylelint *.less",
  "build": "node --max-old-space-size=6144 build/build.js",
  "test": "mocha tests/"
}

比如我们现在需要在执行 build 命令的之前将代码进行检查并测试，那么我们需要利用 && 串行的命令来操作，将命令增加为

```
"scripts": {
  "lint": "eslint *.js",
  "build": "node --max-old-space-size=6144 build/build.js",
  "test": "mocha tests/
  "build:all" "npm run lint && npm run test && npm run build"
}

```

之后直接执行 `npm run bulid:all` 来自动执行多个命令，需要注意的是，串行执行的时候如果前序命令失败（通常进程退出码非0），后续全部命令都会终止，也就是说在运行命令过程中 任何一条命令结果出错，后续命令都不再执行。

## 有串行那有没有并行？

---

在严格串行过程中必须保证每一步命令都不出错才能正常运行完，但有时候我们可能只是想要得到某些命令执行的结果，比如我们仅仅需要测试结果和代码规范结果，这时我们需要将串行命令改为并行 `&` 命令进行改造：

```
"scripts": {
  "lint": "eslint *.js",
  "build": "node --max-old-space-size=6144 build/build.js",
  "test": "mocha tests/ & npm run lint & wait"
  "build:all" "npm run lint && npm run test && npm run build"
}

```


之后运行 `npm run test` 命令后 会相继运行 测试及代码检查 而不会因为某个出错终止。

⚠️ `& wait `命令跟并行没有直接关系，因为npm 在运行多个异步任务的时候，任何子命令中启动了长时间运行的进程，比如启用了 mocha 的 –watch 配置，可以使用 ctrl + c 来结束进程，如果没加的话，你就没办法直接结束启动到后台的进程。

## 除了操作符，有没有更好的方式？

----

如果不喜欢用原生的操作符，可以使用npm 插件包 `npm-run-all` 具体使用方式可以去 [npm](https://www.npmjs.com/package/npm-run-all) 查看

## script 参数

---

在我们使用 eslint 规则时，往往需要在检测的时候运行修复操作 而eslint 提供了 fix 命令来将一些能修复的语法修复

```
"scripts": {
  "lint": "eslint --fix --quit *.js "
}
```
script 参数传递 通过 `--xx` 来操作 多个参数空格分开

## script 钩子

---

npm script 的设计者为命令的执行增加了类似生命周期的机制，具体来说就是 `pre` 和 `post` 钩子脚本。这种特性在某些操作前需要做检查、某些操作后需要做清理的情况下非常有用。

举例来说，运行 `npm run test` 的时候，分 3 个阶段：

1. 检查 scripts 对象中是否存在 pretest 命令，如果有，先执行该命令；
2. 检查是否有 test 命令，有的话运行 test 命令，没有的话报错；
3. 检查是否存在 posttest 命令，如果有，执行 posttest 命令；

那么之前我们使用 `&&` 命令 运行项目对测试，代码规范命令操作，我们可以使用钩子定义。

```
"scripts": {
  "lint": "eslint *.js",
  "build": "node --max-old-space-size=6144 build/build.js",
  pretest: 'npm run lint'
  "test": "mocha tests/
  posttest: 'xx'
}

```

当我们在运行 `npm run test` 时会先将 lint 命令执行 然后执行 test, 而 posttest 命令最后执行 我们可以用 posttest 命令进行将检测结果进行写文件或者生成html展示,
posttest 命令信息收集展示

```
// nyc 信息收集展示依赖包
// opn-cli 打开命令工具
npm i nyc opn-cli -D

```

增加 cover 命令后 运行顺序 `npm run cover > npm run precover > npm run test > npm run lint > npm run postcover` 执行完毕将会把信息收集并生成 html 自动打开

```
"scripts": {
  "lint": "eslint *.js",
  "build": "node --max-old-space-size=6144 build/build.js",
  pretest: 'npm run lint'
  "test": "mocha tests/
  "precover": "rm -rf coverage",
  "cover": "nyc --reporter=html npm test",
  "postcover": "rm -rf .nyc_output && open-cli coverage/index.html"    
}

```

⚠️ rm rf 命令为linux 下 window 替换为 rimraf

---

基础篇到此结束，后续再根据学习进一步更新。