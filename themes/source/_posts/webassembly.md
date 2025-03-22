---
title: WebAssembly
tags: webassembly
categories: webassembly
---

# [WebAssembly](https://www.wasm.com.cn/)

## 安装

### Linux

1. 安装依赖
    ```bash
    apt-get update
    apt-get -y install git build-essential python pip
    ```

1. 获取 emsdk
    ```bash
    # Get the emsdk repo
    git clone https://github.com/emscripten-core/emsdk.git
    ```

2. 安装配置 Emscripten
    ```bash
    # Enter that directory
    cd emsdk

    # Fetch the latest registry of available tools.
    # ./emsdk update

    # Download and install the latest SDK tools. Need install Python first. 
    ./emsdk install latest

    # Make the "latest" SDK "active" for the current user. (writes ~/.emscripten file)
    ./emsdk activate latest

    # Activate PATH and other environment variables in the current terminal
    source ./emsdk_env.sh

    # Verifying Emscripten
    emcc -v
    ```

3. 创建 hello.c
    ```c
    #include <stdio.h>

    int main() {
        printf("hello, world!\n");
        return 0;
    }
    ```

4. 编译
    ```bash
    # 生成 a.out.js, a.out.wasm
    emcc hello.c

    # 生成 hello.js
    emcc hello.c -o hello.js

    # 生成 hello.html 和 hello.js
    emcc hello.c -o hello.html
    ```

5. 运行
    ```bash
    node a.out.js
    ```