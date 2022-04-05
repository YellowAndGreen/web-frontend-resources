# VuePress

## 安装

1. 创建并进入一个新目录

   ```bash
   mkdir vuepress-starter && cd vuepress-starter
   ```

2. 使用你喜欢的包管理器进行初始化

   ```bash
   yarn init # npm init
   ```

3. 将 VuePress 安装为本地依赖，不推荐全局安装 VuePress

   ```bash
   yarn add -D vuepress # npm install -D vuepress
   ```

1. 在 `package.json` 中添加一些 [scripts(opens new window)](https://classic.yarnpkg.com/zh-Hans/docs/package-json#toc-scripts)

   这一步骤是可选的，但我们推荐你完成它。在下文中，我们会默认这些 scripts 已经被添加。

   ```json
   {
     "scripts": {
       "docs:dev": "vuepress dev docs",
       "docs:build": "vuepress build docs"
     }
   }
   ```

2. 在本地启动服务器

   ```bash
   yarn docs:dev # npm run docs:dev
   ```

