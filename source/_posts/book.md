---
title: hexo + github 快速搭建博客
date: 2019-12-29 21:27:08
tags: 博客
categories: 其他
top_img: https://cdn.jsdelivr.net/gh/jerryc127/CDN/img/hexo-theme-butterfly-doc-cover.jpg
cover: https://cdn.jsdelivr.net/gh/jerryc127/CDN/img/hexo-theme-butterfly-doc-cover.jpg
---

# hexo + github 快速搭建博客

话不多说，咱们直接进入主题～

## 1.新建城池

既然要给人访问，如果没有存放代码的地方，既不是白日做梦了，首先上 全球最大同性交友网站 `github` 新建一个仓库，仓库名必须为 `<user-name>.github.io`, 其中 `<user-name>` 就是我们github的昵称，
注意必须一样，不能自定义，不要问我为什么，问了我也不会说。

## 2.手握神器

打开命令行，输入下面的命令，全局安装神奇 `hexo`

```
npm install hexo -g
```

## 3. 组建大军

`hexo` 神器使用口诀如下 急急如律令，嘿嘿哈。。。sorry sorry 跑题了，口诀如下，所有口诀掐完，紧跟 `hexo s`，之后在浏览器访问 `localhost:4000` 神器搭配，手速要快。

```
hexo init // 初始化

hexo s // 本地服务启动
```

我们的文件夹下的文件,主要关注以下几个文件
- source 里面存放我们的页面和文章 _post 下都是文章，其他则为页面
- themes 里面是我们下载的主题
- public 里面是我们编译后的文件
- _config.yml 全局的一些配置

## 4.神器，城池，大军都有，该去收割一波了

虽然此时咱们只要默认的一篇 `Hello World`, 也不能阻止我们想做最靓的仔的决心，首先在根目录下 `_config.yml`找到 `deploy`字段，然后使用绝技 `copy` 大法

#### 注意点， github 记得这是 SSH key 免密登陆 就不用手动了，不会的同学下课记得补习。

```
deploy:
  type: git
  repo: <你的仓库地址> # https://github.com/14138993/14138993.github.io
  branch: master
```

施展完毕，不过我们需要喝瓶 脉动 补充下体力进行最后的操作
```
// 下载自动提交的 npm 包
npm install hexo-deployer-git --save
```

是时候向世界证明我们的存在了，下面开始公布

ps: 不想手动敲这么多命令，下方有配置教你怎么偷懒哟

```
hexo clean // 清除 pubkic

hexo g // 编译

hexo d // 提交
```

## 5. 收获成果

```
访问 https://<user-name>.github.io/ 查看效果。
```

### 课外知识点

1. 如何创建新页面，新文章？

```
hexo new post xxx(文件名) // 创建新文章

hexo new page xxx(文件名) // 创建新页面

```
这里提一下 关于tags link categories 这三个页面 需要在对应md文件里增type 这个属性 值就是本身名字 例如 

```
title: link
type: link
```

2. 更换主题皮肤

```
// 下载主题到themes文件夹下
git clone https://github.com/butterfly/hexo-theme-butterfly xxx/themes

// 修改 根目录_config.yml 配置
theme: 你的主题名称
```

3. 中文乱码

```
// 修改 根目录_config.yml 配置
language: zh-CN // 改为中文
```

4. 部署优化

国际惯例，能偷懒的绝不多写一个字母😂, 修改咱们的`package.json` 在 `script` 里增加命令
```
"dev": "hexo s", // 本地启动
"build": "hexo clean && hexo g && hexo deploy" // 一键部署到github
```


到此搭建博客已经完成，但是你可能觉得美中不足

## 优化点

### 个人域名解析到github域名

1. 上传到github后在根目录创建 `CNAME` 里面为自己的域名地址
2. 在域名服务商将域名解析 不会的请百度，这里就不细说了

### 主题优化，评论功能等

1. 直接去hexo 官网挑选喜欢的主题，一般主题都有详细的配置教程，跟着教程做
2. 我的博客使用的是 `butterfly` 如果你觉得喜欢，或者觉得功能满足你，不想自己琢磨，可以直接[Fork](https://github.com/14138993/book-config.git) 一键使用
3. 评论系统 我使用的是 [valine](https://valine.js.org/quickstart.html), 如果你是fork我的 需要在 `source > _data > butterfly.yml` 里 `valine` 配置你的 `appID` 和 `appKey`


谢谢，到此结束
