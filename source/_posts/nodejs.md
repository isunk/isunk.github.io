---
title: Nodejs
tags: nodejs
categories: nodejs
---

# Nodejs

## 搭建环境

- 使用仓库安装  
    `apt-get install nodejs`

- 从官网下载安装
    1. 下载并解压  
        `cd /opt`  
        `curl -O https://nodejs.org/dist/v16.15.0/node-v16.15.0-linux-x64.tar.xz`  
        `tar -xvf node-v16.15.0-linux-x64.tar.xz`
    2. 设置环境变量  
        - 设置全局环境变量  
            `echo 'export PATH="$PATH:/opt/node-v16.15.0-linux-x64/bin"' >> /etc/profile`
        - ~~设置临时环境变量  
            `export PATH="$PATH:/opt/node-v16.5.0-linux-x64/bin"`~~
    3. 注销并重新登录后即可生效  
        `logout`
    4. 安装coffee script（可选）  
        `npm install -g coffee-script`

## 包管理工具

- cnpm（使用国内淘宝源，速度较快）
    - 安装 cnpm  
        `npm install -g cnpm --registry=https://registry.npm.taobao.org`
    - 使用 cnpm 代替 npm，如  
        `cnpm install -g typescript`

- yarn
    - 安装  
        `npm install -g yarn`
    - 配置淘宝源，用于加速  
        `yarn config set registry 'https://registry.npm.taobao.org'`

- npm
    - 配置
        - 通过 config 命令
            - 查看当前配置内容  
                ~~`npm config ls`~~  
                `npm config list`
            - 设置远程仓库地址（使用 taobao 源镜像加速）  
                `npm config set registry https://registry.npm.taobao.org`
            - 设置 npm 安装 package 的全局 node_modules 路径  
                `npm config set prefix "E:\nodejs"`
            - 设置代理服务器  
                `npm config set proxy http://username:password@server:port`  
                `npm config set https-proxy http://username:password@server:port`
            - 设置不校验 SSL 证书  
                `npm config set strict-ssl false`
        - 通过编辑配置文件 ~/.npmrc（或 nodejs 安装目录下的 node_modules/npm/.npmrc 或 node_modules/npm/npmrc 文件）
            ```
            # 设置远程仓库地址
            # registry = https://registry.npm.taobao.org

            # 设置 npm 安装 package 的全局 node_modules 路径
            prefix = E:\nodejs
            ```
    - 从 npm 默认远程仓库下载安装 uglify-js 到全局 node_modules 路径下（参数 "-g" 表示下载到全局 node_modules 路径下并安装到 node_modules 的上一级目录，这里即 nodejs 安装目录下）  
        `npm install uglify-js -g`
        > 从 npm 默认远程仓库下载安装 uglify-js 到当前路径下的 node_modules 目录中  
        > `npm install uglify-js`  
        > 从 npm 远程仓库 "--registry https://registry.npm.taobao.org" 下载安装 uglify-js  
        > `npm install uglify-js --registry https://registry.npm.taobao.org`

    - 在当前目录下，初始化一个 node 工程  
        `npm init`
    
    - 安装依赖（package.json 中定义的依赖）  
        `npm install`  
        安装 [express](http://www.expressjs.com.cn/) 并将依赖添加至 package.json 中  
        `npm install express --save`
    
    - 自定义运行
        1. 编辑 package.json 如下
            ```json
            "scripts": {
                "start": "electron ."
            }
            ```
        2. 运行上一步骤中的 start 命令  
            `npm run start`

## 第三方 nodejs 应用

- 快速搭建本地 http 服务器
    ```bash
    # 安装
    npm i -g http-server

    # 在当前目录下启动 http 服务器
    http-server .

    # 在当前目录下启动 https 服务器
    http-server \
        -p 443 \ # 要使用的端口（默认为8080）
        -S \ # 启用 https
        -C server.crt \ # 证书文件的路径（默认值：cert.pem）
        -K server.key \ # 密钥文件的路径（默认值：key.pem）
        --cors \ # 通过 Access-Control-Allow-Origin 标题启用 CORS
        .
    ```

- 使用 uglify-js 压缩 js 文件
    1. 安装 uglify-js  
        `npm install -g uglify-js`
    2. ~~压缩 js 文件 template.js 并将结果打印到控制台  
        `uglifyjs template.js`~~  
        使用 uglify-js 高度压缩（混淆变量名）js 文件 template.js，并保存到 文件 template.min.js 中  
        `uglifyjs -m toplevel template.js -o template.min.js`

- [typescript](typescript.md#install)

- gitbook
    1. 安装 gitbook  
        `npm install -g gitbook-cli`
    2. 在当前目录下，初始化一个 gitbook 工程  
        `gitbook init`
    3. 构建  
        ~~`gitbook build`~~  
        `gitbook build --gitbook=2.6.7`  
        > 2.6.7 版本支持离线 html 跳转链接
    4. 运行服务  
        `gitbook serve`  
        浏览器中访问 http://127.0.0.1:4000/ 即可浏览

- docsify
    1. 安装 docsify  
        `npm install -g docsify-cli`
    2. 在当前目录下，初始化一个工程  
        `docsify init .`
    3. 运行服务  
        `docsify serve .`  
        浏览器中访问 http://127.0.0.1:3000/ 即可浏览

- [node-rtsp-rtmp-server(RTSP/RTMP/HTTP hybrid server)](https://github.com/iizukanao/node-rtsp-rtmp-server)  
    1. 下载源码  
        `git clone https://github.com/iizukanao/node-rtsp-rtmp-server.git`
    2. 安装依赖（需要nodejs版本 >= 0.12）  
        `cd node-rtsp-rtmp-server`  
        `npm install -d`
    3. 安装 coffeescript  
        `npm install -g coffeescript`
    4. 运行服务（服务配置见config.coffee）  
        - 直接使用coffee运行  
            `coffee server.coffee`  
        - ~~使用coffee编译后，使用nodejs运行  
            `coffee -c .`  
            `node server.js`~~
    5. 访问（192.168.1.200为服务器ip，80端口和1935端口号配置见config.coffee）
        - 点播  
            1. 复制媒体文件video.mp4到file目录下  
            2. 访问  
                rtsp://192.168.1.200:80/file/video.mp4  
                rtmp://192.168.1.200/file/mp4:video.mp4
        - 直播
            1. 使用ffmpeg推流  
                `ffmpeg -re -i input.mp4 -c:v copy -c:a copy -f rtsp rtsp://192.168.1.200:80/live/video`  
                `ffmpeg -re -i video.mp4 -c:v copy -c:a copy -f flv rtmp://192.168.1.200/live/video`
            2. 访问  
                rtsp://192.168.1.200:80/live/video  
                rtmp://192.168.1.200/live/video

- [swagger-editor](https://github.com/swagger-api/swagger-editor)

- [electron](https://github.com/electron/electron)
    - 运行当前目录下的工程  
        `electron .`

- electron-packager
    - 打包成可执行程序  
        `electron-packager . electron-quick-start`

- grunt  
    `npm install -g grunt grunt-electron-installer`
    - 构建安装程序
        1. 在项目根目录下（即与 package.json 同级目录下），新建 Gruntfile.js 文件如下
            ```javascript
            var grunt = require("grunt");
            grunt.config.init({
                pkg: grunt.file.readJSON("package.json"),
                "create-windows-installer": {
                    x64: {
                        appDirectory: "./electron-quick-start-win32-x64",
                        authors: "no one",
                        exe: "electron-quick-start.exe",
                        description:"electron-quick-start",
                    }       
                }
            })

            grunt.loadNpmTasks("grunt-electron-installer");
            grunt.registerTask("default", ["create-windows-installer"]);
            ```
        2. 执行命令  
            `grunt`

- 使用 zan-proxy 代理服务器
    ```bash
    npm install zan-proxy -g
    zan-proxy --no-update
    ```

- 使用 whistle 代理服务器，浏览器可搭配使用 SwitchyOmega
    ```bash
    npm install whistle -g
    w2 start
    ```

- 使用 marp-cli 转换 markdown 为 ppt/pdf
    ```bash
    npm install -g @marp-team/marp-cli
    
    marp-cli readme.md
    marp-cli readme.md -o readme.pptx
    
    marp-cli -p --server .
    ```

- 增强 shell 脚本工具 [zx](https://github.com/google/zx)
    - 安装（依赖 node.js 需 ≥ 14.8 版本）
        ```bash
        npm i -g zx
        ```
    - 示例
        > 将编写的脚本放在 .mjs 后缀的文件中，或者使用 .js 后缀，但是需要 void async function () {...}() 对脚本进行包装
        ```javascript
        #!/usr/bin/env zx
        
        $.verbose = false; // 默认为 true。关闭打印所有命令以及输出

        await $`cat package.json | grep name`

        let branch = await $`git branch --show-current`
        await $`dep deploy --branch=${branch}`

        await Promise.all([
            $`sleep 1; echo 1`,
            $`sleep 2; echo 2`,
            $`sleep 3; echo 3`,
        ])

        let name = "foo bar"
        await $`mkdir /tmp/${name}`
        ```
    - 执行脚本
        ```bash
        # 执行远程脚本
        zx https://medv.io/example-script.mjs
        ```

- 使用 [ssh2](https://github.com/mscdex/ssh2) 连接 sftp 服务器
    1. 安装
        ```bash
        npm install ssh2
        ```
    2. 示例
        ```javascript
        const { readFileSync } = require("fs");
        const { Client } = require("ssh2");
        
        const conn = new Client();
        conn.on("ready", () => {
            console.log("SSH connection ready.");

            conn.sftp((err, sftp) => {
                if (err) throw err;

                // 读取 "./" 目录
                sftp.readdir("./", (err, list) => {
                    if (err) throw err;
                    console.dir(list.map(f => f.filename));
                    conn.end(); // 退出 ssh 连接
                });

                // 读取文件内容
                sftp.readFile("./webdav.txt", (err, buf) => {
                    if (err) throw err;
                    console.log(buf.toString("base64"));
                    conn.end();
                });

                // 删除文件
                sftp.unlink("./webdav.txt", (err) => {
                    if (err) throw err;
                    console.log("Delete file successfully.");
                    conn.end();
                });

                // 下载 "./webdav.txt" 到本地当前目录下
                sftp.fastGet("./webdav.txt", "./webdav.txt", (err) => {
                    if (err) throw err;
                    console.log("Download file successfully.");
                    conn.end();
                });

                // 上传文件
                sftp.fastPut("./package.json", "./package.json", (err) => {
                    if (err) throw err;
                    console.log("Upload file successfully.");
                    conn.end();
                });
            });
        }).on("error", (err) => {
            throw err;
        }).connect({
            host: "127.0.0.1",
            port: 22,
            username: "sftpuser",
            // privateKey: readFileSync("~/.ssh/id_rsa"),
            password: "123456",
            // debug: function(s) { console.log(new Date(), s); } // 打开调试日志
        });
        ```

- 使用 [PEG.js](https://github.com/pegjs/pegjs) 自定义 DSL 解析
    1. 安装
        ```bash
        npm install -g pegjs
        ```
    2. 编写语法文件，可参考[在线语法设计](https://pegjs.org/online)
        ```
        Expression
          = head:Term tail:(_ ("+" / "-") _ Term)* {
              return tail.reduce(function(result, element) {
                if (element[1] === "+") return result + element[3];
                if (element[1] === "-") return result - element[3];
              }, head);
            }

        Term
          = head:Factor tail:(_ ("*" / "/") _ Factor)* {
              return tail.reduce(function(result, element) {
                if (element[1] === "*") return result * element[3];
                if (element[1] === "/") return result / element[3];
              }, head);
            }

        Factor
          = "(" _ expr:Expression _ ")" { return expr; }
          / Integer
          / arr:Array "[" _ idx:Expression _ "]" { return arr[idx]; }

        Array
          = "[" _ "]" { return []; } // 空数组
          / "[" head:(_ Integer _ ",")* tail:(_ Integer _) "]" { return head.concat([tail]) .map((element) => element[1]); }

        Integer "integer"
          = _ [0-9]+ { return parseInt(text(), 10); }

        _ "whitespace"
          = [ \t\n\r]*
        ```
    3. 根据步骤二的语法文件（如 arithmetics.pegjs），生成 js 解析代码
        ```bash
        pegjs arithmetics.pegjs
        # pegjs -o arithmetics-parser.js arithmetics.pegjs
        ```
    4. 使用
        ```javascript
        const parser = require("./arithmetics");
        parser.parse("2 * (3 + 4)");
        parser.parse("[1,2][1] + 2");
        ```

- 使用 [inliner](https://github.com/remy/inliner) 转换 web 页面到单个 html 文件中
    1. 安装
        ```bash
        npm install -g inliner
        ```
    2. 压缩
        ```bash
        inliner https://www.baidu.com > baidu.html
        ```

## *Snippets*

- [获取内存信息](http://nodejs.cn/api/process.html#process_process_memoryusage)
    ```javascript
    process.memoryUsage();
    // { // 单位：字节（byte）
    //     rss: 34009088, // 常驻集大小, 是为进程分配的物理内存（总分配内存的子集）的大小，包括所有的 C++ 和 JavaScript 对象与代码
    //     heapTotal: 5455872, // V8 堆内存大小，包括已使用的和未使用的
    //     heapUsed: 3022072, // V8 已使用堆内存大小
    //     external: 1583051, // V8 管理的绑定到 Javascript 对象的 C++ 对象的内存使用情况
    //     arrayBuffers: 9399 // 分配给 ArrayBuffer 和 SharedArrayBuffer 的内存，包括所有的 Node.js Buffer。这也包含在 external 值中。当 Node.js 被用作嵌入式库时，此值可能为 0，因为在这种情况下可能无法跟踪 ArrayBuffer 的分配。
    // }
    ```

- Socket（TCP&UDP）
    - TCP
        - 服务端
            ```javascript
            var net = require("net");
            var server = net.createServer();
            server.on("connection", function(socket) {
                socket.setEncoding("utf8");
                socket.on("data", function(data) {
                    console.log(data.toString());
                });
                socket.on("end", function() {
                    console.log("end");
                });
            });
            server.listen(8765, "127.0.0.1");
            ```
        - 客户端
            ```javascript
            var net = require("net");
            var client = new net.Socket();
            client.setEncoding("utf8");
            client.connect(8765, "127.0.0.1", function() {
                console.log("connect");
                client.write("to server");
                client.end("end");
            });
            client.on("data", function(data) {
                console.log("receive data from server");
            });
            ```
    - UDP
        - 服务器
            ```javascript
            var dgram = require("dgram");
            var server = dgram.createSocket("udp4");
            server.on("message", function(msg, rinfo) {
                console.log(msg);
                var buf = new Buffer("测试");
                server.send(buf, 0, buf.length, rinfo.port, rinfo.address);
            });
            server.on("listening", function() {
                console.log("listen");
            });
            server.bind(12345, "127.0.0.1");
            ```
        - 客户端
            ```javascript
            server.on("message", function(msg, rinfo) {
                console.log(msg);
                var buf = new Buffer("测试");
                server.send(buf, 0, buf.length, rinfo.port, rinfo.address);
            });
            ```

- 读取文件内容
    ```javascript
    var fs = require("fs");

    fs.readFile("./services.txt", "utf-8", function(err, data) {
        if (err) throw err;
        console.info(data);
    });
    ```

- 创建一个简单的 http 服务
    ```javascript
    var http = require("http");

    var server = http.createServer(function(req, res) {
        res.writeHead(200, { "Content-Type": "text/html" });

        res.end("<!DOCTYPE html><html lang=\"en\"><head><meta charset=\"UTF-8\"><title>greeting</title></head><body>hello, world</body></html>");
    });

    server.listen(8080, function() {
        console.log("Server is running at http://localhost:8080/");
    });
    ```

- 创建一个简单的 https 服务（双向认证）  
    ```bash
    # 生成 CA 自签名证书
    openssl genrsa -out ca.key 4096
    openssl req -new -x509 -days 7300 -key ca.key -subj "/C=CN/ST=JS/L=NJ/O=Sunke, Inc./CN=Sunke Root CA" -out ca.crt

    # 生成服务器证书
    openssl req -newkey rsa:2048 -nodes -keyout server.key -subj "/C=CN/ST=JS/L=NJ/O=Sunke, Inc./CN=localhost" -out server.csr
    openssl x509 -sha256 -req -extfile <(printf "subjectAltName=DNS:localhost,IP:127.0.0.1") -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt
    # cat server.crt ca.crt > server.b.crt

    # 生成客户端证书
    openssl req -newkey rsa:2048 -nodes -keyout client.key -subj "/C=CN/ST=JS/L=NJ/O=/CN=" -out client.csr
    openssl x509 -sha256 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt
    ```
    ```javascript
    const https = require("https");
    const fs = require("fs");

    https.createServer({
        key: fs.readFileSync("./server.key"),
        // passphrase: "123456", // server.key 加密秘钥
        cert: fs.readFileSync("./server.crt"),
        ca: fs.readFileSync("./ca.crt"),
        requestCert:true, // 请求客户端证书
        rejectUnauthorized: true // 如果请求的客户端没有来自信任 CA 颁发的证书，拒绝客户端的连接
    }, (req, res) => {
        res.writeHead(200, { "Content-Type": "text/html" });
        res.write("<!DOCTYPE html><html lang=\"en\"><head><meta charset=\"UTF-8\"><title>greeting</title></head><body>hello, world</body></html>");
        res.end();
    }).listen(8443, function() {
        console.log("Server is running at https://127.0.0.1:8443/");
    });
    ```
    ```bash
    curl --cacert ./ca.crt --cert ./client.crt --key ./client.key https://127.0.0.1:8443
    ```

- sqlite
    ```bash
    npm install sqlite3
    ```
    ```javascript
    const sqlite3 = require("sqlite3");
    const db = new sqlite3.Database("mydb.db");
    
    ```

# node-webkit

- 创建项目(hello World)
    1. 创建页面文件 index.html
        ```html
        <!DOCTYPE html>
        <html>
            <head>
            </head>
            <body>
                <h1>Hello World!</h1>
            </body>
        </html>
        ```
    2. 创建配置文件 package.json
        ```json
        {
            "name": "helloworld",
            "main": "index.html"
        }
        ```

- 运行
    1. 将index.html和package.json这两个文件压缩到一个zip压缩包里，命名为helloworld.zip
    2. 把helloworld.zip这个文件的扩展名改为nw,变为 helloworld.nw
    3. 之前得到的helloworld.nw这个文件就可以用nw.exe来执行了，直接把helloworld.nw拖到nw.exe上就可以了。
    4. 把helloworld.nw这个文件跟node-webkit的环境文件一起打包成一个可执行文件，打开windows的cmd,然后输入如下命令：  
        `copy /b nw.exe+app.nw app.exe`
    5. 但我们大多数人想的是给用户一个exe文件，用户就可以使用了，不用再附带一些其他文件。需要一个软件叫Enigma Virtual Box，首先下载和安装这个软件，然后打开它。然后在Enter Input File Name那里输入我们的app.exe的路径，在Enter Output File Name那里填写我们要把打包出来的可执行文件输出到哪里。最后是把除app.exe外的其它文件拖入到Files那里，遇到提示的话默认就可以了。
    > - The nw.pak and icudt.dll must be shipped along with nw.exe, the former one contains important javascript lib files, and the latter one is a important network library.  
    > - ffmpegsumo.dll are media library, if you want to use \<video\> and \<audio\> tag, or other media related features, you should ship it.  
    > - libEGL.dll and libGLESv2.dll are used for WebGL and GPU acceleration, you had better ship them. And D3DCompiler_43.dll and d3dx9_43.dll as well if you want to make sure WebGL works on more hardware. These 2 files are from DirectX redistributable.

# electron

- 环境搭建
    1. npm install electron
    2. npm install electron-packager --save-dev
    3. npm install electron-packager -g

- 创建项目(hello World)
    1. 创建目录helloworld
    2. 在helloworld目录下创建页面文件 index.html
        ```html
        <h1>Hello World!</h1>
        ```
    3. 在helloworld目录下创建脚本文件 main.js
        ```javascript
        const {app, BrowserWindow} = require("electron");

        let mainWindow;

        app.on("ready", () => {
            mainWindow = new BrowserWindow({
                height: 600,
                width: 800
            });

            mainWindow.loadURL("file://" + __dirname + "/index.html");
        });
        ```
    4. 在helloworld目录下创建配置文件 package.json
        ```json
        {
            "name":    "helloworld",
            "version": "1.0.1",
            "main":    "main.js"
        }
        ```

- 运行
    ```bash
    electron .
    ```

- 发布
    ```bash
    electron-packager <sourcedir> <appname> --platform=<platform> --arch=<arch> [optional flags...]
    ```

- [使用 protocol 注册自定义协议或拦截协议请求](https://m.w3cschool.cn/electronmanual/electronmanual-protocol.html)
    ```javascript
    const electron = require("electron");
    const app = electron.app;
    const path = require("path");

    app.on("ready", function() {
        var protocol = electron.protocol;

        protocol.registerFileProtocol("atom", function(request, callback) {
            var url = request.url.substr(7);
            callback({path: path.normalize(__dirname + "/" + url)});
        }, function(error) {
            if (error)
                console.error("Failed to register protocol")
        });
        
        // protocol.interceptHttpProtocol("http", handler[, completion]); // 拦截 http 请求
    });
    ```

# [express](https://www.expressjs.com.cn/)

1. 安装 express
    ```bash
    npm install express
    ```

2. 创建 app.js
    ```javascript
    const app = require("express")();
    const bodyParser = require("body-parser");

    app.use(bodyParser.json()); // for parsing application/json

    app.post("/greeting", function(req, res, next) {
        console.log(req.body);
        res.json(req.body);
    });
    ```

3. 运行
    ```bash
    node app.js
    ```

# node-fetch

1. 安装
    ```bash
    npm install node-fetch
    ```

2. 调用
    ```javascript
    const fetch = require("node-fetch");

    fetch("https://www.baidu.com");
    ```

3. 代理访问
    1. 安装
        ```bash
        npm install https-proxy-agent
        ```
    2. 调用
        ```javascript
        const HttpsProxyAgent = require("https-proxy-agent");

        fetch("https://www.baidu.com", {
            agent: new HttpsProxyAgent("http://192.168.0.1:5566")
        });
        ```
