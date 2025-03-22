---
title: CSS
tags: css
categories: css
---

# CSS

## 特效

- 故障风格按钮
    ```html
    <button>立即加入</button>
    ```
    ```css
    button, button::after {
        width: 300px;
        height: 86px;
        font-size: 40px;
        background: linear-gradient(45deg, transparent 5%, #FF013C 5%);
        border: 0;
        color: #fff;
        letter-spacing: 3px;
        line-height: 88px;
        box-shadow: 6px 0px 0px #00E6F6;
        outline: transparent;
        position: relative;
    }
    button::after {
        --slice-0: inset(50% 50% 50% 50%);
        --slice-1: inset(80% -6px 0 0);
        --slice-2: inset(50% -6px 30% 0);
        --slice-3: inset(10% -6px 85% 0);
        --slice-4: inset(40% -6px 43% 0);
        --slice-5: inset(80% -6px 5% 0);
        content: '立即加入';
        display: block;
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background: linear-gradient(45deg, transparent 3%, #00E6F6 3%, #00E6F6 5%, #FF013C 5%);
        text-shadow: -3px -3px 0px #F8F005, 3px 3px 0px #00E6F6;
        clip-path: var(--slice-0);
    }
    button:hover::after {
        animation: 1s glitch;
        animation-timing-function: steps(2, end);
    }
    @keyframes glitch {
        0% { clip-path: var(--slice-1); transform: translate(-20px, -10px); }
        10% { clip-path: var(--slice-3); transform: translate(10px, 10px); }
        20% { clip-path: var(--slice-1); transform: translate(-10px, 10px); }
        30% { clip-path: var(--slice-3); transform: translate(0px, 5px); }
        40% { clip-path: var(--slice-2); transform: translate(-5px, 0px); }
        50% { clip-path: var(--slice-3); transform: translate(5px, 0px); }
        60% { clip-path: var(--slice-4); transform: translate(5px, 10px); }
        70% { clip-path: var(--slice-2); transform: translate(-10px, 10px); }
        80% { clip-path: var(--slice-5); transform: translate(20px, -10px); }
        90% { clip-path: var(--slice-1); transform: translate(-10px, 0px); }
        100% { clip-path: var(--slice-1); transform: translate(0); }
    }
    ```

- 金属质感
    ```html
    <button class="gold">gold 金</button>
    <button class="silver">silver 银</button>
    <button class="bronze">bronze 铜</button>
    <button class="titanium">titanium 钛</button>
    ```
    ```css
    button {
        padding: 2px;
        width: 250px;
        height: 250px;
        border-radius: 12px;
        border: groove 1px transparent;
    }
    .gold {
        box-shadow: inset 0 0 0 1px #eedc00, inset 0 1px 2px rgba(255, 255, 255, 0.5), inset 0 -1px 2px rgba(0, 0, 0, 0.5);
        background: conic-gradient(#edc800, #e3b600, #f3cf00, #ffe800, #ffe900, #ffeb00, #ffe000, #ebc500, #e0b100, #f1cc00, #fcdc00, #d4c005fb, #fad900, #eec200, #e7b900, #f7d300, #ffe800, #ffe300, #f5d100, #e6b900, #e3b600, #f4d000, #ffe400, #ebc600, #e3b600, #f6d500, #ffe900, #ffe90a, #edc800) content-box, linear-gradient(#f6d600, #f6d600) padding-box, radial-gradient(rgba(120, 120, 120, 0.9), rgba(120, 120, 120, 0) 70%) 50% bottom/80% 0.46875em no-repeat border-box;
    }
    .silver {
        box-shadow: inset 0 0 0 1px #c9c9c9, inset 0 1px 2px rgba(255, 255, 255, 0.5), inset 0 -1px 2px rgba(0, 0, 0, 0.5);
        background: conic-gradient(#d7d7d7, #c3c3c3, #cccccc, #c6c6c6, #d3d3d3, #d8d8d8, #d5d5d5, #d8d8d8, #d3d3d3, #c5c5c5, #c0c0c0, #bfbfbf, #d0d0d0, #d9d9d9, #d1d1d1, #c5c5c5, #c8c8c8, #d7d7d7, #d5d5d5, #cdcdcd, #c4c4c4, #d9d9d9, #cecece, #c5c5c5, #c5c5c5, #cdcdcd, #d8d8d8, #d9d9d9, #d7d7d7) content-box, linear-gradient(#d4d4d4, #d4d4d4) padding-box, radial-gradient(rgba(120, 120, 120, 0.9), rgba(120, 120, 120, 0) 70%) 50% bottom/80% 0.46875em no-repeat border-box;
    }
    .bronze {
        box-shadow: inset 0 0 0 1px #bc7e6b, inset 0 1px 2px rgba(255, 255, 255, 0.5), inset 0 -1px 2px rgba(0, 0, 0, 0.5);
        background: conic-gradient(#d95641, #b14439, #b2453a, #d25645, #d56847, #d05441, #b85137, #b2453a, #c34f40, #df4647, #a94338, #c94943, #c85442, #a4413c, #d9543a, #d1564e, #ab4338, #bb4a3c, #dc5843, #b94839, #aa4237, #c24e42, #ce523f, #ab4338, #dd5944, #ca4d33, #ab4338, #cb503e, #d95641) content-box, linear-gradient(#ad3b36, #ad3b36) padding-box, radial-gradient(rgba(120, 120, 120, 0.9), rgba(120, 120, 120, 0) 70%) 50% bottom/80% 0.46875em no-repeat border-box;
    }
    .titanium {
        box-shadow: inset 0 0 0 1px #c7aca0, inset 0 1px 2px rgba(255, 255, 255, 0.5), inset 0 -1px 2px rgba(0, 0, 0, 0.5);
        background: conic-gradient(#e6c9bf, #d2b5aa, #cbaea3, #d4b5ab, #e5c3bd, #d9c0b4, #d9bcb1, #c5a399, #e3c6bc, #e7cac0, #dec0b5, #d3b6ab, #cfada1, #d4b6ac, #e2c6c0, #e2c6c0, #d2b1a6, #d2b1a6, #d1b4a9, #e1c4ba, #e5c9be, #dec1b6, #d3b6ab, #ceb0a6, #cfada3, #d2b5aa, #dabdb2, #e5c9be, #e6c9bf) content-box, linear-gradient(#e5c9be, #e5c9be) padding-box, radial-gradient(rgba(120, 120, 120, 0.9), rgba(120, 120, 120, 0) 70%) 50% bottom/80% 0.46875em no-repeat border-box;
    }
    ```

- [故障风格图片](https://codepen.io/dragonir/full/oNZPLzo)
    ```html
    <body>
        <div class="glitch"><!-- 图片展示容器主体 -->
            <div class="glitch__item"></div><!-- 故障条 -->
            <div class="glitch__item"></div>
            <div class="glitch__item"></div>
            <div class="glitch__item"></div>
            <div class="glitch__item"></div>
        </div>
	</body>
    ```
    ```css
    :root {
        --gap-horizontal: 10px;
        --gap-vertical: 5px;
        --time-anim: 4s;
        --delay-anim: 2s;
        --blend-mode-1: none;
        --blend-mode-2: none;
        --blend-mode-3: none;
        --blend-mode-4: none;
        --blend-mode-5: overlay;
        --blend-color-1: transparent;
        --blend-color-2: transparent;
        --blend-color-3: transparent;
        --blend-color-4: transparent;
        --blend-color-5: transparent;
    }
    body {
        background: #FDEC0A;
        margin: 0;
        padding: 0;
    }
    .glitch {
        width: 1000px;
        height: 500px;
        position: relative;
        margin: 40px auto;
    }
    .glitch .glitch__item {
        background: url("https://www.cyberpunk.net/build/images/home-header/cover-1440-zh-cn-edd5d3b7.jpg") no-repeat 50% 50%/cover;
        height: 100%;
        width: 100%;
        top: 0;
        left: 0;
        position: absolute;
    }
    .glitch .glitch__item:nth-child(n+2) {
        opacity: 0;
        animation-duration: var(--time-anim);
        animation-delay: var(--delay-anim);
        animation-timing-function: linear;
        animation-iteration-count: infinite;
    }
    .glitch .glitch__item:nth-child(2) {
        background-color: var(--blend-color-2);
        background-blend-mode: var(--blend-mode-2);
        animation-name: glitch-anim-2;
    }
    .glitch .glitch__item:nth-child(2) {
        background-color: var(--blend-color-3);
        background-blend-mode: var(--blend-mode-3);
        animation-name: glitch-anim-3;
    }
    .glitch .glitch__item:nth-child(4) {
        background-color: var(--blend-color-4);
        background-blend-mode: var(--blend-mode-4);
        animation-name: glitch-anim-4;
    }
    .glitch .glitch__item:nth-child(5) {
        background-color: var(--blend-color-5);
        background-blend-mode: var(--blend-mode-5);
        animation-name: glitch-anim-flash;
    }
    @keyframes glitch-anim-flash {
        0%,
        5% {
            opacity: 0.2;
            transform: translate3d(var(--glitch-horizontal), 500px, 0);
        }
        5.5%,
        100% {
            opacity: 0;
            transform: translate3d(0, 0, 0);
        }
    }
    @keyframes glitch-anim-2 {
        0% {
            opacity: 1;
            transform: translate3d(var(--gap-horizontal), 0, 0);
            -webkit-clip-path: polygon(0 2%, 100% 2%, 100% 5%, 0 5%);
            clip-path: polygon(0 2%, 100% 2%, 100% 5%, 0 5%);
        }
        2% {
            -webkit-clip-path: polygon(0 15%, 100% 15%, 100% 15%, 0 15%);
            clip-path: polygon(0 15%, 100% 15%, 100% 15%, 0 15%);
        }
        4% {
            -webkit-clip-path: polygon(0 10%, 100% 10%, 100% 20%, 0 20%);
            clip-path: polygon(0 10%, 100% 10%, 100% 20%, 0 20%);
        }
        6% {
            -webkit-clip-path: polygon(0 1%, 100% 1%, 100% 2%, 0 2%);
            clip-path: polygon(0 1%, 100% 1%, 100% 2%, 0 2%);
        }
        8% {
            -webkit-clip-path: polygon(0 33%, 100% 33%, 100% 33%, 0 33%);
            clip-path: polygon(0 33%, 100% 33%, 100% 33%, 0 33%);
        }
        10% {
            -webkit-clip-path: polygon(0 44%, 100% 44%, 100% 44%, 0 44%);
            clip-path: polygon(0 44%, 100% 44%, 100% 44%, 0 44%);
        }
        12% {
            -webkit-clip-path: polygon(0 50%, 100% 50%, 100% 20%, 0 20%);
            clip-path: polygon(0 50%, 100% 50%, 100% 20%, 0 20%);
        }
        14% {
            -webkit-clip-path: polygon(0 70%, 100% 70%, 100% 70%, 0 70%);
            clip-path: polygon(0 70%, 100% 70%, 100% 70%, 0 70%);
        }
        16% {
            -webkit-clip-path: polygon(0 80%, 100% 80%, 100% 80%, 0 80%);
            clip-path: polygon(0 80%, 100% 80%, 100% 80%, 0 80%);
        }
        18% {
            -webkit-clip-path: polygon(0 50%, 100% 50%, 100% 55%, 0 55%);
            clip-path: polygon(0 50%, 100% 50%, 100% 55%, 0 55%);
        }
        20% {
            -webkit-clip-path: polygon(0 70%, 100% 70%, 100% 80%, 0 80%);
            clip-path: polygon(0 70%, 100% 70%, 100% 80%, 0 80%);
        }
        21.9% {
            opacity: 1;
            transform: translate3d(var(--gap-horizontal), 0, 0);
        }
        22%,
        100% {
            opacity: 0;
            transform: translate3d(0, 0, 0);
            -webkit-clip-path: polygon(0 0, 0 0, 0 0, 0 0);
            clip-path: polygon(0 0, 0 0, 0 0, 0 0);
        }
    }
    @keyframes glitch-anim-3 {
        0% {
            opacity: 1;
            transform: translate3d(calc(-1 * var(--gap-horizontal)), 0, 0);
            -webkit-clip-path: polygon(0 25%, 100% 25%, 100% 30%, 0 30%);
            clip-path: polygon(0 25%, 100% 25%, 100% 30%, 0 30%);
        }
        3% {
            -webkit-clip-path: polygon(0 3%, 100% 3%, 100% 3%, 0 3%);
            clip-path: polygon(0 3%, 100% 3%, 100% 3%, 0 3%);
        }
        5% {
            -webkit-clip-path: polygon(0 5%, 100% 5%, 100% 20%, 0 20%);
            clip-path: polygon(0 5%, 100% 5%, 100% 20%, 0 20%);
        }
        7% {
            -webkit-clip-path: polygon(0 20%, 100% 20%, 100% 20%, 0 20%);
            clip-path: polygon(0 20%, 100% 20%, 100% 20%, 0 20%);
        }
        9% {
            -webkit-clip-path: polygon(0 40%, 100% 40%, 100% 40%, 0 40%);
            clip-path: polygon(0 40%, 100% 40%, 100% 40%, 0 40%);
        }
        11% {
            -webkit-clip-path: polygon(0 52%, 100% 52%, 100% 59%, 0 59%);
            clip-path: polygon(0 52%, 100% 52%, 100% 59%, 0 59%);
        }
        13% {
            -webkit-clip-path: polygon(0 60%, 100% 60%, 100% 60%, 0 60%);
            clip-path: polygon(0 60%, 100% 60%, 100% 60%, 0 60%);
        }
        15% {
            -webkit-clip-path: polygon(0 75%, 100% 75%, 100% 75%, 0 75%);
            clip-path: polygon(0 75%, 100% 75%, 100% 75%, 0 75%);
        }
        17% {
            -webkit-clip-path: polygon(0 65%, 100% 65%, 100% 40%, 0 40%);
            clip-path: polygon(0 65%, 100% 65%, 100% 40%, 0 40%);
        }
        19% {
            -webkit-clip-path: polygon(0 45%, 100% 45%, 100% 50%, 0 50%);
            clip-path: polygon(0 45%, 100% 45%, 100% 50%, 0 50%);
        }
        20% {
            -webkit-clip-path: polygon(0 14%, 100% 14%, 100% 33%, 0 33%);
            clip-path: polygon(0 14%, 100% 14%, 100% 33%, 0 33%);
        }
        21.9% {
            opacity: 1;
            transform: translate3d(calc(-1 * var(--gap-horizontal)), 0, 0);
        }
        22%,
        100% {
            opacity: 0;
            transform: translate3d(0, 0, 0);
            -webkit-clip-path: polygon(0 0, 0 0, 0 0, 0 0);
            clip-path: polygon(0 0, 0 0, 0 0, 0 0);
        }
    }
    @keyframes glitch-anim-4 {
        0% {
            opacity: 1;
            transform: translate3d(0, calc(-1 * var(--gap-vertical)), 0) scale3d(-1, -1, 1);
            -webkit-clip-path: polygon(0 1%, 100% 1%, 100% 3%, 0 3%);
            clip-path: polygon(0 1%, 100% 1%, 100% 3%, 0 3%);
        }
        1.5% {
            -webkit-clip-path: polygon(0 10%, 100% 10%, 100% 9%, 0 9%);
            clip-path: polygon(0 10%, 100% 10%, 100% 9%, 0 9%);
        }
        2% {
            -webkit-clip-path: polygon(0 5%, 100% 5%, 100% 6%, 0 6%);
            clip-path: polygon(0 5%, 100% 5%, 100% 6%, 0 6%);
        }
        2.5% {
            -webkit-clip-path: polygon(0 20%, 100% 20%, 100% 20%, 0 20%);
            clip-path: polygon(0 20%, 100% 20%, 100% 20%, 0 20%);
        }
        3% {
            -webkit-clip-path: polygon(0 10%, 100% 10%, 100% 10%, 0 10%);
            clip-path: polygon(0 10%, 100% 10%, 100% 10%, 0 10%);
        }
        5% {
            -webkit-clip-path: polygon(0 30%, 100% 30%, 100% 25%, 0 25%);
            clip-path: polygon(0 30%, 100% 30%, 100% 25%, 0 25%);
        }
        5.5% {
            -webkit-clip-path: polygon(0 15%, 100% 15%, 100% 16%, 0 16%);
            clip-path: polygon(0 15%, 100% 15%, 100% 16%, 0 16%);
        }
        7% {
            -webkit-clip-path: polygon(0 40%, 100% 40%, 100% 39%, 0 39%);
            clip-path: polygon(0 40%, 100% 40%, 100% 39%, 0 39%);
        }
        8% {
            -webkit-clip-path: polygon(0 20%, 100% 20%, 100% 21%, 0 21%);
            clip-path: polygon(0 20%, 100% 20%, 100% 21%, 0 21%);
        }
        9% {
            -webkit-clip-path: polygon(0 60%, 100% 60%, 100% 55%, 0 55%);
            clip-path: polygon(0 60%, 100% 60%, 100% 55%, 0 55%);
        }
        10.5% {
            -webkit-clip-path: polygon(0 30%, 100% 30%, 100% 31%, 0 31%);
            clip-path: polygon(0 30%, 100% 30%, 100% 31%, 0 31%);
        }
        11% {
            -webkit-clip-path: polygon(0 70%, 100% 70%, 100% 69%, 0 69%);
            clip-path: polygon(0 70%, 100% 70%, 100% 69%, 0 69%);
        }
        13% {
            -webkit-clip-path: polygon(0 40%, 100% 40%, 100% 41%, 0 41%);
            clip-path: polygon(0 40%, 100% 40%, 100% 41%, 0 41%);
        }
        14% {
            -webkit-clip-path: polygon(0 80%, 100% 80%, 100% 75%, 0 75%);
            clip-path: polygon(0 80%, 100% 80%, 100% 75%, 0 75%);
        }
        14.5% {
            -webkit-clip-path: polygon(0 50%, 100% 50%, 100% 51%, 0 51%);
            clip-path: polygon(0 50%, 100% 50%, 100% 51%, 0 51%);
        }
        15% {
            -webkit-clip-path: polygon(0 90%, 100% 90%, 100% 90%, 0 90%);
            clip-path: polygon(0 90%, 100% 90%, 100% 90%, 0 90%);
        }
        16% {
            -webkit-clip-path: polygon(0 60%, 100% 60%, 100% 60%, 0 60%);
            clip-path: polygon(0 60%, 100% 60%, 100% 60%, 0 60%);
        }
        18% {
            -webkit-clip-path: polygon(0 100%, 100% 100%, 100% 99%, 0 99%);
            clip-path: polygon(0 100%, 100% 100%, 100% 99%, 0 99%);
        }
        20% {
            -webkit-clip-path: polygon(0 70%, 100% 70%, 100% 71%, 0 71%);
            clip-path: polygon(0 70%, 100% 70%, 100% 71%, 0 71%);
        }
        21.9% {
            opacity: 1;
            transform: translate3d(0, calc(-1 * var(--gap-vertical)), 0) scale3d(-1, -1, 1);
        }
        22%,
        100% {
            opacity: 0;
            transform: translate3d(0, 0, 0);
            -webkit-clip-path: polygon(0 0, 0 0, 0 0, 0 0);
            clip-path: polygon(0 0, 0 0, 0 0, 0 0);
        }
    }
    ```

- loading
    ```html
    <div class="showbox">
        <div class="loader" style="width: 47px;"><!-- 设置 loader 大小为 47 x 47 -->
            <svg class="circular" viewBox="25 25 50 50">
                <circle class="path" cx="50" cy="50" r="20" fill="none" stroke-width="2" stroke-miterlimit="10" />
            </svg>
        </div>
    </div>
    ```
    ```css
    .loader {
        position: relative;
        margin: 0 auto;
        width: 100px;
    }
    .loader:before {
        content: '';
        display: block;
        padding-top: 100%;
    }
    .circular {
        animation: rotate 2s linear infinite;
        height: 100%;
        transform-origin: center center;
        width: 100%;
        position: absolute;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;
        margin: auto;
    }
    .path {
        stroke-dasharray: 1, 200;
        stroke-dashoffset: 0;
        animation: dash 1.5s ease-in-out infinite, color 6s ease-in-out infinite;
        stroke-linecap: round;
    }
    @keyframes rotate {
        100% {
            transform: rotate(360deg);
        }
    }
    @keyframes dash {
        0% {
            stroke-dasharray: 1, 200;
            stroke-dashoffset: 0;
        }
        50% {
            stroke-dasharray: 89, 200;
            stroke-dashoffset: -35px;
        }
        100% {
            stroke-dasharray: 89, 200;
            stroke-dashoffset: -124px;
        }
    }
    @keyframes color {
        100%,
        0% {
            stroke: #d62d20;
        }
        40% {
            stroke: #0057e7;
        }
        66% {
            stroke: #008744;
        }
        80%,
        90% {
            stroke: #ffa700;
        }
    }
    body {
        background-color: #eee;
    }
    .showbox {
        position: absolute;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;
        padding: 5%;
    }
    ```

- 按钮的冷光效果
    ```html
    <div>
        <button>Button</button>
        <button>Button</button>
    </div>
    ```
    ```css
    * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }
    html, body {
        display: grid;
        height: 100%;
        place-items: center;
        background: #000;
        overflow: hidden;
    }
    button {
        position: relative;
        height: 60px;
        width: 200px;
        margin: 0 35px;
        border-radius: 50px;
        border: none;
        outline: none;
        background: #111;
        color: #fff;
        font-size: 20px;
        letter-spacing: 2px;
        text-transform: uppercase;
        cursor: pointer;
    }
    button:first-child:hover {
        background: linear-gradient(90deg, #03a9f4, #f441a5, #ffeb3b, #03a9f4);
        background-size: 400%;
    }
    button:last-child:hover {
        background: linear-gradient(90deg, #fa7199, #f5ce62, #e43603, #fa7199);
        background-size: 400%;
    }
    button:first-child:before,
    button:last-child:before {
        content: '';
        position: absolute;
        background: inherit;
        top: -5px;
        right: -5px;
        bottom: -5px;
        left: -5px;
        border-radius: 50px;
        filter: blur(20px);
        opacity: 0;
        transform: opacity 0.5s;
    }
    button:first-child:hover:before,
    button:last-child:hover:before {
        opacity: 1;
        z-index: -1;
    }
    button:hover {
        z-index: 1;
        animation: glow 8s linear infinite;
    }
    @keyframes glow{
        0% {
            background-position: 0%;
        }
        100% {
            background-position: 400%;
        }
    }
    ```

- [“前后”图像比较功能](https://coding.zhangbing.site/view.html?url=./list/slider-before-after-image.html)
    ```html
    <div class='container'>
        <div class='img background-img'></div>
        <div class='img foreground-img'></div>
        <input type="range" min="1" max="100" value="50" class="slider" name='slider' id="slider">
        <div class='slider-button'></div>
    </div>  
    ```
    ```css
    .container {
        position: relative;
        width: 900px;
        height: 600px;
        border: 2px solid white;
    }
    .img {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-size: 900px 100%;
    }
    .background-img {
        background-image: url('https://i.loli.net/2020/12/28/1dGpFx3zJ9Pjcme.jpg');
    }
    .foreground-img {
        background-image: url('https://i.loli.net/2020/12/28/xIZmjtBR5VWoqiz.jpg');
        width: 50%;
    }
    .slider {
        display: flex;
        justify-content: center;
        align-items: center;
        position: absolute;
        -webkit-appearance: none;
        appearance: none;
        width: 100%;
        height: 100%;
        background: rgba(242,242,241,.3);
        outline: none;
        margin: 0;
        transition: all .2s;
    }
    .slider:hover {
        background: rgba(242,242,241,.1); 
    }
    .slider::-webkit-slider-thumb {
        -webkit-appearance: none;
        appearance: none;
        width: 6px;
        height: 600px;
        background: white;
    }
    .slider-button {
        pointer-events: none;
        position: absolute;
        width: 30px;
        height: 30px;
        border-radius: 50%;
        background-color: white;
        left: calc(50% - 18px);
        top: calc(50% - 18px);
        display: flex;
        justify-content: center;
        align-items: center;
    }
    .slider-button::after {
        content: '';
        padding: 3px;
        display: inline-block;
        border: solid #5D5D5D;
        border-width: 0 2px 2px 0;
        transform: rotate(-45deg);
    }
    .slider-button::before {
        content: '';
        padding: 3px;
        display: inline-block;
        border: solid #5D5D5D;
        border-width: 0 2px 2px 0;
        transform: rotate(135deg);
    }
    ```
    ```javascript
    document.getElementById("slider").addEventListener("input", function(e) {
        let sliderPos = e.target.value;
        document.querySelector(".foreground-img").style.width = `${sliderPos}%`
        document.querySelector(".slider-button").style.left = `calc(${sliderPos}% - 18px)`
    });
    ```

- div 撑满页面
    ```html
    <!DOCTYPE html>
    <html>
        <head>
            <meta charset="UTF-8">
            <style>
                * {
                    margin: 0;
                    padding: 0;
                }
                html, body {
                    width: 100%;
                    height: 100%;
                }
                div {
                    width: 100%;
                    height: 100%;
                    background: red;
                }
            </style>
        </head>
        <body>
            <div></div>
        </body>
    </html>
    ```

