# myblog
[hexo 博客项目](https://hexo.io/zh-cn/docs/index.html)

## 使用

安装 hexo-cli
```shell
npm install -g hexo-cli
```
进入我的项目

安装依赖
```shell
npm install
```
新建文章
```shell
hexo new <post> title
```
发布
```shell
hexo d --g
# 或
hexo g # hexo generate
hexo d # hexo deploy
```
现已支持通过 github action 自动部署到 github pages。只需提交代码便可自动编译部署
