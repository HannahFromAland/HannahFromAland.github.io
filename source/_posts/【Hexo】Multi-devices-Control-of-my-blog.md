---
title: 【Hexo】Multi-devices Control of my blog
date: 2021-09-05 09:48:40
tags: Hexo
category: Hexo Optimization
---

## 使用github分支对Hexo实现多设备管理

试图对hexo进行双设备的更新和同步，（在找教程的时候遇到许多和我排版一致的伙伴们x），记录一下自己实践的过程（同时，本文也是来自新设备的更新测试v1:hand:

<!-- more -->

### Hexo项目的目录结构

```
HEXO
├──.deploy_git/
├──node_modules/
├──public/
├──scaffolds/
├──source/
├──themes/
├──_config.yml
├──.gitignore
├──db.json
├──debug.log
└──package.json
```

本地的文件是这样的，但hexo deploy上传的github端的文件只有`.deploy_git/`文件夹。具体解释如下：

> ithub pages托管的只是由hexo生成的静态文件，hexo的全部的框架和环境配置文件是只保留在本地的。

因此要实现多设备博客同步管理，只需要重新创建仓库/分支将所有的hexo文件都利用github进行管理即可～

为了美观和一致性，选择在github原有的master分支外新建hexo分支，之后master分支的内容作为生成博客的静态文件（由hexo deploy命令自己管理和实现），hexo分支的内容就是一个单纯可以实现多设备同步和管理的仓库了:beers:

具体步骤如下：

- 基于master分支创建新的hexo分支并将其设为default（方便clone和“推拉”（逃

- 将本地文件放在hexo分支内：`git clone`该分支到本地后，将内部的`.git/`文件夹直接移动到**原来的本地文件内部**（在我这里是`blog`文件夹的内部）

  - Notice：`.git/`目录即为记录该目录作为本地git分支的标志（如果在git目录中删除该文件夹就会与git失去联系，因此移动到原有的博客文件夹即将hexo分支内容变更为本地所有文件的内容
  - 由于git仓库不能嵌套上传，需要进入theme下的主题内部同样删掉`.git/`文件夹

- 之后利用git命令上传本地文件到hexo分支即可～

### 新环境的部署、编辑和同步

- 不同设备更新和修改博客内容之前，需要通过`git pull`拉取hexo内容以保证版本一致

- 更新博客后需要先提交源文件到hexo分支

  ```bash
  git add . 
  git commit -m "new post" 
  git push origin hexo
  ```

- 之后再依赖本机环境进行`hexo d`的上传操作



Reference：

[hexo博客不同设备进行同步更新](https://youngz.top/posts/53714/)

[hexo博客同步管理及迁移](https://www.jianshu.com/p/fceaf373d797)

