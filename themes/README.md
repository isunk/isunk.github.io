# isunk.gitee.io

## [环境搭建](https://www.diandian100.cn/58462.html)

1. 安装 Hexo
    ```bash
    npm install -g hexo
    ```

2. 初始化 Hexo
    ```bash
    cd isunk.gitee.io && hexo init
    ```

3. 下载主题
    - [ayer](https://github.com/shen-yu/hexo-theme-ayer)
        ```bash
        git clone --depth 1 https://github.com/Shen-Yu/hexo-theme-ayer.git themes/ayer
        ```
    - [next](https://github.com/theme-next/hexo-theme-next)
        ```bash
        # 参考 http://theme-next.iissnan.com/getting-started.html
        git clone https://github.com/theme-next/hexo-theme-next themes/next
        ```
    - [material](https://github.com/iblh/hexo-theme-material)
        ```bash
        # 参考 https://neko-dev.github.io/material-theme-docs/#/
        git clone -b 1.5.6 --depth 1 https://github.com/bollnh/hexo-theme-material.git themes/material
        ```

4. 修改配置文件 `_config.yml` 中主题、仓库分支
    ```yaml
    theme: material

    deploy:
      type: git
      repository: git@gitee.com:isunk/isunk.gitee.io.git
      branch: master
    ```

5. 创建一个新博客文章
    ```bash
    hexo new 'my-first-blog'
    ```
    > 博客文件见路径 `source/_posts/my-first-blog.md`

6. 生成静态文件并启动博客服务
    ```bash
    # 编译
    hexo g

    # 启动服务
    hexo s
    ```

7. 上传博客
    ```bash
    # 如果使用 hexo d 出现如下错误 “Deployer not found: gitee” 或者 “Deployer not found: git”，需安装插件如下
    # npm install hexo-deployer-git --save

    hexo d -g # 生成静态文件并上传至仓库
    ```

8. 登录 https://gitee.com/isunk/isunk.gitee.io/pages，更新 Gitee Pages 服务

9. 访问 https://isunk.gitee.io

## [目录结构](https://hexo.io/zh-cn/docs/themes.html)

.  
├── _config.yml # 主题的配置文件。和 Hexo 配置文件不同，主题配置文件修改时会自动更新，无需重启 Hexo Server。  
├── languages # 语言文件夹。请参见[国际化 (i18n)](https://hexo.io/zh-cn/docs/internationalization)。  
├── layout # 布局文件夹。用于存放主题的模板文件，决定了网站内容的呈现方式。  
├── scripts # 脚本文件夹。在启动时，Hexo 会载入此文件夹内的 JavaScript 文件。  
└── source # 资源文件夹，除了模板以外的 Asset，例如 CSS、JavaScript 文件等，都应该放在这个文件夹中。文件或文件夹开头名称为 _（下划线线）或隐藏的文件会被忽略。
