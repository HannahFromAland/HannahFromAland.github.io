---
title: Hello World
tags: Hexo
date: 2020-10-30 09:48:40
category: Hexo Optimization
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)

## 不断摸索的Hexo配置（持续更新）

### 修改Post内部代码段字体大小

在`_config.yml`中直接修改fonts参数发现没有变化，直接修改样式文件
`\source\css\_common\scaffolding\highlight\highlight.styl`

```stylus
$code-block
  background: highlight-background
  margin: 20px 0
  padding: 15px
  overflow: auto
  font-size 13px    // 改这里
  color: highlight-foreground
  line-height:  $line-height-code-block
```

deploy之后生效.

## 设置阅读全文

- 在文中加入`<!-- more -->`手动进行截断