# [nap-blog 网站](http://blog.nxap.org)

## 安装hexo

hexo基于[nodejs](https://nodejs.org)，请先安装nodejs和npm
安装hexo：`npm install hexo-cli -g`

## 从github获取代码

`git clone https://github.com/icsnju/nap-blog.git`

## 安装hexo的组件

`cd nap-blog`
`npm install`

## 添加、修改内容

使用`hexo new "article title"` 创建新文章，或编辑`source\_posts\`目录下已有的`.md`文件

## 发布内容

使用`hexo generate` 命令重新生成网页内容，并使用`hexo deploy`命令发布生产的内容到网站。

## 提交更新内容

+ 按git流程commit变更
+ 执行`hexo d -g`，hexo生成静态html页，并将其push到`gh-pages`分支上去。
