---
title: Javascript
tags: javascript
categories: javascript
---

# [Javascript](http://www.w3school.com.cn/js/index.asp)

## DOM

- 开启内容编辑
    ```javascript
    javascript:document.body.contentEditable="true"; // "false" if you want to diable
    document.designMode="on";
    ```

- 提示消息框  
    `var name = prompt("please input your name", "luffy");`  
    确认消息框  
    `var isOK = window.confirm("Are you OK?");`

- 获取页面高度
    ```javascript
    // console.info(document.body.scrollHeight, document.documentElement.scrollHeight, document.body.offsetHeight, document.documentElement.offsetHeight, document.body.clientHeight, document.documentElement.clientHeight);
    var height = Math.max(
        Math.max(document.body.scrollHeight, document.documentElement.scrollHeight),
        Math.max(document.body.offsetHeight, document.documentElement.offsetHeight),
        Math.max(document.body.clientHeight, document.documentElement.clientHeight)
    );
    ```

- 打开新窗口
    ```javascript
    // let c = window.open("http://www.baidu.com", "_blank");
    let c = window.open("", "", "width=360,height=640"); // 子窗口 window 对象
    c.document.write("hello, world");
    c.focus();
    ```
    ```javascript
    let p = window.opener; // 父窗口 window 对象
    ```

- 下载文件
    ```javascript
    fetch("/img/flexible/logo/pc/result.png").then(res => res.blob().then(blob => {
        var url = window.URL.createObjectURL(blob);
        var filename = "baidu.png";
        var a = document.createElement("a");
        a.href = url;
        a.download = filename;
        a.click();
        window.URL.revokeObjectURL(url); // 释放 url 对象内存
    })); 
    ```

- 拖拽文件
    ```javascript
    let div = document.createElement("div");
    div.style.width = "400px";
    div.style.height = "300px";
    div.style.border = "dashed";
    
    div.ondragenter = div.ondragover = div.ondragleave = function(e) {
        e.preventDefault();
    };
    div.ondrop = function(e) {
        e.preventDefault();

        let file = e.dataTransfer.files[0];

        let reader = new FileReader();
        reader.readAsBinaryString(file);
        reader.onload = function() {
            let arr = new Uint8Array(this.result.length);
            for (let i = 0; i < this.result.length; i++) {
                arr[i] = this.result.charCodeAt(i);
            }
            let url = window.URL.createObjectURL(new Blob([arr]));
            let a = document.createElement("a");
            a.href = url;
            a.download = file.name;
            a.click();
            window.URL.revokeObjectURL(url);
        }
    };
    
    document.body.appendChild(div);
    ```

- Blob 切分、合并
    ```javascript
    let blob = e.dataTransfer.files[0]; // some file from user...

    let cursor = 0,
        arr = [];
    while(cursor < blob.size) {
        let b = blob.slice(cursor, cirsor += 1 * 1024 * 1024); // 切分，分片不超过 1 MB
        arr.push(b);
    }

    let merged = new Blob(arr); // 合并
    ```

- FileReader
    ```javascript
    let file = e.dataTransfer.files[0];

    let reader = new FileReader();
    
    var url = null, blob = null;
    
    // 读取为 URL 对象
    reader.readAsDataURL(file);
    reader.onload = function(e) {
        // url = e.target.result;
        url = this.result;
        
        // 下载
        let a = document.createElement("a");
        a.href = url;
        a.download = file.name;
        a.click();
    }
    
    // 读取为 ArrayBuffer 对象
    reader.readAsArrayBuffer(file);
    reader.onload = function() {
        blob = new Blob([this.result]); // ArrayBuffer 转 Blob
    
        url = window.URL.createObjectURL(blob); // Blob 转 URL
        window.URL.revokeObjectURL(url); // 需要释放 URL
    }
    
    // 读取为 BinaryString 字符串
    reader.readAsBinaryString(file);
    reader.onload = function() {
        let arr = new Uint8Array(this.result.length);
        for (let i = 0; i < this.result.length; i++) {
            arr[i] = this.result.charCodeAt(i);
        }
        blob = new Blob([arr]); // BinaryString 转 Blob
    }
    ```

- HTML 编码（转义）、解码
    ```javascript
    function htmlEncode(str) {
        var div = document.createElement("div");
        div.appendChild(document.createTextNode(str));
        return div.innerHTML;
    }
    function htmlDecode(str) {
        var div = document.createElement("div");
        div.innerHTML = str;
        return div.innerHTML;
    }
    ```

- Base64 编码、解码
    ```javascript
    // 编码
    window.btoa("china is so nb") // "Y2hpbmEgaXMgc28gbmI="

    // 解码
    window.atob("Y2hpbmEgaXMgc28gbmI=") // "china is so nb"
    ```

- 使用 canvas 处理图片
    ```javascript
    // document.body.innerHTML = "";

    let input = document.createElement("input");
    input.type = "file";
    input.name = "picture";
    document.body.appendChild(input);

    let canvas = document.createElement("canvas");
    document.body.appendChild(canvas);

    let image = new Image();
    image.onload = function () {
        canvas.width = image.width;
        canvas.height = image.height;

        let context = canvas.getContext("2d");
        context.drawImage(image, 0, 0);

        let imageData = context.getImageData(0, 0, image.width, image.height); // 获取图像信息

        for (y = 0; y < imageData.height; y++) { // 图像的高/像素
            for (x = 0; x < imageData.width; x++) { // 图像的宽/像素
                let idx = (imageData.width * y + x) * 4; // 像素点在 ImageData.data: Uint8ClampedArray 中的位置（每连续四个值为一个像素的 RGBA 值）

                let r = imageData.data[idx + 0], // red
                    g = imageData.data[idx + 1], // green
                    b = imageData.data[idx + 2], // blue
                    a = imageData.data[idx + 3]; // alpha

                let gray = 0.299 * r + 0.578 * g + 0.114 * b; // 计算灰度值
                imageData.data[idx + 0] = imageData.data[idx + 1] = imageData.data[idx + 2] = gray;
            }
        }

        context.putImageData(imageData, 0, 0); // 设置图像信息
        
        // canvas.toDataURL("image/webp", 0.8) // 转换成 webp 格式（默认为 image/png，其它可选有 image/jpeg），质量为 0.8（默认为 0.92，取值范围为 0-1）
    };

    input.onchange = function(e) {
        var reader = new FileReader();
        // reader.readAsDataURL(e.target.files[0]);
        reader.readAsDataURL(this.files[0]);
        reader.onload = function(e) {
            // image.src = e.target.result;
            image.src = this.result;
        }
    };
    ```

- 屏幕、应用窗口、页面捕捉
    ```javascript
    let video = document.createElement("video");
    video.controls = true;
    video.autoplay = true;
    video.playsinline = true;
    video.muted = false;
    video.volume = 0;
    document.body.appendChild(video);

    (navigator.mediaDevices || navigator).getDispayMedia({
        video: true
    }).then(stream => {
        video.srcObject = stream;
    }).catch(err => {
        console.error(err.message);
    });
    ```

- 双向绑定（仿 vue 实现）

- 有限状态机

- 语音朗读
    ```javascript
    let voice = window.speechSynthesis.getVoices().filter(v => v.lang === "zh-CN" && v.localService === true && v.default === true).pop();

    let utterance = new SpeechSynthesisUtterance("hello, world");
    utterance.voice = voice;
    // utterance.pitch
    // utterance.rate
    window.speechSynthesis.speak(utterance);
    ```

- 获取地理位置信息
    ```javascript
    navigator.geolocation.getCurrentPosition(function (position) {
        console.debug("The current position is", position);
    }, function (error) {
        console.error("Can not get current position", error.message);
    }, {
        enableHighAccuracy: false, // 位置是否精确获取
        timeout: 8000, // 获取位置允许的最长时间，默认为 infinity
        maximumAge: 1000 // 多久更新获取一次位置，位置可以缓存的最大时间，默认为 0
    });
    ```

- 异步动态加载 js
    ```javascript
    const require = function (src) {
        return new Promise((resolve, reject) => {
            if (window.modules == null) {
                window.modules = [];
            }
            if (window.modules.indexOf(src) === -1) {
                let script = document.createElement("script");
                script.src = src;
                script.onload = function () {
                    resolve(src);
                    window.modules.push(src);
                };
                document.body.appendChild(script);
            }
        });
    };
    require("https://cdn.jsdelivr.net/mark.js/latest/").then(s => { console.debug("loaded success", s) });
    ```

- 修改浏览器 UA（即 User Agent）
    ```javascript
    Object.defineProperty(navigator, "platform", {
        get: function () {
            return "Android";
        }
    });

    navigator.platform // Android
    ```

- 网页的宽度自动适应手机屏幕的宽度
    ```html
    <head>
        <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no">
    </head>
    ```
    > 1. width=device-width ：表示宽度是设备屏幕的宽度
    > 2. initial-scale=1.0：表示初始的缩放比例，1.0 就是占网页的 100%
    > 3. minimum-scale=1.0：表示最小的缩放比例
    > 4. maximum-scale=1.0：表示最大的缩放比例
    > 5. user-scalable=no：表示用户是否可以调整缩放比例

## 语法

- 闭包
    ```javascript
    (function() {
        // 闭包函数内容
    })(); // 在定义后调用外层函数
    ```

- 计算的属性名称
    ```javascript
    var foo = {
        [`bar` + 1]: `baz`
    }; // {"bar1":"baz"}
    ```

- 按照指定的周期(以毫秒计)来调用函数或计算表达式
    ```javascript
    // 每隔1秒打印一次当前时间
    var obj = setInterval(function(){
        console.info(new Date());
    }, 1000);
    // 取消由 setInterval() 设置的 timeout
    clearInterval(obj);
    ```

- 在指定的毫秒数后调用函数或计算表达式
    ```javascript
    // 5 秒后打印 "hello, world"
    var obj = setTimeout(function(){
        console.info("hello, world");
    }, 5000);
    // 取消由 setTimeout() 方法设置的 timeout
    clearTimeout(obj);
    ```

- 中文字符串转换为 unicode 码
    ```javascript
    var u = escape("中文").toLowerCase().replace(/%u/g, "\\u");
    console.info(u);
    ```

- URL 编码、解码
    ```javascript
    var e = encodeURI("xxx"); // 转义空格
    car e = encodeURIComponent("xxx"); // 转义所有非字母数字字符
    var d = decodeURI("xxx");
    var d = decodeURIComponent("xxx");
    ```

- Number 和 BigInt
    ```javascript
    // 显示 Number 类型最大值，不使用科学记数法
    console.info(Number.MAX_VALUE.toLocaleString());
    ```
    ```javascript
    0.1 + 0.2 // 0.30000000000000004
    
    0.1 + 0.2 === 0.3000000000000000[2-7] // true
    0.1 + 0.2 === 0.3000000000000000[0189] // false
    ```
    > js 中，Number 类型是用双精度 64 位浮点格式表示，只能安全表示 Number.MIN_SAFE_INTEGER (-2 ^ 53 - 1) ~ Number.MAX_SAFE_INTEGER (2 ^ 53 - 1) 之间的整数，解决方案是使用 BigInt 类型表示，如 9n 或 BigInt("9")

- 字符串与整数互转以及移位操作 byte 截取 bit
    > JavaScript 的所有位操作都是先将操作对象转化为 32 位有符号数进行的。详见 Ecma-262 规范（JavaScript 的正式名称是 "ECMAScript"，其语法由 Ecma-262 规范描述）。

    ```javascript
    // 显示整数 "257" 的十进制形式
    console.info((257).toString(2)); // "100000001"
    // 取二进制形式的低八位（即 "00000001 00000001" 中后八位 "00000001"）
    console.info((257 & 0xff).toString(2)); // "1"

    // 将二进制字符串 "10101010" 转换为无符号整数i
    var i = parseInt("10101010", 2); // "170"
    // i = parseInt("-10101010", 2); // "-170"

    // 将整数i右移5位并转换为二进制字符串，即截取二进制字符串 "10101010" 前3位（"101"）
    console.info((i >> (8 - 3)).toString(2));

    // 显示 "-5" 的二进制表示形式（"11111011"）
    console.info((-5 & 0xff).toString(2));
    // 对 "5" 进行按位非操作，并显示其二进制表示形式（"11111011"）
    console.info((~5 & 0xff).toString(2));

    console.assert(~255 + 1 == 255 * -1);
    
    // 通过使用无符号右移运算符，位动位数为0，可以将32位有符号整数，转化为32位无符号整数，即"unsigned = signed >>> 0;"
    console.info(-5 >>> 0); // "4294967291"
    // 通过使用左移运算符，位动位数为0，可以将32位无符号整数，转化为32位有符号整数，即"signed = unsigned << 0;"
    console.info(4294967291 << 0); // "-5"

    // 将8位有符号整数，转化为8位无符号整数
    console.info(-5 >>> 0 & 0xff); // "251"，二进制为"11111011"
    // <=> console.info(parseInt((-5 & 0xff).toString(2), 2));
    // 将8位无符号整数，转化为8位有符号整数
    (function(uint8_value) {
        return (uint8_value & 0xff) >> 7 == 1 ? ((~uint8_value & 0xff) + 1) * -1 : uint8_value;
        // 第一位为符号位，剩余七位为数值位。1表示负数，其值等于所有位取反加1；0表示正数，所有位直接转化为十进制即为值大小。
    })(251); // "-5"，将所有位（251，二进制为"11111011"）按位取反（"00000100"）并加1（"00000101"，即十进制5），由于"11111011"第一位为1，表示负数，因此结果为"-5"
    ```

- 字符与 ascii 码互转
    ```javascript
    "a".charCodeAt() // 97
    
    String.fromCharCode(97) // "a"
    String.fromCharCode(97, 98) // "ab"
    // String.fromCharCode(...[97, 98]) // "ab"
    String.fromCharCode.apply(null, [97, 98]) // "ab"
    ```

- 模板字面量与标签函数
    ```javascript
    let name = "world";

    // 模板字面量   
    let str = `hello, ${name}`;
    console.log(str); // hello, world

    // 标签函数
    let greeting = (name) => {
        return `hello, ${name}`;
    };
    greeting`world`; // hello, world
    ```

- 可选链
    ```javascript
    const obj = {
        prop: {
            a: "value"
        }
    }

    // Optional Chaining
    obj.foo?.a // undefined
    ```

- 丢弃小数部分，保留整数部分  
    `var r = parseInt(5/2);`   
    向上取整，有小数就整数部分加 1  
    `var r = Math.ceil(5/2);`  
    四舍五入  
    `var r = Math.round(5/2);`  
    向下取整  
    `var r = Math.floor(5/2);`  
    | 方法 | 描述 |
    | :--- | :--- |
    | abs(x) | 返回数的绝对值 |
    | acos(x) | 返回数的反余弦值 |
    | asin(x) | 返回数的反正弦值 |
    | atan(x) | 以介于 -PI/2 与 PI/2 弧度之间的数值来返回 x 的反正切值 |
    | atan2(y, x) | 返回从 x 轴到点 (x,y) 的角度（介于 -PI/2 与 PI/2 弧度之间） |
    | ceil(x) | 对数进行上舍入 |
    | cos(x) | 返回数的余弦 |
    | exp(x) | 返回 e 的指数 |
    | floor(x) | 对数进行下舍入 |
    | log(x) | 返回数的自然对数（底为 e） |
    | max(x, y) | 返回 x 和 y 中的最高值 |
    | min(x, y) | 返回 x 和 y 中的最低值 |
    | pow(x, y) | 返回 x 的 y 次幂 |
    | random() | 返回 0 ~ 1 之间的随机数 |
    | round(x) | 把数四舍五入为最接近的整数 |
    | sin(x) | 返回数的正弦 |
    | sqrt(x) | 返回数的平方根 |
    | tan(x) | 返回角的正切 |
    | toSource() | 返回该对象的源代码 |
    | valueOf() | 返回 Math 对象的原始值 |

- 断言  
    `console.assert(1 == 2, "not equal");`

- js 中 "==" 与 "===" 区别
    - "==" 是比较两边**值**是否相同
    - "===" 是比较两边**值**和**类型**是否相同

- 对象展开运算符 (ES6)
    ```javascript
    let a = [1, 2, 3];
    let b = [0, ...a, 4]; // [0, 1, 2, 3, 4]
 
    let obj = { a: 1, b: 2 };
    let obj2 = { ...obj, c: 3 }; // { a: 1, b: 2, c: 3 }
    let obj3 = { ...obj, a: 3 }; // { a: 3, b: 2 }
    ```
    ```javascript
    // 剩余操作符
    let a = [1, 2, 3];
    let [b, ...c] = a;
    b; // 1
    c; // [2,3]
 
    let a = [1, 2, 3];
    let [b, ...[c, d, e]] = a;
    b; // 1
    c; // 2
    d; // 3
    e; // undefined

    function test(a, ...rest){
        console.log(a); // 1
        console.log(rest); // [2, 3]
    }
    test(1, 2, 3)
    ```
    ```javascript
    let object = {
        a: "01", b: "02"
    };
    let newObject = {
        c: "03",
        ...object
    };
    console.log(newObject); // { c: "03", a: "01", b: "02" }
    ```

- 从对象中删除属性
    ```javascript
    const user = {
        id: 1,
        name: "zhangsan",
        gender: "male"
    };

    // 使用 delete 运算符
    delete user.name;
    delete user["name"];

    // 使用 rest 语法进行对象解构
    const { name, ...restObject } = user;
    console.log(restObject); // { id: 1, gender: "male" }
    ```

- 正则表达式
    ```javascript
    var content = "hello, world";

    var pattern = new RegExp("o", "g");
    // var pattern = /o/g; // 修饰符 i 表示忽略大小写，g 表示全局匹配，m 表示多行匹配
    while ((result = pattern.exec(content)) != null) {
        var above = content.substring(0, result.index);
        var lineNum = (above.match(/[\n]/g) || []).length + 1;
        console.info("'%s' found in row: %s, col: %s", result[0], lineNum, result.index + 1);
    }

    console.info(content.match(/o/g));
    ```
    ```javascript
    // 命名分组
    "data:image/png;base64,iVBORw0KGgoAAAANSUh...".match(/^data:image\/(?<type>[a-z]+);base64,(?<data>.+)$/).groups // { "type": "png", "data": "iVBORw0KGgoAAAANSUh..." }
    ```

- [集合](https://blog.csdn.net/u010003835/article/details/79042135)
    ```javascript
    var a = [1, 2, 3], b = [2, 4, 5];

    // ES7
    {
        // 并集
        let union = a.concat(b.filter(v => !a.includes(v))); // [1, 2, 3, 4, 5]
        // 交集
        let intersection = a.filter(v => b.includes(v)); // [2]
        // 差集
        let difference = a.concat(b).filter(v => a.includes(v) && !b.includes(v)); // [1, 3]
    }

    // ES6
    {
        // 并集
        let union = Array.from(new Set(a.concat(b))); // [1, 2, 3, 4, 5]
        // 交集
        var bSet = new Set(b);
        let intersection = Array.from(new Set(a.filter(v => bSet.has(v)))); // [2]
        // 差集
        var aSet = new Set(a);
        let difference = Array.from(new Set(a.concat(b).filter(v => aSet.has(v) && !bSet.has(v)))); //  [1, 3]
    }

    // ES5
    {
        // 并集
        var union = a.concat(b.filter(function (v) { return a.indexOf(v) === -1 })); // [1, 2, 3, 4, 5]
        // 交集
        var intersection = a.filter(function (v) { return b.indexOf(v) > -1 }); // [2]
        // 差集
        var difference = a.filter(function (v) { return b.indexOf(v) === -1 }); // [1, 3]
    }
    ```
    ```javascript
    var m = [[1, 3.0], [2, 3.9], [3, 2.0], [4, 1.2], [5, 1.3], [6, 2.5], [7, 2.0], [8, 3.1], [9, 2.9], [10, 0.9]];

    // 数组降维
    var arr = [].concat.apply([], m); // [1, 3, 2, 3.9, 3, 2, 4, 1.2, 5, 1.3, 6, 2.5, 7, 2, 8, 3.1, 9, 2.9, 10, 0.9]

    // 数组去重
    // var distinction = Array.from(new Set(arr));
    // var distinction = [...new Set(arr)];
    var distinction = arr.filter((item, index, arr) => { return arr.indexOf(item) === index; }); // [1, 3, 2, 3.9, 4, 1.2, 5, 1.3, 6, 2.5, 7, 8, 3.1, 9, 2.9, 10, 0.9]

    // 数组排序
    distinction.sort(); // [0.9, 1, 1.2, 1.3, 10, 2, 2.5, 2.9, 3, 3.1, 3.9, 4, 5, 6, 7, 8, 9]
    
    // 数组求和
    arr.reduce((total, current) => {
        return total + current;
    }, 0);
    ```
    ```javascript
    // 多个数组求交集
    let arr = [
        [1, 2, 3, 4],
        [3, 4, 6],
        [4, 5],
        [4, 5, 8, 9],
        [4, 5, 2, 7],
        [4, 5, 3],
        [4, 5, 0],
    ];
    let res = arr.reduce((a, b) => { 
        return a.filter(c => b.includes(c));
    });
    ```
    ```javascript
    var users = [{ name: "zhangsan", age: 14 }, { name: "lisi", age: 15 }];

    // 映射数组为新数组
    // 语法: array.map(function(currentValue, index, arr), thisValue)
    var names = users.map(u => u.name); // ["zhangsan", "lisi"]
    names.map(n => { return { name: n, sex: "male" } }); // [{ name: "zhangsan", sex: "male" }, { name: "lisi", sex: "male" }]

    // 过滤数组为新数组
    users.filter(u => u.age > 14); // [{ name: "lisi", age: 15 }]
    
    // 数组转对象
    users.reduce((p, c) => {
        p[c.name] = c.age;
        return p;
    }, {}); // { zhangsan: 14, lisi: 15 }
    ```
    ```javascript
    // 优雅地递归取数据
    var data = [
        {
            name: "123",
            type: "menu",
            children: [{ name: "321", type: "operation", children: null }],
        },
        {
            name: "456",
            type: "menu",
            children: [{ name: "654", type: "menu", children: [{ name: "546", type: "operation", children: null }] }],
        }
    ];
    const filterByType = (target, init = []) => {
        target.forEach(item => {
            item.type === "operation" && init.push(item.name);
            item.children && filterByType(item.children, init);
        });
        return init;
    };
    filterByType(data); // ["321", "546"]
    ```
    ```javascript
    // ip 地址转换为二进制
    "192.168.0.101".split(".").reduce((total, cur) => total + ("00000000" + parseInt(cur).toString(2)).substr(-8), "")
    ```
    ```javascript
    // 栈
    let stack = [1, 2];
    stack.push(3); // 入栈
    console.debug(stack); // [1, 2, 3]
    // 出栈最后一个元素
    stack.pop(); // 3
    console.debug(stack); // [1, 2]
    
    // 队列
    let queue = [1, 2];
    queue.push(3); // 把数据放入队尾
    console.debug(queue); // [1, 2, 3]
    // 从队列中取出队列中的第一个数
    queue.shift(); // 1
    console.debug(queue); // [2, 3]
    ```
    ```javascript
    Array.from({ length: 3 }, (v, k) => k + 1); // [1, 2, 3]

    // 创建一个 5x4 的二维数组
    new Array(5).fill("").map(d => new Array(4).fill(0));
    ```
    ```javascript
    // 比较两个数组元素的出现顺序，时间复杂度均为 O(n)   

    // using foreach
    function compare(a, b) {
        var i = 0;
        b.forEach(function(el) {
            if(el == a[i]) i++;
        })
        return i == a.length;
    }

    // using reduce
    function compare2(a, b) {
        return b.reduce(function(i, el) {
            return el == a[i] ? i + 1 : i;
        }, 0) == a.length;
    }

    var a = [1,2,3];
    var b = [0,1,4,3,9,10,2,5,6]; // 1,2,3 in wrong order
    var c = [0,4,1,5,6,2,8,3,5];  // 1,2,3 in right order

    console.log(compare(a,b)  ==  false);
    console.log(compare(a,c)  ==  true);
    console.log(compare2(a,b) ==  false);
    console.log(compare2(a,c) ==  true);
    ```

- Json
    ```javascript
    // json 序列化  
    var string = JSON.stringify(object);
    var string2 = JSON.stringify(object, (key, value) => {
        if (key === "phone" && value) {
            return value.replace(/^(\d{3})\d*(\d{2})$/, "$1***$2"); // 脱敏
        }
        return value;
    }, 4); // 文本添加缩进：空格、换行符。为 4 表示 4 个空格（最大为 10）；为 "\t" 表示 1 个制表符。
    // json 反序列化
    var object = JSON.parse(string);

    // 格式化显示 json
    document.body.innerHTML = `<pre>${JSON.stringify(obj, null, "\t")}</pre>`;
    ```

- Console
    ```javascript
    console.log("this is log message");
    console.info("this is info message");
    console.warn("this is warn message");
    console.error("this is error message");
    
    console.log(
        "Nothing here %cHi Cat %cHey Bear",  // Console Message
        "color: blue", "color: red" // CSS Style
    );
    console.log("\x1b[33m%s\x1b[0m", "yellow"); // nodejs 控制台设置颜色

    // 1.将css样式传递给数组
    const styles = [ 
        "color:green",
        "background:yellow",
        "font-size:30px",
        "border:1px solid red",
        "text-shadow:2px 2px black",
        "padding:10px"
    ].join(";"); // 2.连接单个数组项并将它们连接成一个用分号分隔的字符串（;）
    // 3.传递样式变量
    console.log("%cHello There", styles);
    // or
    console.log("%c%s", styles, "Some Important Message Here");
    
    console.log("%d 年 %d 月 %d 日", 2011, 3, 26); // 占位符
    
    console.group("第一组信息");
    console.log("第一组第一条");
    console.log("第一组第二条");
    console.groupEnd();
    console.group("第二组信息");
    console.log("第二组第一条");
    console.groupEnd();
    
    console.dir({ // 显示一个对象所有的属性和方法
        name: "zhangsan"
    });
    
    console.dirxml(document.getElementById("info")); // 显示网页的某个节点（node）所包含的html/xml代码
    
    console.assert(year === 2020, "not equal"); // 判断一个表达式或变量是否为真。如果结果为否，则在控制台输出一条相应信息，并且抛出一个异常
    
    console.trace(); // 追踪函数的调用轨迹/调用栈
    
    console.time("控制台计时器一");
    for (var i = 0; i < 1000; i++) {
        for (var j = 0; j < 1000; j++) {}
    }
    console.timeEnd("控制台计时器一"); // 显示代码的运行时间
    
    console.profile("性能分析器");
    // ...
    console.profileEnd(); // 分析程序各个部分的运行时间

    // 通过表格的形式打印数组和对象
    console.table([{ name: "zhangsan" }]);
    ```

- async 和 await
    ```javascript
    // async 用于申明一个 function 是异步的，而 await 用于等待一个异步方法执行完成。await 只能出现在 async 函数中
    function sleep(ms) {
        return new Promise((resolve, reject) => {
            setTimeout(resolve, ms);
        });
    }

    async function print(value, ms) {
        await sleep(ms);
        console.log(value);
    }

    print("hello world", 50); // 指定 50 毫秒以后，输出 hello world
    ```
    ```javascript
    (async () => {
        for (let i = 0; i < 100; i++) {
            const promise = fetch("https://127.0.0.1/greeting");
            const body = await (await promise).json();
            console.info(`The ${i}st request with response is`, body);
        }
    })();
    ```
    ```javascript
    // 仿 fetch 方法实现
    function fetch(url) {
        return new Promise(function (resolve, reject) {
            var xhr = new XMLHttpRequest();
            xhr.open("GET", url, true);
            xhr.onreadystatechange = function () {
                if(xhr.readyState == 4) {
                    if(xhr.status == 200) {
                        resolve(xhr.responseText);
                    } else {
                        reject("Server response failed.");
                    }
                }
            };
            xhr.send(null);
        });
    }
    fetch("http://www.baidu.com").then(function (res) {
        console.info(res);
    }).catch(function (err) {
        console.log(err);
    });
    ```
    > - async 与 await 配套使用，await 必须在 async 方法内部，await 返回结果为 Promise 对象中执行结果，即 resolved 或 rejected 的值
    > - await 后面跟 Promise 对象或值，如果是值，则会转到一个立即 resolve 的 Promise 对象；async 返回一个 Promise 对象，如果是值，则自动包装为 Promise
    > - async 方法中如果有多个 await，如果有一个 await 状态是 reject，则后面操作不会继续执行
    > - await 用来串行的执行异步操作，并行的可以考虑 Promise.all([p1, p2, ...])

- 其他
    ```javascript
    var as = [{ name: "zhangsan", age: 14 }, { name: "lisi", age: 15 }];

    for (var aa in as[0]) { // for ... in 表示遍历属性
        console.assert(aa == "name" || aa == "age");
    }

    for (var a of as) { // for ... of 表示遍历数组，for 循环的性能优于 forEach、map、filter 等
        console.assert(a.name == "zhangsan" || a.name == "lisi");
    }
    ```
    ```javascript
    for (let t = Date.now(); Date.now() - t <= 5000;); // 当前线程等待 5 秒
    ```
    ```javascript
    console.time(); // 计时开始
    // console.time("timer01");
    // ...
    console.timeEnd(); // 计时结束，返回时差，单位毫秒，如 default: 944.423095703125ms
    // console.timeEnd("timer01");
    ```
    ```javascript
    !Object.getOwnPropertyNames(obj).length && (obj = null); // 如果 obj 是空对象，则赋值 obj 为 null
    ```

- getter 和 setter
    ```javascript
    var obj = {
        _val: 0,
        get val(): {
            return this._val;
        },
        set val(_val): {
            this._val = _val;
        }
    };
    ```
    ```javascript
    Object.defineProperty(obj, "val", {
        get: function() {
            return this._val;
        }
    });
    ```

- 动态执行
    ```javascript
    // https://m.jb51.net/article/180809.htm
    // 格式：let func = new Function(arg1, arg2, ..., body);
    let sum = new Function("a", "b", "return a + b;");
    console.log(sum(1, 2));
    
    console.log(eval("1 + 2"));
    ```

- 格式化字符串
    ```javascript
    String.prototype.format = function(...args) {
        if (args.length == 1 && typeof args[0] == "object") {
            let k = "", v = "";
            return this.replace(/{[A-Za-z]+}/g, (it, i) => {
                k = it.slice(1, -1);
                v = args[0][k];
                return typeof v != "undefined" ? v : "";
            })
        }
        return this.replace(/{(\d+)}/g, (it, i) => {
            return typeof args[i] != "undefined" ? args[i] : "";
        });
    };
    console.log("我是{0}，今年{1}了. {2}".format("zhgl", 42, 0));
    console.log("我是{name}，今年{age}了.".format({ name: "zhgl", age: 42 }));
    ```
    ```javascript
    // 数字或字符串，指定长度，不足则前补零
    ((num, len) => {
        return (Array(len).join(0) + num).slice(-len);
    })(12, 3); // "012"
    ```
    ```javascript
    // 正则替换为自增数列
    let i = 0;
    "00000000".replace(/\d/g, s => i++); // 012345678
    ```

- 格式化时间
    ```javascript
    Date.prototype.toString = function(fmt)   
    {
        var o = {
            "M+": date.getMonth() + 1,
            "d+": date.getDate(),
            "h+": date.getHours(),
            "m+": date.getMinutes(),
            "s+": date.getSeconds(),
            "q+": Math.floor((date.getMonth() + 3) / 3),
            "S": date.getMilliseconds()
        };
        if(/(y+)/.test(fmt)) {
            fmt = fmt.replace(RegExp.$1, (date.getFullYear() + "").substr(4 - RegExp.$1.length));
        }
        for(var k in o) {
            if(new RegExp("("+ k +")").test(fmt)) {
                fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
            }
        }
        return fmt;
    };
    new Date().toString("yyyy-MM-dd hh:mm:ss");
    ```

- 获取 url 参数
    ```javascript
    function getQueryParameter(name) {
        var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)", "i");
        var r = window.location.search.substr(1).match(reg);
        if (r != null) {
            return unescape(r[2]);
        }
        return null;
    }
    ```

- csv 转 json 对象
    ```javascript
    function parse(csv) {
        const COMMA_REGEX = /,(?=(?:[^\"]*\"[^\"]*\")*[^\"]*$)/g,
              QUOTES_REGEX = /^"(.*)"$/g;

        let lines = csv.split("\n"),
            result = [],
            headers = lines[0].split(COMMA_REGEX).map(h => h.replace(QUOTES_REGEX, "$1"));

        for(let i = 1; i < lines.length; i++) {
            let obj = {},
                currentline = lines[i].split(COMMA_REGEX);
            for(let j = 0; j < headers.length; j++) {
                obj[headers[j]] = currentline[j].replace(QUOTES_REGEX, "$1");
            }
            result.push(obj);
        }
        return result;
    }
    // parse(`id,name\n1,zhangsan\n`)
    ```

- 数组分页
    ```javascript
    function pagination(index, size, array) {
        var offset = (index - 1) * size;
        return (offset + size >= array.length) ? array.slice(offset, array.length) : array.slice(offset, offset + size);
    }
    // pagination(1, 10, [...])
    ```

- 生成器函数
    ```javascript
    function * gen(){
        yield 10;
        y = yield "foo";
        yield y;
    }

    var gen_obj = gen();
    console.log(gen_obj.next()); // { value: 10, done: false }
    console.log(gen_obj.next()); // { value: "foo", done: false }
    console.log(gen_obj.next(11)); // { value: 11, done: false }
    console.log(gen_obj.next()); // { value: undefine, done: true }
    ```

- [阻塞队列](https://www.it1352.com/1013354.html)
    ```javascript
    class AsyncBlockingQueue {
        constructor() {
            // invariant: at least one of the arrays is empty
            this.resolvers = [];
            this.promises = [];
        }
        _add() {
            this.promises.push(new Promise(resolve => {
                this.resolvers.push(resolve);
            }));
        }
        enqueue(t) {
            // if (this.resolvers.length) this.resolvers.shift()(t);
            // else this.promises.push(Promise.resolve(t));
            if (!this.resolvers.length) this._add();
            this.resolvers.shift()(t);
        }
        dequeue() {
            if (!this.promises.length) this._add();
            return this.promises.shift();
        }
        // now some utilities:
        isEmpty() { // there are no values available
            return !this.promises.length; // this.length == 0
        }
        isBlocked() { // it's waiting for values
            return !!this.resolvers.length; // this.length < 0
        }
        get length() {
            return this.promises.length - this.resolvers.length;
        }
    }

    let queue = new AsyncBlockingQueue();
    let a = await queue.dequeue(); // 这里会阻塞，直到 queue.enqueue 调用
    console.asset(a == "hello, world");

    // 新开启一个线程执行如下
    queue.enqueue("hello, world");
    ```

- [防抖、节流](https://www.jianshu.com/p/7f0d3785b54a)
    ```javascript
    // 防抖：在一定时间内，触发多次事件，只认第一次触发的，到了时间结束执行事件
    function throttle(fn, time) {
        let oldTime = 0,
            timer = null;
        return () => {
            const nowTime = new Date();
            if (nowTime - oldTime >= time) {
                fn();
                oldTime = nowTime;
            }
        }
    }

    // 节流：在一定时间内，触发多次事件，只认最后一次触发的并且重置时间，到了时间结束执行事件
    function debounce(fn, wait, immediate) { // 立即执行，停止触发 n 秒后，才可以重新触发执行
        let timer;
        return function () {
            if (timer) clearTimeout(timer);
            if (immediate) {
                // 如果已经执行过，不再执行
                var callNow = !timer;
                timer = setTimeout(() => {
                    timer = null;
                }, wait);
                if (callNow) {
                    fn.apply(this, arguments);
                }
            } else {
                timer = setTimeout(() => {
                    fn.apply(this, arguments);
                }, wait);
            }
        }
    }

    // containr.addEventListener("scroll", debounce(onScroll, 1000), false);
    ```

- 监听事件
    ```javascript
    (function () {
        window.addEventListener("click", function (event) {
            console.info("The event of click triggered on location (%s, %s) in browser.", event.clientX, event.clientY);
            console.info("The event of click triggered on location (%s, %s) in screen.", event.screenX, event.screenY);
        });
    })();
    ```

- [自定义事件](https://blog.csdn.net/jyb123/article/details/86574365)
    ```javascript
    // 添加一个事件监听器
    window.addEventListener("myEvent", function(e) {
        console.info("the data received is", e.detail);
    });
    
    // var event = new Event("myEvent");
    // 创建一个支持冒泡且不能被取消的事件
    var event = new Event("myEvent", {
        "bubbles": true,
        "cancelable": false
    });
    // 创建一个事件，并传递自定义数据
    var event = new CustomEvent("myEvent", {
        "detail": { // 自定义数据需要通过 CustomEvent.detail 属性传递
            "greeting": "hello, world"
        }
    });
    
    // 分发事件
    window.dispatchEvent(event);
    ```

- float to IEEE
    ```javascript
    ((f) => {
        var buf = new ArrayBuffer(4);
        (new Float32Array(buf))[0] = f;
        return (new Uint32Array(buf))[0];
    })(0.5); // 1056964608
    ```

- [类型化数组](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Typed_arrays)  
    类型化数组是一种类似数组的对象，并提供了一种用于访问原始二进制数据的机制。
    - ArrayBuffer
        ```javascript
        // 创建一个 16 字节固定长度的缓冲
        var buffer = new ArrayBuffer(16);
        
        // 获取数据的字节长度
        console.info(buffer.byteLength); // 16
        
        // 在实际开始操作这个缓冲之前，需要创建一个视图。我们将创建一个视图，此视图将把缓冲内的数据格式化为一个 32 位的有符号整数数组
        var int32View = new Int32Array(buffer);

        // 现在我们可以像普通数组一样访问该数组中的元素
        for (var i = 0; i < int32View.length; i++) {
            int32View[i] = i * 2;
        }
        // 该代码会将数组以0, 2, 4 和 6填充 （一共4个4字节元素，所以总长度为16字节）
        ```
        > - ArrayBuffer 又称类型化数组，是一个二进制数据的原始缓存区，无法直接读取或者写入（也就是说只能存放 0,1 并且不能直接修改和读取）
        > - 数组是放在堆中，ArrayBuffer 数组则把数据放在栈中（所以取数据时后者快）
        > - 可以根据需要传递类型化数组或者 DataView 对象来解释原始缓存区
        > - ArrayBuffer 初始化后固定大小，数组则可以自由增减。
    - 数据视图 DataView
    - Uint8Array
        ```javascript
        // 创建一个 Uint8Array
        let uint8array = new Uint8Array([1, -1, 999]);

        // Uint8Array 转 Array
        Array.from(uint8array); // [1, 255, 231] // 231 = 999 % 256
        [].slice.call(uint8array); // [1, 255, 231]
        ```

- 大额数值数据容量单位转换
    ```javascript
    ((bytes) => {
        if (bytes === 0) return "0 B";
        let k = 1000, // or 1024
            sizes = ["B", "KB", "MB", "GB", "TB", "PB", "EB", "ZB", "YB"],
            i = Math.floor(Math.log(bytes) / Math.log(k));
        return (bytes / Math.pow(k, i)).toPrecision(3) + " " + sizes[i];
    })(8164674); // 8.16 MB
    ```

- [锁](https://zhuanlan.zhihu.com/p/97188370)
    ```javascript
    class Lock {
        constructor() {
            this.locked = null;
        }
        async lock() {
            if (this.locked === null)
                this.locked = [];
            else
                await new Promise(res => this.locked.push(res));
        }
        unlock() {
            if (this.locked.length)
                this.locked.shift()();
            else
                this.locked = null;
        }
    }
    
    (async () => {
        const a = new Lock();
        await a.lock();
        console.log("locked");
        a.unlock();
    })().catch (err => {
        console.log(err);
    });
    ```

- 函数柯里化
    ```javascript
    function currying(reg) {
        return function(txt) {
            return reg.test(txt)
        }
    }
    
    let hasNumber = currying(/\d+/g);
    hasNumber("test1"); // true
    ```

- Web Worker  
    > 当在 HTML 页面中执行脚本时，页面是不可响应的，直到脚本已完成。Web worker 是运行在后台的 JavaScript，独立于其他脚本（在新的独立的线程中运行），不会影响页面的性能，同时也不能操作 DOM 元素。
    1. 创建 Web Worker 文件，如 greeting_worker.js
        ```javascript
        setInterval(function() {
            postMessage("hello, world"); // 向 HTML 页面传回一段消息
        }, 5000);
        ```
    2. HTML 页面中调用如下
        ```javascript
        if(typeof(Worker) === "undefined") {
            console.error("Sorry! No Web Worker support.");
        }

        let worker;
        if(typeof(worker) === "undefined") { // 检测是否存在 worker，如果不存在，创建一个新的 web worker 对象，然后运行 "greeting_worker.js" 中的代码
            worker = new Worker("demo_workers.js");
        }
        worker.onmessage = function(event) { // 当 web worker 传送消息时，会执行事件监听器中的代码。来自 web worker 的数据会存储于 event.data 中
            console.info("receive message", event.data);
        };

        worker.terminate(); // 终止 web worker，并释放浏览器/计算机资源
        worker = undefined; // 把 worker 变量设置为 undefined，在其被终止后，可以重复使用该代码
        ```

- Service Worker
    > Service Worker 基于 Web Worker，是运行在浏览器后台的 JS 程序，可以操控浏览器端的存储、网络请求、消息推送等。
    1. 创建 Service Worker 文件，如 service_worker.js
        ```javascript
        // 监听安装事件，install 事件一般是被用来设置你的浏览器的离线缓存逻辑
        this.addEventListener("install", function (event) {
            // 通过这个方法可以防止缓存未完成，就关闭 Service Worker
            event.waitUntil(
                // 创建一个名叫 v1 的缓存版本
                caches.open("v1").then(function (cache) {
                    // 指定要缓存的内容，地址为相对于跟域名的访问路径
                    return cache.addAll([
                        "./index.html"
                    ]);
                })
            );
        });

        // 注册 fetch 事件，拦截全站的请求
        this.addEventListener("fetch", function(event) {
             event.respondWith(
                 // 在缓存中匹配对应请求资源直接返回
                 caches.match(event.request)
             );
        });
        ```
    2. 在 HTML 页面中注册 Service Worker 如下
        ```javascript
        // 判断当前浏览器是否支持 Service Worker
        if ("serviceWorker" in navigator) {
            // 当页面加载完成就创建一个 Service Worker
            window.addEventListener("load", function () {
                // 创建并指定对应的执行内容
                navigator.serviceWorker.register("./service_worker.js", {
                    scope: "./" // 可选，可以用来指定 service worker 控制的内容的子目录。在这个例子里，我们指定了 "/"，表示根网域下的所有内容，这也是默认值。
                })
                    .then(function (registration) {
                        console.log("Service Worker registration successful with scope: ", registration.scope);
                    })
                    .catch(function (err) {
                        console.log("Service Worker registration failed: ", err);
                    });
            });
        }
        ```

- Shared Worker

- 重试机制 retry
    ```javascript
    const retry = (fn, times = [1000, 1500]) => {
        return new Promise(async (resolve, reject) => {
            let n = 0, s = false;
            while (true) {
                switch (n) {
                    case 0:
                        await fn(r => {
                            s = true;
                            resolve();
                        }, reject);
                        n = s == true ? 2 : 1;
                        break;
                    case 1:
                        let t = times.shift();
                        if (t) {
                            await new Promise(r => setTimeout(r, t));
                            i = 0;
                        } else {
                            i = 2;
                        }
                        break;
                    case 2:
                        return;
                }
            }
        });
    }
    retry(async (resolve, reject) => {
        await fetch("/").then(r => {
            resolve(r);
        }).catch(e => {
            reject(e);
        });
    });
    ```

- 延迟队列 DelayQueue

- js 函数转代码字符串
    ```javascript
    let func = n => `hello, ${n || "world"}`;
    func.toString() // 'n => `hello, ${n || "world"}`'
    ```

## [存储](https://www.sitepoint.com/client-side-storage-options-comparison/)

- DOM 节点（读写快，但页面刷新会丢失）
    > 通过以 "data-" 为前缀的数据集属性存储和读取，对象类型需要序列化为字符串存储。
    ```javascript
    const main = document.querySelector("main");
    
    // 赋值
    main.dataset.name = "zhangsan";
    main.dataset.user = JSON.stringify({ name: "zhangsan" });
    
    // 取值
    console.info(JSON.parse(main.dataset.user).name);
    ```

- Local Storage 和 Session Storage（每个域名下只有 5 MB 空间，值需要序列化为字符串类型）
    ```javascript
    // localStorage 用于持久化的本地存储，存储资料在客户端（client）的浏览器上，除非主动删除数据，否则数 据是永远不会过期的。
    localStorage.setItem("name", "zhangsan");
    localStorage.getItem("name");
    localStorage.removeItem("name");
    
    // sessionStorage 用于本地存储一个会话中的数据，这些数据只有在同一个会话中的页面才能访问并且会话结束，例如关闭窗口后，数据也会随之被销毁。它是一种会话级别的存储。
    sessionStorage.setItem("name", "zhangsan");
    sessionStorage.getItem("name");
    ```

- IndexedDB（索引数据库）（取决于设备，最少 1 GB 空间，最多剩余磁盘空间的 60%，读写快）
    ```javascript
    (async () => {
        const db = await new Promise((resolve, reject) => {
            const request = indexedDB.open("myDB", 1.0);
            request.onsuccess = e => {
                resolve(e.target.result);
            };
            request.onerror = e => {
                console.error(`indexedDB error: ${ e.target.errorCode }`);
            };
            request.onupgradeneeded = e => {
                const store = e.target.result.createObjectStore("todo", { keyPath: "id", autoIncrement: true });
            };
        });
    
        db.transaction(["todo"], "readwrite")
            .objectStore("todo")
            .add({ task: "do something" })
            .onsuccess = () => console.log("added");
    
        db.transaction(["todo"], "readonly")
            .objectStore("todo")
            .get(1)
            .onsuccess = data => console.log(data.target.result);
        // { id: 1, task: "do something" }
    })();
    ```

- Cache API（适用于缓存网络请求如网页等）
    ```javascript
    (async () => {
        const cache = await caches.open("myCache");

        // 请求并保存
        await cache.add("/greeting");
        await cache.add(new Request("/greeting", {
            method: "GET",
            headers: new Headers({
                "Content-Type": "application/json"
            })
        }));
    
        // 查询
        console.info("the cache response context is", await (await caches.match("/greeting"))).text());
        
        // 请求并更新
        const response = await fetch("/greeting");
        await cache.put("/greeting", response);
        
        // 删除
        await cache.delete("/greeting");

        await cache.delete("myCache");
    })();
    ```

- 文件系统访问 API
    ```javascript
    // 使用 `try...catch` 可以捕获用户取消选择时抛出的错误，如果你对错误不在意，不捕获也行
    try {
        const [handle] = await showOpenFilePicker({
            multiple: false, // 只选择一个文件
            types: [
               {
                   description: "Navlang Files",
                   accept: {
                      "text/x-navlang": ".nav"
                   }
               }
            ],
            excludeAcceptAllOption: true
        });
    } catch (e) {
        if (e.message.indexOf("The user aborted a request") === -1) {
            console.error(e);
            return;
        }
    }

    // 如果没有选择文件，就不需要继续执行了
    if (!handle) {
        return;
    }

    // 这里的 options 用来声明对文件的权限，能否写入
    const options = {
        writable: true,
        mode: "readwrite"
    };
    // 然后向用户要求权限
    if ((await handle.queryPermission(options)) !== "granted" && (await handle.requestPermission(options)) !== "granted") {
        alert("Please grant permissions to read & write this file.");
        return;
    }

    // 前面获取的是 FileHandle，需要转换 File 才能用
    const file = await handle.getFile();
    // 接下来，`file` 就是普通 File 实例，你想怎么处理都可以，比如，获取文本内容
    const code = await file.text();
    ```
    ```javascript
    async function save(blob) {
        // create handle to a local file chosen by the user
        const handle = await window.showSaveFilePicker();

        // create writable stream
        const stream = await handle.createWritable();

        // write the data
        await stream.write(blob);

        // save and close the file
        await stream.close();
    }
    ```

- Web SQL（每个域名下 5 MB 空间，读写速度慢）
    > Chrome 浏览器数据库（SQLite）位置 "C:\Users\%username%\AppData\Local\Google\Chrome\User Data\Default\databases\Databases.db"
    ```javascript
    var db = openDatabase("MyData", "", "My Database", 102400);

    // 创建一个数据库表，里面有 3 个字段 
    db.transaction(function(tx){ 
        tx.executeSql("CREATE TABLE IF NOT EXISTS InfoData(name TEXT, info TEXT, time INTEGER)", []); 
    }

    // 显示所有
    db.transaction(function(tx) {      
        // 然后定义了一个回调函数，表明对于结果集的处理 
        tx.executeSql("SELECT * FROM InfoData", [], function(tx, rs) { 
            // 遍历结果集，对于每一行，依次调用 showData 来在 table 上创建对于的 html 文本 
            for(var i = 0; i < rs.rows.length; i++){ 
                // 对于 item(i)，也就是某一行记录，我们显示其内容到页面的表格中（构建对应的 HTML 片断） 
                alert(rs.rows.item(i)); 
            } 
        }); 
    } 

    // 增加数据
    db.transaction(function(tx) {      
        // 插入的语句是个模板语句 
        // 插入成功的回调就是在控制台上输入一行日志 
        tx.executeSql("INSERT INTO InfoData VALUES(?, ?, ?)", [name, info, time], function(tx, rs) { 
            console.log("成功保存数据!"); 
        }, 
        // 插入失败的回调就是在控制台上输入一行错误日志 
        function(tx, error) { 
            console.log(error.source + "::" + error.message); 
        }); 
    }
    ```

- Cookie（每个域名下 80 KB 空间，不同浏览器对每个域名下 Cookie 有个数限制，不应超过 20 个，每个不超过 4 KB）
    ```javascript
    document.cookie = `user=${encodeURIComponent(JSON.stringify({ name: "zhangsan" }))}`;

    document.cookie = "username=zhangsan; expires=Session; domain=.example.com; path=/; secure=true; priority=High;";
    ```

- window.name
    ```javascript
    window.name = JSON.stringify({ name: "zhangsan" });
    
    console.info(JSON.parse(window.name).name);
    ```

## 网络

- XMLHTTPRequest
    ```javascript
    var xmlhttp = null;
    //create xml http request
    if (window.XMLHttpRequest) {
        // for all new browsers
        xmlhttp = new XMLHttpRequest();
    } else if (window.ActiveXObject) {
        // for IE5 and IE6
        xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
    }
    //send xml http request
    if (xmlhttp != null) {
        xmlhttp.onreadystatechange = function(){
            if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
                // 4 = "loaded"
                // 200 = OK
                console.info("response text: " + xmlhttp.responseText);
            }
        };
        xmlhttp.open("GET", url, true);
        xmlhttp.send(null);
    } else {
        console.error("current browser does not support XMLHTTP.");
    }
    ```

- fetch
    ```javascript
    fetch("https://127.0.0.1/greeting", {
        method: "post",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify({
            name: "zhangsan"
        })
    })
    .then(response => response.json())
    .then(data => {
        console.info("response is", data);
    })
    .catch(error => console.log("error is", error));
    ```
    ```javascript
    await fetch("https://www.baidu.com").then(re => re.text());
    await fetch("http://127.0.0.1/greeting", { method: "post", headers: { "Content-Type": "application/json" }, body: JSON.stringify({ name: "zhangsan" })}).then(re => re.json());
    ```
    ```javascript
    fetch("http://127.0.0.1/greeting", {
        method: "post",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify({ name: "zhangsan" })
    })
    .then(re => {
        if (re.ok) {
            return re.json();
        }
        throw new Error("Request reject with status " + re.status);
    })
    .then(data => {
        console.info("response is", data);
    });
    ```

- [WebSocket](http://www.ruanyifeng.com/blog/2017/05/websocket.html)
    ```javascript
    var ws = new WebSocket("wss://echo.websocket.org");

    ws.onopen = function(evt) { 
        console.log("Connection opened."); 
        ws.send("hello, world");
    };

    ws.onmessage = function(evt) {
        console.log("Received message:", evt.data);
        ws.close();
    };

    ws.onclose = function(evt) {
        // evt.code // 错误码，number 类型
        // evt.reason // 断开原因，string 类型
        // evt.wasClean // 是否正常断开，boolean 类型
        console.log("Connection closed.");
    };
    ```

- [EventSource 服务端推送事件](https://www.jianshu.com/p/622a3549728a)
    ```javascript
    var es = new EventSource("/event/source");

    // 收到服务器发生的事件时触发
    evtSource.onmessage = function (e) {
        console.log(e.data);
    }
    // 成功与服务器发生连接时触发
    es.onopen = function () {
        console.log("Server open")
    } 
    // 出现错误时触发
    es.onerror = function () {
        console.log("Error")
    }

    // 自定义事件
    es.addEventListener("myEvent", function (e) {
        console.log(e.data);
    }, false);
    ```
    nodejs 服务器端实现
    > - 需设置响应消息头 "Content-Type" 设置为 "text/event-stream"
    > - 报文格式为 "字段:..."，每个事件之间通过空行来分隔
    >   - 字段为空白，表示该行是注释，会在处理时被忽略
    >   - event: 表示该行用来声明事件的类型。浏览器在收到数据时，会产生对应类型的事件
    >   - data: 消息的数据字段。如果该条消息包含多个 data 字段，则客户端会用换行符把它们连接成一个字符串来作为字段值
    >   - id: 事件 ID，会成为当前 EventSource 对象的内部属性"最后一个事件 ID"的属性值
    >   - retry: 一个整数值，指定了重新连接的时间(单位为毫秒)，如果该字段值不是整数，则会被忽略。默认浏览器每隔 3 秒断线重连
    ```javascript
    var http = require("http");
    var fs = require("fs");

    http.createServer(function (req, res) {
        if(req.url === "/sendMessage") {
            res.writeHead(200, {
                "Content-Type": "text/event-stream" //设置头信息
            });

            setInterval(function () {
                res.write(
                    // 事件一
                    "data:" + new Date().toLocaleTimeString() + "\n\n" + //必须 "\n\n" 结尾
                    // 事件二
                    ": '这是注释！'" + "\n" +
                    "event: myEvent" + "\n" + 
                    "data:" + new Date().toLocaleString() + "\n\n"
                );
            }, 1000);
        } else {
            fs.readFile("./index.html", function (err, content) {
                res.writeHead(200, {"Content-Type": "text/html"});
                res.end(content, "utf-8");
            });
        }
    }).listen(3000);
    ```

- WebRTC

## 第三方库

### Jquery

```javascript
// http://www.youdict.com/root/root.php
$("table a[target='_blank']").each((index, obj) => {
    console.info(obj.innerHTML.replace(/,/g, ""));
});

var arr = new Array(0);
$("table a[target='_blank']").each((index, obj) => {
    // arr = arr.concat(obj.innerHTML.replace(/,/g, "").split(" "));
    arr = arr.concat(obj.innerHTML.replace(/[^a-z]+/g, "").split(" "));
});
var set = new Set(arr); // ES6 中支持
arr = Array.from(set); // 去重
console.info(arr);
// document.body.innerHTML = arr.join(" ");

Array.prototype.remove = function(val) { // 定义数组的 remove 拓展方法
    var index = this.indexOf(val);
    if (index > -1) {
        this.splice(index, 1);
    }
};
for (var i of arr) {
    if (i.length > 4 || i.length < 2) {
        arr.remove(i);
    }
}
```

```javascript
$.get(url, data, success(response, status, xhr), dataType);

$.post(url, data, success(data, textStatus, jqXHR), dataType);

$.ajax({
    type: "POST",
    url: url,
    data: data,
    success: success,
    dataType: dataType
});
```

### 使用 vConsole 调试移动端 web

```html
<!-- 页面中嵌入以下代码，则页面底部会生成 console 框工具 -->
<script src="https://cdn.bootcdn.net/ajax/libs/vConsole/3.3.4/vconsole.min.js"></script>
<script>var vConsole = new VConsole();</script>
```

### [lunr.js](https://github.com/olivernn/lunr.js/)

```javascript
var idx = lunr(function () {
    this.field("title")
    this.field("body")

    this.add({
        "title": "Twelfth-Night",
        "body": "If music be the food of love, play on: Give me excess of it…",
        "author": "William Shakespeare",
        "id": "1"
    })
});

idx.search("love");

// idx2 = lunr.Index.load(idx.toJSON());
idx2 = lunr.Index.load(JSON.parse(JSON.stringify(idx)));

idx2.search("love");
```

### vue

- [v-clock](https://www.jianshu.com/p/f56cde007210?utm_source=oschina-app)  
    当网络较慢，网页还在加载 Vue.js ，而导致 Vue 来不及渲染，这时页面就会显示出 Vue 源代码。我们可以使用 v-cloak 指令来解决这一问题

- 外联引用 vant: <van-cell-group> 组件渲染出错  
    应使用闭合标签的这种写法如 <van-cell></van-cell>。参考 https://github.com/youzan/vant/issues/800

### [socket.io](https://socket.io/get-started)

- websocket 服务端
    1. 安装
        ```bash
        # npm install express@4
        npm install socket.io
        ```
    2. 创建 server.js 如下
        ```javascript
        // const express = require("express");
        // const app = express();

        const http = require("http");
        const server = http.createServer(
        //     app
        );

        const io = require("socket.io")(server);

        // app.get("/", (req, res) => {
        //     res.sendFile(__dirname + "/index.html");
        // });

        io.on("connection", (socket) => { // 当有客户端连接时触发
            console.log("a user connected");

            socket.on("chat message", (msg) => { // 接受客户端 "chat message" 事件
                console.log("message: " + msg);
            });

            socket.on("disconnect", () => { // 当有客户端断开连接时触发
                console.log("user disconnected");
            });
        });

        server.listen(3000, () => {
            console.log("listening on *:3000");
        });

        setInterval(() => {
            // 向所有连接广播发送 "some event" 事件，并传递对象参数
            io.emit("some event", {
                greeting: "hello, all clients"
            });
        }, 5000); // 定时器，每 5 秒执行一次
        ```
    3. 运行
        ```bash
        node server.js
        ```

- websocket 客户端
    ```html
    <html>
    <head></head>
    <body>
        <script src="https://cdn.bootcdn.net/ajax/libs/socket.io/3.1.3/socket.io.js"></script>
        <script>
            // var socket = io("http://127.0.0.1:3000/");
            var socket = io();

            socket.emit("chat message", "hello, server");

            socket.on("some event", (msg) => {
                console.log("message: " + msg);
            });
        </script>
    </body>
    </html>
    ```

### [js-sha256](https://github.com/emn178/js-sha256)

1. 引入
    ```html
    https://cdn.bootcdn.net/ajax/libs/js-sha256/0.9.0/sha256.min.js
    ```
2. 语法
    ```javascript
    sha256("The quick brown fox jumps over the lazy dog"); // d7a8fbb307d7809469ca9abcb0082e4f8d5651e46d3cdb762d02d0bf37c9e592
    ```

### jsencrypt

1. 引入 jsencrypt
    ```html
    <script src="https://cdn.bootcdn.net/ajax/libs/jsencrypt/3.1.0/jsencrypt.min.js"></script>
    ```
2. 语法
    ```javascript
    let PUBLIC_KEY = "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC8HMr2CBpoZPm3t9tCVlrKtTmI4jNJc7/HhxjIEiDjC8czP4PV+44LjXvLYcSV0fwi6nE4LH2c5PBPEnPfqp0g8TZeX+bYGvd70cXee9d8wHgBqi4k0J0X33c0ZnW7JruftPyvJo9OelYSofBXQTcwI+3uIl/YvrgQRv6A5mW01QIDAQAB", // 公钥
        PRIVATE_KEY = "MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBALwcyvYIGmhk+be320JWWsq1OYjiM0lzv8eHGMgSIOMLxzM/g9X7jguNe8thxJXR/CLqcTgsfZzk8E8Sc9+qnSDxNl5f5tga93vRxd5713zAeAGqLiTQnRffdzRmdbsmu5+0/K8mj056VhKh8FdBNzAj7e4iX9i+uBBG/oDmZbTVAgMBAAECgYEAmgNU5NTDkj9B+Pnt6UU8doSjw3+3j+bV2K2yS3QUOvAUus/Ax7x6ktjWxzCXvDY9IfUil2RNv9vtKEAqYLCWjc+lf8PV/yH1b7NEgyeAPBXtAJRoOnmYL2bdPW92kP9KgxJruF6Dz/C5AmMOncsvq8ABD+9Darn4p8dwj2ZC4O0CQQDf/AHmZsQokEItfCy4mHS9UbxbfIhEUv1ApPh/+Sr7NkJkHWYCtBQo+8jKO6zurAZQgWBPD1XX2UE4R+VIiZazAkEA1wAqtMvGhccyRZr+6kpkpDIa8+9jOE+nGUzqTDvgCID6as8AzOONFVVK6m/UUqkhcJ8Qu1pF36BGojy5BX2KVwJBAJSFpbji0hXXupowqfLp3RcgmNbNWAp+QUJZYhJx5cdYbmO2fssyH+AhPT6knYJR/YnqkDM8hv6vKCkqu2YDHjMCQAOA8TE5EOclM+CGghj3VWSHnIDVKdzFD4gOBNNxNlltIKeU8AJmwunSFgJ0CBXAw9a+ANvMwM7AIeaK7sj0HskCQAvxfDCq7gaNx+pfu0FHG8Gix08A/A6foggBl1fVu+L9sr9ZuOQ3HbXnl28F9ewuB9xdjnLUDjp7W7U0pB+vKoQ=", // 私钥

    // 使用公钥加密
    let encrypt = new JSEncrypt();
    encrypt.setPublicKey("-----BEGIN PUBLIC KEY-----" + PUBLIC_KEY + "-----END PUBLIC KEY-----");
    let encrypted = encrypt.encrypt("hello, world");
    console.log("加密后数据", encrypted);

    // 使用私钥解密
    let decrypt = new JSEncrypt();
    decrypt.setPrivateKey("-----BEGIN RSA PRIVATE KEY-----" + PRIVATE_KEY + "-----END RSA PRIVATE KEY-----");
    let decrypted = decrypt.decrypt(encrypted);
    console.log("解密后数据", decrypted);

    console.assert("hello, world" === decrypted);
    ```

### [multiavatar](https://github.com/multiavatar/Multiavatar)

1. 引入
    ```html
    <script src="https://cdn.jsdelivr.net/npm/@multiavatar/multiavatar/multiavatar.min.js"></script>
    ```
2. 语法
    ```javascript
    multiavatar("meter") // "<svg ...</svg>"
    ```
    ```javascript
    let image = new Image();
    image.src = "data:image/svg+xml," + multiavatar("Binx Bond");
    ```
    > 好看的头像："appinn"、"meter"

### tensorflow.js

### 使用 mark.js 标注网页内容（关键词前后包裹 mark 标签，如 "<mark>hello</mark>"）

1. 引入
    ```html
    <script src="https://cdn.jsdelivr.net/mark.js/latest/"></script>
    ```
2. 语法
    ```javascript
    new Mark(document.body).mark("hello");
    ```

### [qrcodejs](https://github.com/davidshimjs/qrcodejs)

1. 引入
2. 语法
    ```javascript
    var div = document.createElement("div");
    document.body.appendChild(div);

    new QRCode(div, "hello, world");
    ```

### [Monaco Editor](https://github.com/microsoft/monaco-editor)

1. 示例
    ```html
    <!DOCTYPE html>
    <html>

    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no"><!-- 网页的宽度自动适应手机屏幕的宽度 -->
        <title>Cube</title>
        <link rel="stylesheet" data-name="vs/editor/editor.main" href="https://cdn.bootcdn.net/ajax/libs/monaco-editor/0.31.0/min/vs/editor/editor.main.css">
        <style>
            * {
                margin: 0;
                padding: 0;
            }

            html,
            body {
                width: 100%;
                height: 100%;
            }

            html {
                overflow: hidden;
            }
        </style>
    </head>

    <body>
        <div id="container" style="width: 100%; height: 100%;"></div>

        <script>var require = { paths: { "vs": "https://cdn.bootcdn.net/ajax/libs/monaco-editor/0.31.0/min/vs" } };</script>
        <script src="https://cdn.bootcdn.net/ajax/libs/monaco-editor/0.31.0/min/vs/loader.js"></script>
        <script src="https://cdn.bootcdn.net/ajax/libs/monaco-editor/0.31.0/min/vs/editor/editor.main.nls.js"></script>
        <script src="https://cdn.bootcdn.net/ajax/libs/monaco-editor/0.31.0/min/vs/editor/editor.main.js"></script>

        <script src="https://cdn.bootcdn.net/ajax/libs/typescript/4.5.4/typescript.min.js"></script>

        <script>
            (function () {
                // compiler options
                monaco.languages.typescript.typescriptDefaults.setCompilerOptions({
                    target: monaco.languages.typescript.ScriptTarget.ES2016,
                    allowNonTsExtensions: true,
                    moduleResolution: monaco.languages.typescript.ModuleResolutionKind.NodeJs,
                    module: monaco.languages.typescript.ModuleKind.CommonJS,
                    noEmit: true,
                    // noLib: true,
                    experimentalDecorators: true,
                    typeRoots: ["node_modules/@types"]
                });

                // diagnostics options
                // monaco.languages.typescript.typescriptDefaults.setDiagnosticsOptions({
                //     noSemanticValidation: false, // 关闭语义校验，默认为 false
                //     noSyntaxValidation: false // 关闭语法校验，默认为 false
                // });

                // extra libraries
                monaco.languages.typescript.typescriptDefaults.addExtraLib(`/*\n * Post method\n */\ndeclare function Post(path: string): Function;`);
                monaco.languages.typescript.typescriptDefaults.addExtraLib(`export let user = { name: "zhangsan" };`, "node_modules/@types/user/index.d.ts");

                let editor = monaco.editor.create(document.getElementById("container"), {
                    language: "typescript",
                    theme: "vs-dark",
                    value: "// code here",
                    model: monaco.editor.createModel("", "typescript", new monaco.Uri("main.tsx")),
                    options: {
                        selectOnLineNumbers: true,
                        roundedSelection: false,
                        readOnly: false,
                        cursorStyle: "line",
                        automaticLayout: true
                    }
                });
                
                editor.setValue(`import { user } from "user";\n\nclass Greeting {\n\t@Post("/greeting")\n\tpublic greeting(name: string = "world"): string {\n\t\treturn \`hello, \${user?.name ?? name}\`;\n\t}\n}`);

                document.onkeydown = function (e) {
                    let keyCode = e.keyCode || e.which || e.charCode,
                        ctrlKey = e.ctrlKey || e.metaKey;
                    if (ctrlKey && keyCode == 82) { // Ctrl + R
                        let tsSrc = editor.getValue(),
                            jsSrc = ts.transpileModule(tsSrc, {
                            compilerOptions: { module: ts.ModuleKind.CommonJS }
                        })?.outputText;

                        var fd = window.open("about:blank", "", "width=600,height=450");
                        setTimeout(() => {
                            fd.document.open()
                            fd.document.write(`<!DOCTYPE html><html><body><script>console.info=function(message, ...optionalParams){document.write(message + "<br/>");}<\/script><script>${jsSrc}<\/script><\/body><\/html>`);
                            fd.document.close();
                        }, 50);

                        e.preventDefault();
                        return false;
                    }
                }
            })();
        </script>
    </body>
    </html>
    ```

2. 代码提示
    ```javascript
    monaco.languages.registerCompletionItemProvider("sql", {
        provideCompletionItems: function (model, position) {
            // 获取提示符 sym
            const line = position.lineNumber, // 获取当前行数
                column = position.column, // 获取当前列数
                content = model.getLineContent(line); // 获取当前输入行的所有内容
            const sym = content[column - 2]; // 通过下标来获取当前光标后一个内容，即为刚输入的内容
            let textUntilPosition = model.getValueInRange({
                startLineNumber: 1,
                startColumn: 1,
                endLineNumber: position.lineNumber,
                endColumn: position.column
            });
            let word = model.getWordUntilPosition(position);
            let range = {
                startLineNumber: position.lineNumber,
                endLineNumber: position.lineNumber,
                startColumn: word.startColumn,
                endColumn: word.endColumn
            };

            let suggestions = [];
            if (sym === "$") {
                // ...
                // 拦截到用户输入 $，开始设置提示内容，同 else 中代码一致，自行拓展
            } else {
                // 直接提示，以下为 sql 语句关键词提示
                let sqlStr = ["SELECT", "FROM", "WHERE", "AND", "OR", "LIMIT", "ORDER BY", "GROUP BY", "LEFT", "ON", "if(){}", "for(){}", "size", "get()", "substring", "return"];
                for (let i in sqlStr) {
                    suggestions.push({
                        label: sqlStr[i], // 显示的提示内容
                        kind: monaco.languages.CompletionItemKind["Function"], // 用来显示提示内容后的不同的图标
                        insertText: sqlStr[i], // 选择后粘贴到编辑器中的文字
                        detail: "", // 提示内容后的说明
                        range: range
                    });
                }
            }
            return {
                suggestions: suggestions
            };
        },
        triggerCharacters: ["$", ""]
    });
    ```

### 使用 [Mock.js](http://mockjs.com/examples.html) 生成随机数据，拦截 ajax 请求

1. 引入 mockjs
    ```html
    <script src="https://cdn.bootcdn.net/ajax/libs/Mock.js/1.0.0/mock-min.js"></script>
    ```
2. 语法
    ```javascript
    Mock.mock("@paragraph");
    Mock.mock("@sentence");

    Mock.mock({
        "users|20-30": [{
            "id|+1": 1,
            "name": "@name",
            "cname": "@cname",
            "gender": "@pick(['male', 'female'])",
            "birth": "@date('yyyy-MM-dd')",
            "telephone": /^1[385][1-9]\d{8}/,
            "email": "@email",
            "address": "@province" + "@city" + "@county"
        }]
    });
    ```
    ```javascript
    Mock.mock("/greeting", "post", function (options) {
        // console.log(options);
        return {
            "greeting": "hello, world"
        }
    });
    $.post("/greeting", {
        name: "zhangsan"
    }).then(res => console.log(res));

    Mock.mock("/greeting2", function (options) {
        return {
            "_status": 404,
            "_headers": {
                
            }
        }
    });
    ```

### [decimal.js](https://github.com/MikeMcl/decimal.js)

1. 引入
    ```html
    <script src="https://cdn.bootcdn.net/ajax/libs/decimal.js/10.3.1/decimal.min.js"></script>
    ```
    > nodejs 安装
    > ```bash
    > npm install --save decimal.js
    > ```
2. 语法
    ```javascript
    // nodejs 需引入 decimal.js 模块
    // var Decimal = require("decimal.js");

    // 加法
    var a = 0.1;
    var b = 0.2;
    console.log(a.toString(2)); // 0.0001100110011001100110011001100110011001100110011001101
    console.log(a + b); // 0.30000000000000004
    console.log(new Decimal(a + "").add(new Decimal(b)).toNumber()); // 0.3
    
    // 减法
    var a = 1.0;
    var b = 0.7;
    console.log(a - b); // 0.30000000000000004
    console.log(new Decimal(a).sub(new Decimal(b)).toNumber()); // 0.3
    
    // 乘法
    var a = 1.01;
    var b = 1.003;
    console.log(a * b); // 1.0130299999999999
    console.log(new Decimal(a).mul(new Decimal(b)).toNumber()); // 1.01303
    
    // 除法
    var a = 0.029;
    var b = 10;
    console.log(a / b); // 0.0029000000000000002
    console.log(new Decimal(a).div(new Decimal(b)).toNumber()); // 0.0029
    ```

## 其他

- 使用百度 api 查询经纬度位置信息
    ```javascript
    // # npm install node-fetch
    // const fetch = require("node-fetch");
    (async function (latitude, longitude) {
        return await fetch(`http://api.map.baidu.com/geocoder/v2/?ak=wLyevcXk5QY36hTKmvV5350F&callback=renderReverse&location=${latitude},${longitude}s&output=json&pois=0`, {
            method: "GET"
        }).then(r => r.text()).then(r => JSON.parse(r.replace("renderReverse&&renderReverse(", "").replace(/\)$/, "")));
    })(39.916527, 116.397128);
    ```

# 参考

- [高薪的web前端工程师必会的19 个 JavaScript 简写方法](https://blog.csdn.net/fdsdidgf/article/details/102566928)

<details>

<summary>优雅的语法</summary>

```javascript
// 多个条件判断
// long
if (x === "a" || x === "b" || x === "c" || x === "d") {
    // todo
}
// short
if (["a", "b", "c", "d"].includes(x)) {
    // todo
}

// 三目运算符
// long
let flag
if (x > 10) {
    flag = true
} else {
    flag = false
}
// short
let flag = x > 10 ? true : false

// 变量声明
// long
let a
let b = 1
// short
let a, b = 1

// 空/未定义检查和分配默认值
// long
if (test1 !== null || test1 !== undefined || test1 !== "") {
    let test2 = test1;
} else {
    let test2 = ""
}
// short
let test2 = test1 || ""

// 给多个变量赋值
//long 
let test1, test2, test3;
test1 = 1;
test2 = 2;
test3 = 3;
//Short 
let [test1, test2, test3] = [1, 2, 3];

// 赋值运算符的简写
// long
test1 = test1 + 1;
test2 = test2 - 1;
test3 = test3 * 20;
// short
test1++;
test2--;
test3 *= 20;

// 真值判断
// long
if (test1 === true)
// short
if (test1)

// 多条件的与/或运算
//long 
if (test1) {
    callMethod();
}
//short 
test1 && callMethod();

// forEach
// long
for (var i = 0; i < testList.length; i++)
// short
testList.forEach(item => console.log(item))

// 比较返回值
// long
let test;
function checkReturn() {
    if (!(test === undefined)) {
        return test;
    } else {
        return callMe("test");
    }
}
var data = checkReturn();
console.log(data); //output test
function callMe(val) {
    console.log(val);
}
// short
function checkReturn() {
    return test || callMe("test");
}

// 箭头函数
// long 
function add(a, b) {
    return a + b;
}
// short 
const add = (a, b) => a + b;

// 短函数调用
// long
function test1() {
    console.log("test1");
};
function test2() {
    console.log("test2");
};
var test3 = 1;
if (test3 == 1) {
    test1();
} else {
    test2();
}
// short
(test3 === 1 ? test1 : test2)();

// switch
// long
switch (data) {
    case 1:
        test1();
        break;

    case 2:
        test2();
        break;

    case 3:
        test();
        break;
    // And so on...
}
// short
var data = {
    1: test1,
    2: test2,
    3: test
};
data[something] && data[something]();

// 默认参数
// long
function add(test1, test2) {
    if (test1 === undefined)
        test1 = 1;
    if (test2 === undefined)
        test2 = 2;
    return test1 + test2;
}
// short
add = (test1 = 1, test2 = 2) => (test1 + test2);
add() // output: 3

// 参数必传校验
// long
function hello(obj) {
    let { name, age } = obj
    if (!name) {
        console.warn("name is null, pls check!")
        return ""
    }
    if (!age) {
        console.warn("age is null, pls check!")
        return ""
    }
    return `${name}: ${age}`
}
// short
function hello(obj) {
    let { name = required("name"), age = required("age") } = obj
    return `${name}: ${age}`
}
function required(key) {
    console.warn(`${key} is null, pls check!`)
}

// 扩展运算符
// long
const data = [1, 2, 3];
const test = [4, 5, 6].concat(data);
// short
const data = [1, 2, 3];
const test = [4, 5, 6, ...data];
console.log(test); // [ 4, 5, 6, 1, 2, 3]

// 克隆
// long
const test1 = [1, 2, 3];
const test2 = test1.slice()
// short
const test1 = [1, 2, 3];
const test2 = [...test1];

// 模板字符串/模板字面量
// long
const welcome = "Hi " + user + " " + name + "."
// short
const welcome = `Hi ${user} ${name}`;

// 对象属性赋值
let test1 = "a";
let test2 = "b";
// long
let obj = { test1: test1, test2: test2 };
// short 
let obj = { test1, test2 };

// 字符串转换成数字
// long
let test1 = parseInt("123");
let test2 = parseFloat("12.3");
// short
let test1 = +"123";
let test2 = +"12.3";

// Array.find
const data = [{
    type: "test1",
    name: "abc"
},
{
    type: "test2",
    name: "cde"
},
{
    type: "test1",
    name: "fgh"
},
]
// long
function findtest1(name) {
    for (let i = 0; i < data.length; ++i) {
        if (data[i].type === "test1" && data[i].name === name) {
            return data[i];
        }
    }
}
//shorthand
filteredData = data.find(data => data.type === "test1" && data.name === "fgh");
console.log(filteredData); 

// 多条件判断
// long
if (type === "test1") {
    test1();
}
else if (type === "test2") {
    test2();
}
else if (type === "test3") {
    test3();
}
else if (type === "test4") {
    test4();
} else {
    throw new Error("Invalid value " + type);
}
// short
const types = {
    test1: test1,
    test2: test2,
    test3: test3,
    test4: test4
};
let func = types[type];
(!func) && throw new Error("Invalid value " + type); func();

// 索引查找
// long
if (arr.indexOf(item) > -1) { // item found 
}
if (arr.indexOf(item) === -1) { // item not found
}
// short
if (~arr.indexOf(item)) { // item found
}
if (!~arr.indexOf(item)) { // item not found
}
// 按位~运算符将返回非-1的真实值。取反就像做~一样简单。另外，我们也可以使用include()函数
if (arr.includes(item)) { 
    // true if the item found
}

// Object.entries()
const data = { test1: "abc", test2: "cde", test3: "efg" };
const arr = Object.entries(data);
console.log(arr); // [[ "test1", "abc" ], [ "test2", "cde" ], [ "test3", "efg" ]]

// Object.values()
const data = { test1: "abc", test2: "cde" };
const arr = Object.values(data);
console.log(arr); // [ "abc", "cde"]

// 重复一个字符串多次
// long 
let test = "";
for (let i = 0; i < 5; i++) {
    test += "test ";
}
console.log(str); // test test test test test 
// short 
"test ".repeat(5);

// 在数组中查找最大值和最小值
const arr = [1, 2, 3]; 
Math.max(…arr); // 3
Math.min(…arr); // 1
Math.min.apply(null, arr); // 1

// 使用展开运算符号 `...` 有条件地向对象快速添加属性
const condition = true;
const person = {
  id: 1,
  name: "John Doe",
  ...(condition && { age: 16 }),
};

// 合并对象
let a = { 1: 1, 2: 2, 3: 0 },
    b = { 3: 3, 4: 4 };
console.log({ ...b, ...a, 5: 5 }); // { 1: 1, 2: 2, 3: 0, 4: 4, 5: 5 } // 相同属性 3，后添加的(a: { 3: 0 })会覆盖之前同名属性(b: { 3: 3 })值，最终为 { 3: 0 }

// 使用 `in` 关键字来检查对象中是否存在某个属性
const person = { name: "前端小智", salary: 1000 };
console.log("salary" in person); // true
console.log("age" in person); // false

// 使用 `["key name"]` 动态键设置对象属性
const dynamic = "flavour";
var item = {
  name: "前端小智",
  [dynamic]: "巧克力"
}
console.log(item); // { name: "前端小智", flavour: "巧克力" }
const keyName = "name";
console.log(item[keyName]); // returns "前端小智"

// 使用 `:` 来对解构的属性进行重命名
const person = { id: 1, name: "前端小智" };
const { name: personName } = person;
console.log(personName); // "前端小智"
// 用动态键来解构属性
const templates = {
  "hello": "Hello there",
  "bye": "Good bye"
};
const templateName = "bye";
const { [templateName]: template } = templates;
console.log(template); // Good bye

// 空值合并 `??` 操作符，当它的左侧操作数为 null 或 undefined 时，它返回右侧的操作数，否则返回其左侧的操作数
const foo = null ?? "Hello";
console.log(foo); // "Hello"
const bar = "Not null" ?? "Hello";
console.log(bar); // "Not null"
const baz = 0 ?? "Hello";
console.log(baz); // 0

// https://m.toutiao.com/is/dnGrHRV/?=有个开发者总结这 15 优雅的 JavaScript 个技巧
```

</details>
