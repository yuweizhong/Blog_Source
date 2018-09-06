---
layout: '''first'
title: First Blog
date: 2017-08-21 18:42:06
tags:
---


# **hexo+GitHub**
 很早就有写博客这个想法，希望有一个个人的独立博客，最后采用了hexo+github的方式建立了这个个人博客。

## hexo
hexo是一款基于Node.js的静态博客框架，和jekyll相似，鉴于hexo以下优点，最后，使用了hexo作为博客框架
  1. 依赖少（仅依赖node），易于安装
  2. 台湾人写的，不用担心对中文支持不好
  3. 对程序员友好，要是愿意折腾还是有的折腾的

## 构建博客步骤
- 安装Git Bash
- 安装node.js
++（傻瓜式安装不多做解释）++
- 创建文件夹作仓库（例如blog目录）进入到该目录 
- 安装hexo： npm i -g hexo 
- 初始化：hexo init
### 生成目录
- [ ]   node_modules：是依赖包
- [ ]   public：存放的是生成的页面
- [ ]   scaffolds：命令生成文章等的模板
- [ ]   source：用命令创建的各种文章
- [ ]   themes：主题
- [ ]   _config.yml：整个博客的配置
- [ ]   db.json：source解析所得到的
- [ ]   package.json：项目所需模块项目的配置信息

## github
- 注册github账号（已注册的忽略）
- 创建一个repo，名称为yourname.github.io, 其中yourname是你的github名称 
- 创建完之后，用编辑器打开你的blog项目，修改_config.yml文件最后的一些配置(冒号之后都是有一个半角空格的)
deploy:
  type: git
  repo: https://github.com/YourgithubName/YourgithubName.github.io.git
  branch: master
- 回到gitbash中，进入你的blog目录，分别执行以下命令：
```
hexo clean
hexo generate
hexo server
```
打开浏览器输入：http://localhost:4000 可以查看到博客主页


## 上传github
- 先安装一波：npm install hexo-deployer-git--save（这样才能将你写好的文章部署到github服务器上并让别人浏览到）

- 执行命令(建议每次都按照如下步骤部署)：
```
hexo clean
hexo generate
hexo deploy
```
在浏览器中输入 http://yourgithubname.github.io 就可以看到你的个人博客啦

### 修改博客主页主题
网址[https://hexo.io/themes/](https://hexo.io/themes/)
github上clone到themes里面更改config配置后即可生效

### 新建文章
```
hexo new '文章名'
```
然后你就可以在source/_posts路径下看到你创建的文章啦，编辑完成之后按照前面说的方式部署，在浏览器刷新就能看到你的文章了。至于markdown，可以自行发挥！！！

---

可以说，2017年是特殊的一年，从学校步入到了社会，在近两年iOS形势大败之下能够成功入职了某大型互联网金融公司，现在看来也是不错的。当然，在仔细品味大神的博客之后，对博客也萌生了更多的想法，在这后博客时代，博客是一个单纯美好的平台，一个分享、记录的平台。动笔写博客主要是想能够充实自己
1. 积累==积累一些实在的东西，踏实的继续走下去
2. 坚持==坚持撰写博客，不断激励自己，不断进步
3. 分享==分享一些自己学习到的技术和知识
4. 记录==记录技术道路上的点滴
#### 我的博客诞生于2017年，伴随着第一份正式工作的尘埃落定。希望接下来能够坚持这个习惯，能够不断充实自己！！！












