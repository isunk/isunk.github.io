---
title: Deno
tags: deno
categories: deno
---

# [Deno](https://github.com/denoland/deno)

## 环境搭建

- 在线安装
    - linux  
        ~~`curl -fsSL https://deno.land/x/install/install.sh | sh`~~  
        `curl -fsSL https://x.deno.js.cn/install.sh | sh`
    - [windows](https://dl.deno.js.cn/release/v1.25.0/deno-x86_64-pc-windows-msvc.zip)（powershell）  
        ~~`iwr https://deno.land/x/install/install.ps1 -useb | iex`~~  
        `iwr https://x.deno.js.cn/install.ps1 -useb | iex`

- [Deno for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=denoland.vscode-deno)
    1. 安装  
        `ext install denoland.vscode-deno`
    2. 配置，在工程根目录下创建 settings.json 并配置开启插件
        ```bash
        mkdir .vscode
        cat > .vscode/settings.json << EOF
        {
            "deno.enable": true
        }
        EOF
        ```

## 命令

- 执行脚本
    ```bash
    # deno run greeting.ts
    deno run https://deno.land/std/examples/welcome.ts
    ```

- 安装 HTTP 文件服务器
    ```bash
    deno install --allow-net --allow-read https://deno.land/std/http/file_server.ts

    file_server . # 启动 HTTP 文件服务器
    ```

## 设置

- 设置代理服务器
    ```bash
    export HTTP_PROXY=http://127.0.0.1:5566 # 设置代理服务器
    export HTTPS_PROXY=http://127.0.0.1:5566 # 设置代理服务器

    deno run --allow-net service.ts # --allow-net 表示允许网络访问
    # deno run -A service.ts # -A 表示允许所有权限，如网络访问等
    ```

## 运行时

- [外部方法接口](https://deno.land/manual@v1.16.3/runtime/ffi_api)

## 标准库

- uuid
    ```typescript
    import * as uuid from "https://deno.land/x/std/uuid/v4.ts";

    // uuid.generate()
    ```

- http & websocket
    ```typescript
    import { listenAndServe } from "https://deno.land/std/http/server.ts";
    import { acceptWebSocket, acceptable, WebSocket, isWebSocketCloseEvent } from "https://deno.land/std/ws/mod.ts";

    listenAndServe({ port: 3000 }, async (request) => {
        if (request.method === "GET" && request.url === "/") {
            request.respond({
                status: 200,
                headers: new Headers({
                    "content-type": "text/html"
                }),
                body: await Deno.open("./index.html")
            });
        }
        if (request.method === "GET" && request.url === "/ws") { // WebSocket
            if (acceptable(request)) {
                acceptWebSocket({
                    conn: request.conn,
                    bufReader: request.r,
                    bufWriter: request.w,
                    headers: request.headers
                }).then(async function(websocket: WebSocket): Promise<void> {
                    for await (const event of websocket) { // wait for new messages
                        const message = typeof event === "string" ? event : "";
                        console.info("new message received", message);
                        if (!message && isWebSocketCloseEvent(event)) { // disconnect
                            break;
                        }
                    }
                });
            }
        }
    });
    console.log("server has started on http://127.0.0.1:3000");
    ```

- crypto
    ```typescript
    import { crypto } from "https://deno.land/std@0.114.0/crypto/mod.ts";

    // This will delegate to the runtime's WebCrypto implementation.
    console.log(
        await crypto.subtle.digest(
            "SHA-384",
            new TextEncoder().encode("hello world"),
        )
    );

    // This will use a bundled WASM/Rust implementation.
    console.log(
        await crypto.subtle.digest("BLAKE3", new TextEncoder().encode("hello world"))
    );
    ```

- workers  
    Workers can be used to run code on multiple threads. Each instance of Worker is run on a separate thread, dedicated only to that worker.
    ```typescript
    new Worker(new URL("./worker.ts", import.meta.url).href, { type: "module" });
    ```

- sleep
    ```typescript
    function sleep(ms: number){
        return new Promise((resolve) => setTimeout(resolve, ms));
    }

    await sleep(500); // sleep for 500ms
    ```

### example

#### chat - WebSocket chat server and browser client

```bash
deno run --allow-net --allow-read https://deno.land/std/examples/chat/server.ts
```

#### curl - print the contents of a url to standard output

```bash
deno run --allow-net=deno.land https://deno.land/std/examples/curl.ts https://deno.land/
```

## 第三方库

- [opine](https://github.com/asos-craigmorten/opine) 实现 web 服务器
    - hello, world
        ```typescript
        import { opine } from "https://deno.land/x/opine/mod.ts";

        const app = opine();

        app.get("/greeting", function (req, res) {
            res.send("<html><body>hello world</body></html>");
        });

        app.listen(3000, () => console.log("server has started on http://127.0.0.1:3000 🚀"));
        ```
    - json
        ```typescript
        import { opine, json } from "https://deno.land/x/opine/mod.ts";

        const app = opine();

        app.use(json()); // for parsing application/json

        app.get("/greeting/:name", function (req, res) {
            res.json({
                "greeting": "hello, " + req.params.name
            });
        });
        app.post("/greeting", function (req, res) {
            res.json({
                "greeting": "hello, " + req.body.name
            });
        });

        app.listen(3000);
        ```
    - 下载文件
        ```typescript
        import { opine } from "https://deno.land/x/opine/mod.ts";
        import { dirname, join } from "https://deno.land/x/opine@1.5.4/deps.ts";

        const app = opine();
        const __dirname = dirname(import.meta.url);

        app.get("/files/:file(*)", async function (req, res, next) { // 下载文件，如 http://127.0.0.1:3000/files/a.txt（./files/a.txt）
            const filePath = join(__dirname, "files", req.params.file);

            try {
                await res.download(filePath);
            } catch (err) {
                // file for download not found
                if (err instanceof Deno.errors.NotFound) {
                    res.status = 404;
                    res.send("Cant find that file, sorry!");
                    return;
                }
                // non-404 error
                return next(err);
            }
        });

        app.listen(3000);
        ```

- [mysql](https://github.com/denodrivers/mysql) 连接数据库
    ```typescript
    import { Client } from "https://deno.land/x/mysql/mod.ts";

    const client = await new Client().connect({
        hostname: "127.0.0.1",
        username: "root",
        db: "dbname",
        password: "password",
        // poolSize: 3, // 最大连接数
    });

    // 创建数据库
    await client.execute(`CREATE DATABASE IF NOT EXISTS enok`);

    // 创建表
    await client.execute(`
        CREATE TABLE users (
            id int(11) NOT NULL AUTO_INCREMENT,
            name varchar(100) NOT NULL,
            created_at timestamp not null default current_timestamp,
            PRIMARY KEY (id)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    `);

    // 插入数据
    await client.execute(`INSERT INTO users(name) values(?)`, ["manyuanrong"]); // { affectedRows: 1, lastInsertId: 1 }

    // 修改数据
    await client.execute(`update users set ?? = ?`, ["name", "MYR"]); // { affectedRows: 1, lastInsertId: 0 }

    // 删除数据
    await client.execute(`delete from users where ?? = ?`, ["id", 1]); // { affectedRows: 1, lastInsertId: 0 }

    // 查询
    const { rows: users } = await client.execute(`select * from users`);
    console.log(users);
    await client.query("select ??, name from ?? where id = ?", ["id", "users", 1]);

    // execute
    await client.execute(`select * from users`);

    // iterator
    await client.useConnection(async (conn) => {
        // note the third parameter of execute() method.
        const { iterator: users } = await conn.execute(
            `select * from users`,
            [], // params
            false, // iterator
        );
        for await (const user of users) {
            console.log(user);
        }
    });

    // transaction
    await client.transaction(async (conn) => {
        await conn.execute(`insert into users(name) values(?)`, ["test"]);
        return await conn.query(`select ?? from ??`, ["name", "users"]);
    });

    await client.close();
    ```

- [sqlite](https://github.com/dyedgreen/deno-sqlite)
    ```typescript
    import { DB } from "https://deno.land/x/sqlite/mod.ts";

    // Open a database
    const db = new DB("test.db");

    db.query("CREATE TABLE IF NOT EXISTS people (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT)");

    // Run a simple query
    for (const name of ["Peter Parker", "Clark Kent", "Bruce Wayne"]) {
        db.query("INSERT INTO people (name) VALUES (?)", [name]);
    }

    // Print out data in table
    for (const [name] of db.query("SELECT name FROM people")) {
        console.log(name);
    }

    // Close connection
    db.close();
    ```

- [redis](https://github.com/keroxp/deno-redis)
    ```typescript
    import { connect } from "https://denopkg.com/keroxp/deno-redis/mod.ts";

    const redis = await connect({
        hostname: "127.0.0.1",
        port: 6379
    });
    const ok = await redis.set("example", "this is an example");
    const example = await redis.get("example");
    ```
