---
title: Algorithm
tags: algorithm
categories: algorithm
mathjax: true
---

# Algorithm

> - [时间复杂度](https://blog.csdn.net/u011614717/article/details/82624095)
>     - O(1) 表示一次操作即可直接取得目标元素（比如字典或哈希表）
>     - O(n) 意味着先要检查 n 个元素来搜索目标
>     - 二分搜索最好情况下的时间复杂度是 O(1)，最坏情况（平均情况）下 O(log(n))

- [运算符](http://www.cnblogs.com/shangyu/archive/2012/07/28/2612832.html)
    - 按位与 "&"
        - 清零指定位  
            将十进制整数 "257"（转换成二进制为 "00000001 00000001"）的高八位清零，结果应为十进制整数 "1"（转换成二进制为 "00000000 00000001"  
            `console.assert((257 & 0xff) == 1);`
        - 取某数中指定位  
            判断十进制整数 "225"（转换成二进制为 "11100001"）的第七位是否为 "1"  
            `console.assert((225 & 0x40) == 0x40);`
    - 按位或 "|"
        - 将指定位置 "1"  
            将十进制整数 "225"（转换成二进制为 "11100001"）的第四位置 "1"，结果应为十进制整数 "233"（转换成二进制为 "11101001"  
            `console.assert((225 | 0x08) == 0xE9);`
    - 按位取反 "~"

    - 按位异或 "^"
        - 将指定位的值取反  
            将十进制整数 "225"（转换成二进制为 "11100001"）的第七位取反，结果应为十进制整数 "161"（转换成二进制为 "10100001"  
            `console.assert((225 ^ 0x40) == 0xA1);`
        - 不引入第三个变量，交换两个变量的值  
            ```javascript
            var arg1 = 12, arg2 = 35;
            arg1 = arg1 ^ arg2;
            arg2 = arg1 ^ arg2;
            arg1 = arg1 ^ arg2;
            console.assert(arg1 == 35);
            console.assert(arg2 == 12);
            ```
    - 左移运算符 "<<"
        - 把运算符左边的运算数的各二进位全部左移 n 位，实现运算数乘以 2 的 n 次方  
            将十进制整数 "225"（转换成二进制为 "00000000 11100001"）左移两位，结果应为十进制整数 "900"（转换成二进制为 "00000011 10000100"）  
            `console.assert((225 << 2) == 0x0384);`
    - 右移运算符 ">>"
        - 把运算符左边的运算数的各二进位全部右移 n 位，实现运算数除以 2 的 n 次方  
            将十进制整数 "225"（转换成二进制为 "11100001"）右移两位，结果应为十进制整数 "56"（转换成二进制为 "00111000"）  
            `console.assert((225 >> 2) == 0x38);`
    - 无符号右移 ">>>"
        - 把运算符左边的运算数的各二进位全部右移 n 位，忽略符号位，空位都以 0 补齐
    - 其他
        ```javascript
        var x, y;
        
        // 判断奇偶数
        x & 1 == 1
        
        // 交换两个数
        x = x ^ y; y = x ^ y; x = x ^ y;
        
        // 两个相同的数异或的结果是 0，一个数和 0 异或的结果是它本身
        
        // 将最右边的 1 设置为 0
        x & (x - 1)
        
        // x 取反再与 x 相加，相当于把所有二进制位设为 1，其十进制结果为 -1
        x + (~x) == -1
        
        // 对于 int32 而言，使用 n >> 31 取得 n 的正负号。并且可以通过 (n ^ (n >> 31)) - (n >> 31) 来得到绝对值。（n 为正，n >> 31 的所有位等于 0。若 n 为负数，n >> 31 的所有位等于 1，其值等于 -1）
        
        // 使用 (x ^ y) >= 0 来判断符号是否相同。（如果两个数都是正数，则二进制的第一位均为 0，x ^ y = 0；如果两个数都是负数，则二进制的第一位均为 1；x ^ y = 0 如果两个数符号相反，则二进制的第一位相反，x ^ y = 1。有 0 的情况例外，^ 相同得 0，不同得 1）
        
        // 异或是一个无进位加法，说白了就是把进位砍掉。比如 01 ^ 01 = 00
        
        // 与可以用来获取进位，比如 01 & 01 = 01，然后再把结果左移一位，就可以获取进位结果
        ```

- [求平方根的倒数（出自《雷神之锤3》源码）](http://www.cnblogs.com/tntboom/p/4096036.html)
    ```c
    float Q_rsqrt( float number )
    {
        long i;
        float x2, y;
        const float threehalfs = 1.5F;
    
        x2 = number * 0.5F;
        y  = number;
        i  = * ( long * ) &y;   // 将浮点数转化为整数
        i  = 0x5f3759df - ( i >> 1 );
        y  = * ( float * ) &i;  // 将 i 表示成浮点数
        y  = y * ( threehalfs - ( x2 * y * y ) );   // 一次牛顿迭代
        // y  = y * ( threehalfs - ( x2 * y * y ) );   // 二次牛顿迭代，可以提升结果的精度

        return y;
    }
    ```

- 斐波拉契数列
    - 动态规划
        ```javascript
        (function (n) {
            let dp = new Array(n < 2 ? 2 : n);
            dp[0] = 1;
            dp[1] = 1;

            for (let i = 2; i < n; i++) {
                dp[i] = dp[i - 1] + dp[i - 2];
            }

            return dp[n - 1];
        })(50);
        ```
    - [矩阵快速幂](https://www.cnblogs.com/iwiniwin/p/10798884.html)
        ```javascript
        (function (n) {
            /**
            * 2x2 矩阵乘法
            * @param m 
            * @param n 
            */
            const matrix_dot = function (m, n) {
                return [
                    [m[0][0] * n[0][0] + m[0][1] * n[1][0], m[0][0] * n[0][1] + m[0][1] * n[1][1]],
                    [m[1][0] * n[0][0] + m[1][1] * n[1][0], m[1][0] * n[0][1] + m[1][1] * n[1][1]]
                ];
            };
            /**
            * 矩阵快速幂
            * @param m 
            * @param n 
            */
            const matrix_pow = function (m, n) {
                let ret = [
                    [1, 0],
                    [0, 1]
                ]; // 单位矩阵，作用相当于整数乘法中的 1
                while (n > 0) {
                    if ((n & 1) > 0) {
                        ret = matrix_dot(m, ret);
                    }
                    m = matrix_dot(m, m);
                    n >>= 1;
                }
                return ret;
            };

            let matrix = [
                [1, 1],
                [1, 0]
            ];

            // 这里的 F1 和 F0 矩阵多加了一列 0,0，不会影响最终结果，是因为 matrix_mul 只实现了 2*2 矩阵的乘法
            let unit = [
                [1, 0],
                [0, 0]
            ];
            let ret = matrix_dot(matrix_pow(matrix, n - 1), unit);
            return ret[0][0];
        })(50);
        ```
        $$ \begin{bmatrix} f(n) \\\\ f(n - 1) \end{bmatrix} = \begin{bmatrix} f(n - 1) + f(n - 2) \\\\ f(n - 1) \end{bmatrix} = \begin{bmatrix} f(n - 1) * 1 + f(n - 2) * 1 \\\\ f(n - 1) * 1 + f(n - 2) * 0 \end{bmatrix} = \begin{bmatrix} 1 & 1 \\\\ 1 & 0 \end{bmatrix} \begin{bmatrix} f(n - 1) \\\\ f(n - 2) \end{bmatrix} = \begin{bmatrix} 1 & 1 \\\\ 1 & 0 \end{bmatrix} ^{n - 1} \begin{bmatrix} f(1) \\\\ f(0) \end{bmatrix} $$

- 矩阵运算
    - 点乘
        ```javascript
        const matrix_dot = function (m, n) {
            console.assert(m instanceof Array && m[0] instanceof Array 
                && n instanceof Array && n[0] instanceof Array
                && m.length == n[0].length && m[0].length == n.length);

            let ret = new Array(m.length);
            for (let i = 0; i < m.length; i++) {
                let j = n[0].length;
                ret[i] = new Array(j);
                while (j--) {
                    ret[i][j] = 0;
                }
            }

            for (let x = 0; x < m.length; x++) {
                for (let y = 0; y < n[0].length; y++) {
                    for (let k = 0; k < m[0].length; k++) {
                        ret[x][y] += m[x][k] * n[k][y];
                    }
                }
            }

            return ret;
        };

        console.info(matrix_dot([
            [1, 2, 3],
            [4, 5, 6]
        ], [
            [1, 2],
            [3, 4],
            [5, 6]
        ])); // [[22, 28], [49, 64]]
        ```

- 排列组合
    ```javascript
    /**
     * 排列
     */
    const permutation = function (arr, str = "", res = []) {
        if (str.length === arr.length) {
            res.push(str);
        } else {
            for (let i = 0; i < arr.length; i++) {
                if (str.indexOf(arr[i]) === -1) {
                    permutation(arr, str + arr[i], res);
                }
            }
        }
        return res;
    };

    /**
     * 组合
     */
    const combination = function (arr) {
        let res = [];
        let count = 1 << arr.length; // 全组合的个数为 2^n
        for (let i = 0; i < count; i++) {
            let str = "";
            for (let j = 0; j < arr.length; j++) {
                if (((1 << j) & i) != 0) {
                    str += arr[j];
                }
            }
            res.push(str);
        }
        return res;
    };

    let arr = "abc".split("");
    console.info("permutation of", arr, "is", permutation(arr)); // ["abc", "acb", "bac", "bca", "cab", "cba"]
    console.info("combination of", arr, "is", combination(arr)); // ["", "a", "b", "ab", "c", "ac", "bc", "abc"]
    ```

- 进制转换
    ```javascript
    const CHARSET = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ".split("");
    const str = function (n) {
        let radix = CHARSET.length,
            qutient = +n,
            arr = [];
        do {
            mod = qutient % radix;
            qutient = (qutient - mod) / radix;
            arr.unshift(CHARSET[mod]);
        } while (qutient);
        return arr.join("");
    }
    const int = function (s) {
        s = String(s);
        let radix = CHARSET.length,
            len = s.length,
            i = 0,
            n = 0;
        while (i < len) {
            n += Math.pow(radix, i++) * CHARSET.indexOf(s.charAt(len - i) || 0);
        }
        return n;
    }
    console.info(str(9999999)); // "FXsj"
    console.info(int("FXsj")); // 9999999
    ```

- 最大公约数
    ```javascript
    function gcd(a, b) { 
        if (b == 0) 
            return a; 
        return gcd(b, a % b); 
    }
    ```

- [快速乘](https://www.cnblogs.com/812-xiao-wen/p/10543023.html)
    ```cpp
    inline ll ksc(ll x, ll y, ll p) {
        ll z = (ld)x / p * y;
        ll res = (ull)x * y - (ull)z * p;
        return (res + p) % p;
    }
    // ll 表示 long long
    // ld 表示 long double
    // ull 表示 unsigned long long
    // 一种自动溢出的数据类型（存满了就会自动变为 0）
    ```

- B+ 树

- 贪心算法

- 卷积神经网络

- 迪杰斯特拉

- 盲水印

- 布隆过滤器
    ```typescript
    /**
     * 字节，8 位、有符号的，以二进制补码表示的整数
     */
    export declare type byte = number;

    /**
     * 布隆过滤器
     *
     * @see https://developer.aliyun.com/article/3607
     */
    export class BloomFilter {
        /**
         * 布隆过滤器位图映射变量
         */
        private bitMap: byte[];
    
        /**
         * 位图变量的长度
         */
        private bitSize: number;
    
        /**
         * 哈希数量
         */
        private hashCount: number;

        /**
         * 已加入元素数量
         */
        private keyCount: number;

        /**
         * Bloom Filter
         *
         * @param maxKeys 最多可放的数量
         * @param errorRate 错误率
         */
        constructor(maxKeys, errorRate) {
            this.bitMap = [];
            this.bitSize = Math.ceil(maxKeys * (-Math.log(errorRate) / (Math.log(2) * Math.log(2)))); // 需要根据 maxKeys 和 errorRate 来计算
            this.hashCount = Math.ceil(Math.log(2) * (this.bitSize / maxKeys));
            this.keyCount = 0;
        }

        /**
         * 新增
         *
         * @param key
         */
        public add(key) {
            if (this.contain(key)) {
                return -1;
            }

            let hash1 = this.murmurhash(key, 0, 0),
                hash2 = this.murmurhash(key, 0, hash1);

            for (let i = 0; i < this.hashCount; i++) {
                // 设置位
                let bit = Math.abs(Math.floor((hash1 + i * hash2) % (this.bitSize)));
                let numArr = Math.floor(bit / 31),
                numBit = Math.floor(bit % 31);
                this.bitMap[numArr] |= (1 << numBit);
            }

            this.keyCount++;
        }

        /**
         * 判断是否存在
         *
         * @param key
         */
        public contain(key): boolean {
            let hash1 = this.murmurhash(key, 0, 0),
                hash2 = this.murmurhash(key, 0, hash1);

            for (let i = 0; i < this.hashCount; i++) {
                // 读取位
                let bit = Math.abs(Math.floor((hash1 + i * hash2) % (this.bitSize)));
                let numArr = Math.floor(bit / 31),
                numBit = Math.floor(bit % 31);
                if (!(this.bitMap[numArr] &= (1 << numBit))) {
                    return false;
                }
            }

            return true;
        }

        /**
         * Murmur Hash
         * 
         * @param data 待哈希的值
         * @param offset
         * @param seed 种子集
         * @see http://murmurhash.googlepages.com/
         */
        private murmurhash(data: string, offset: number = 0, seed: number = 0): number {
            let len = data.length,
                m = 0x5bd1e995,
                r = 24,
                h = seed ^ len,
                len_4 = len >> 2;

            for (let i = 0; i < len_4; i++) {
                let i_4 = (i << 2) + offset,
                    k = data.charCodeAt(i_4 + 3);

                k = k << 8;
                k = k | (data.charCodeAt(i_4 + 2) & 0xff);
                k = k << 8;
                k = k | (data.charCodeAt(i_4 + 1) & 0xff);
                k = k << 8;
                k = k | (data.charCodeAt(i_4 + 0) & 0xff);
                k *= m;
                k ^= k >>> r;
                k *= m;
                h *= m;
                h ^= k;
            }

            // avoid calculating modulo  
            let len_m = len_4 << 2,
                left = len - len_m,
                i_m = len_m + offset;

            if (left != 0) {
                if (left >= 3) {
                    h ^= data.charCodeAt(i_m + 2) << 16;
                }
                if (left >= 2) {
                    h ^= data.charCodeAt(i_m + 1) << 8;
                }
                if (left >= 1) {
                    h ^= data.charCodeAt(i_m);
                }

                h *= m;
            }

            h ^= h >>> 13;
            h *= m;
            h ^= h >>> 15;

            return h;
        }
    }

    (() => {
        let bf = new BloomFilter(100000000, 0.01);
        bf.add("hello, world");
        console.assert(bf.contain("hello, world"));
    })();
    ```

- 位图索引/Bitmap  
    有 40 亿个不重复的无序的无符号整数，现给出一个无符号整数，如何快速判断这个数是否在这 40 亿个数中
    ```
    class Bitmap {
        bytes = null;

        constructor(length) {
            this.bytes = new Uint8Array((length >>> 3) + 1);
        }

        add(num) {
            this.bytes[num >>> 3] |= 1 << num % 0x7;
        }

        exist(num) {
            return (this.bytes[num >>> 3] & 1 << num % 0x7) != 0;
        }

        remove(num) {
            this.bytes[num >>> 3] &= ~(1 << num % 0x7);
        }
    }
    
    let bitmap = new Bitmap(100);
    bitmap.add(99);
    bitmap.exist(99); // true
    bitmap.remove(99);
    bitmap.exist(99); // false
    ```

- 字符串匹配算法
    - 编辑距离（动态规划）
        ```javascript
        (function (a, b) {
            let m = a.length,
                n = b.length;
            let dp = new Array(m + 1).fill("").map(d => new Array(n + 1).fill(0));
            for (let i = 1; i < m + 1; i++) {
                dp[i][0] = i;
            }
            for (let j = 1; j < n + 1; j++) {
                dp[0][j] = j;
            }
            for (let i = 1; i < m + 1; i++) {
                for (let j = 1; j < n + 1; j++) {
                    if (a.charAt(i - 1) == b.charAt(j - 1)) {
                        dp[i][j] = dp[i - 1][j - 1];
                    } else {
                        dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1]) + 1;
                    }
                }
            }
            return dp[m][n];
        })("hello", "hollow"); // 2
        ```
    - 余弦相似度
        ```javascript
        const CosineSimilarity = {
            token: function (text) { // 分词
                return text.split(/[^A-Za-z0-9]+/).map(word => {
                    return word.toLowerCase();
                });
            },
            frequence: function (words, dict) { // 计算词频
                return dict.map(word => {
                    return words.reduce((f, element) => {
                        if (element == word)
                            return f += 1;
                        else 
                            return f += 0;
                    }, 0);
                });
            },
            dot: function (a, b) { // 计算向量点乘
                return a.reduce((sum, f, index) => {
                    return sum += (f * b[index]);
                }, 0);
            },
            norm: function (vector) { // 计算向量的模
                return Math.sqrt(vector.reduce((sum, f) => {
                    return sum += (f * f);
                }, 0));
            },
            compare: function (a, b) {
                let aw = this.token(a),
                    bw = this.token(b);
            
                let dict = aw.concat(bw.filter(v => !aw.includes(v))); // 取 a, b 词的并集
            
                let av = this.frequence(aw, dict),
                    bv = this.frequence(bw, dict);
            
                return this.dot(av, bv) / (this.norm(av) * this.norm(bv));
            }
        };

        CosineSimilarity.compare("Julie loves me more than Linda loves me", "Jane likes me more than Julie loves me"); // 0.9302605094190635
        ```
    - Simhash

- 排序算法
    - [快速排序](https://www.geeksforgeeks.org/quick-sort/)
        ```javascript
        (function (arr) {
            const swap = function (nums, a, b) {
                const temp = nums[a];
                nums[a] = nums[b];
                nums[b] = temp;
            };
            const partition = function (arr, low, high) {
                let pivot = arr[high], // pivot (Element to be placed at right position)
                    i = low - 1; // Index of smaller element and indicates the right position of pivot found so far

                for (j = low; j <= high- 1; j++) {
                    // If current element is smaller than the pivot
                    if (arr[j] < pivot) {
                        i++; // increment index of smaller element
                        swap(arr, i, j);
                    }
                }
                swap(arr, i + 1, high);
                return i + 1;
            };
            const sort = function (arr, low, high) {
                if (low < high) {
                    let pi = partition(arr, low, high); // pi is partitioning index, arr[pi] is now at right place
                    sort(arr, low, pi - 1);  // before pi
                    sort(arr, pi + 1, high); // after pi
                }
                return arr;
            };
            return sort(arr, 0, arr.length - 1);
        })([10, 80, 30, 90, 40, 50, 70]);
        ```
        > 时间复杂度：最好 O(nlog(n))，最差 O(n^2)，平均 O(nlog(n))
        - 双基准快速排序

- 图片匹配算法
    - 感知哈希算法

- GeoHash
    ```javascript
    class GeoHash {
        bits = [16, 8, 4, 2, 1];

        base32 = "0123456789bcdefghjkmnpqrstuvwxyz";

        neighbors = {
            right: { even: "bc01fg45238967deuvhjyznpkmstqrwx" },
            left: { even: "238967debc01fg45kmstqrwxuvhjyznp" },
            top: { even: "p0r21436x8zb9dcf5h7kjnmqesgutwvy" },
            bottom: { even: "14365h7k9dcfesgujnmqp0r2twvyx8zb" }
        };

        borders = {
            right: { even: "bcfguvyz" },
            left: { even: "0145hjnp" },
            top: { even: "prxz" },
            bottom: { even: "028b" }
        };

        constructor() {
            this.neighbors.bottom.odd = this.neighbors.left.even;
            this.neighbors.top.odd = this.neighbors.right.even;
            this.neighbors.left.odd = this.neighbors.bottom.even;
            this.neighbors.right.odd = this.neighbors.top.even;
            this.borders.bottom.odd = this.borders.left.even;
            this.borders.top.odd = this.borders.right.even;
            this.borders.left.odd = this.borders.bottom.even;
            this.borders.right.odd = this.borders.top.even;
        }

        /**
         * 计算相邻
         * 
         * @param hash 
         * @param dir 
         * @returns 
         */
        adjacent(hash, dir) {
            hash = hash.toLowerCase();
            var lastchr = hash.charAt(hash.length - 1);
            var type = (hash.length % 2) ? "odd" : "even";
            var base = hash.substring(0, hash.length - 1);
            if (this.borders[dir][type].indexOf(lastchr) != -1)
                base = adjacent(base, dir);
            return base + this.base32[this.neighbors[dir][type].indexOf(lastchr)];
        }

        /**
         * GeoHash 编码
         * 
         * @param latitude 
         * @param longitude 
         * @returns 
         */
        encode(latitude, longitude) {
            var is_even = 1, // 是否是偶数
                lat = [-90.0, 90.0], lon = [-180.0, 180.0],
                bit = 0,
                ch = 0,
                precision = 12, // 精度，即 GeoHash 长度。这里为 12，误差为 ±0.0000186km
                hash = "",
                mid = 0;

            while (hash.length < precision) {
                if (is_even) {
                    mid = (lon[0] + lon[1]) / 2;
                    if (longitude > mid) {
                        ch |= this.bits[bit];
                        lon[0] = mid;
                    } else
                        lon[1] = mid;
                } else {
                    mid = (lat[0] + lat[1]) / 2;
                    if (latitude > mid) {
                        ch |= this.bits[bit];
                        lat[0] = mid;
                    } else
                        lat[1] = mid;
                }

                is_even = !is_even;
                if (bit < 4)
                    bit++;
                else {
                    hash += this.base32[ch];
                    bit = 0;
                    ch = 0;
                }
            }
            return hash;
        }

        /**
         * GeoHash 解码
         * 
         * @param hash 
         * @returns 
         */
        decode(hash) {
            var is_even = 1,
                lat = [-90.0, 90.0], lon = [-180.0, 180.0],
                lat_err = 90.0, lon_err = 180.0;

            const refine_interval = function (interval, cd, mask) {
                if (cd & mask)
                    interval[0] = (interval[0] + interval[1]) / 2;
                else
                    interval[1] = (interval[0] + interval[1]) / 2;
            };

            for (let i = 0; i < hash.length; i++) {
                let c = hash[i],
                    cd = this.base32.indexOf(c);
                for (let j = 0; j < 5; j++) {
                    let mask = this.bits[j];
                    if (is_even) {
                        lon_err /= 2;
                        refine_interval(lon, cd, mask);
                    } else {
                        lat_err /= 2;
                        refine_interval(lat, cd, mask);
                    }
                    is_even = !is_even;
                }
            }
            lat[2] = (lat[0] + lat[1]) / 2;
            lon[2] = (lon[0] + lon[1]) / 2;

            return { latitude: lat, longitude: lon };
        }
    }

    var hasher = new GeoHash();
    hasher.encode(30.589297, 114.293488); // wt3q0b9gx638
    // hasher.adjacent("wt3q0b9gx638", "left"); // wt3q0b9gx632
    ```

- 随机生成密码
    ```javascript
    function randpwd(len) {
	    // let dic = ["ABCDEFGHIJKLMNOPQRSTUVWXYZ", "abcdefghijklmnopqrstuvwxyz", "0123456789", "~!@#$%^&*()-+"],
	    let dic = ["ABCDEFGHJKLMNPQRSTUVWXYZ", "abcdefghijkmnpqrstuvwxyz", "23456789", "~!@#$%^&*()-+"], // 去除容易混淆的字符：大写字母 IO、小写字母 lo、数字 01
	        dics = dic.join("");

        let arr = dic.map(d => d.charAt(Math.random() * d.length));
        arr.sort((a, b) => 0.5 - Math.random());

        for (let i = dic.length; i < len; i++) {
        	arr.push(dics.charAt(Math.random() * dics.length));
        }

        return arr.join("").substr(0, len);
    }

    console.info(randpwd(8));
    ```

- [给定二维空间中四点的坐标，返回四点是否可以构造一个正方形](https://leetcode-cn.com/problems/valid-square/)
    ```javascript
    (function (p1, p2, p3, p4) {
        // 从一个点出发，到达另外三个点的的三个向量
        let side12 = [p2[0] - p1[0], p2[1] - p1[1]],
            side13 = [p3[0] - p1[0], p3[1] - p1[1]],
            side14 = [p4[0] - p1[0], p4[1] - p1[1]];

        // 三个向量的两两点积（内积）
        let dotProduct1213 = (side12[0] * side13[0]) + (side12[1] * side13[1]),
            dotProduct1214 = (side12[0] * side14[0]) + (side12[1] * side14[1]),
            dotProduct1314 = (side13[0] * side14[0]) + (side13[1] * side14[1]);

        // 1. 判断从一个点出发的三个向量的两两相乘有一个点积为零，其余两个则不为零（内积为 0 的两个向量垂直）
        if (((dotProduct1213 === 0) + (dotProduct1214 === 0) + (dotProduct1314 === 0)) !== 1) {
            return false;
        }

        // 三个向量的两两叉积的绝对值
        let corssProduct1213 = Math.abs((side12[0] * side13[1]) - (side13[0] * side12[1])),
            corssProduct1214 = Math.abs((side12[0] * side14[1]) - (side14[0] * side12[1])),
            corssProduct1314 = Math.abs((side13[0] * side14[1]) - (side14[0] * side13[1]));

        // 2. 判断三个叉积的绝对值是否两两相等
        if (corssProduct1213 != corssProduct1214 || corssProduct1213 != corssProduct1314 || corssProduct1214 != corssProduct1314) {
            return false;
        }

        // 3. 叉积绝对值乘以 2 是否等于点积和
        if ((dotProduct1213 + dotProduct1214 + dotProduct1314) != (corssProduct1213 << 1)) {
            return false;
        }

        return true;
    })([0, 0], [1, 2], [3, 1], [2, -1]);
    ```

- 抽象语法树
    ```typescript
    // 参考在线解析 https://astexplorer.net/

    type Token = {
        type: "name" | "comma" | "number" | "paren" | "string";
        value: string;
    }

    class AbstractSyntaxTree {
        public tokenizer(input: string): Token[] {
            const REG_NUMBER = /[0-9]/, REG_LETTER_NUMBER = /[`a-z0-9_\.]/i;

            let current = 0,
                tokens: Token[] = [];

            while (current < input.length) {
                let char = input[current];

                if (char === "(" || char === ")") {
                    tokens.push({ type: "paren", value: char });
                    current++;
                    continue;
                }

                if (char === ",") {
                    tokens.push({ type: "comma", value: char });
                    current++;
                    continue;
                }

                if (REG_NUMBER.test(char)) {
                    let value = "";
                    while (char && REG_NUMBER.test(char)) {
                        value += char;
                        char = input[++current];
                    }
                    tokens.push({ type: "number", value });
                    continue;
                }

                if (REG_LETTER_NUMBER.test(char)) {
                    let value = "";
                    while (char && REG_LETTER_NUMBER.test(char)) {
                        value += char;
                        char = input[++current];
                    }
                    tokens.push({ type: "name", value: value.replace(/`/g, "") });
                    continue;
                }

                if (char === "'" || char === "\"") {
                    let value = "";
                    let s = char;
                    char = input[++current];
                    while (char !== s) {
                        value += char;
                        char = input[++current];
                        if (char === s && input[current - 1] === "\\") {
                            value += char;
                            char = input[++current];
                        }
                    }
                    char = input[++current];
                    tokens.push({ type: "string", value });
                    continue;
                }
                current++;
            }
            return tokens;
        }

        public parser(tokens: Token[]): any {
            let current = 0,
                statement: any = {};

            while (current < tokens.length) {
                let token = tokens[current];

                if (token.type === "name") {
                    if (token.value.toUpperCase() === "SELECT") {
                        statement = { variant: "select", result: [] };
                    } else if (token.value.toUpperCase() === "FROM") {
                        statement.from = {};
                    } else if (typeof token.type === "string" || typeof token.type === "number") {
                        if (statement.from) {
                            statement.from = {
                                type: "identifier",
                                variant: "table",
                                name: token.value
                            };
                        } else if (statement.result) {
                            statement.result.push({
                                type: "identifier",
                                variant: "column",
                                name: token.value
                            });
                        }
                    }
                }
                current++;
            }

            return statement;
        }

        public compile(statement: any): string {
            if (statement.variant === "select") {
                return ["select", statement.result.map(this.compile).join(", "), "from", this.compile(statement.from)].join(" ");
            }
            if (statement.type === "identifier") {
                return statement.name + (statement.alias ? ` as ${statement.alias}` : "");
            }
            throw new Error("Unknown");
        }
    }

    let tree = new AbstractSyntaxTree();
    let tokens = tree.tokenizer(
        "select id, name from user"
    );
    let ast = tree.parser(tokens);
    ast.result.forEach((currentValue: any, index: number, arr: any[]) => {
        currentValue.alias = currentValue.name;
        currentValue.name = "data0.field" + index;
    });
    ast.from.name = "data0";
    let sql = tree.compile(ast);
    console.log(sql);
    ```

- BPMN 工作流引擎
    ```typescript
    import { DB } from "https://deno.land/x/sqlite@v3.3.0/mod.ts";
    import { xml2js } from "https://deno.land/x/xml2js/mod.ts";
    
    class BusinessProcessModelingNotation {
        private db: DB;
    
        private id: string;
    
        private name: string;
    
        private documentation: string;
    
        private activities: { [id: string]: { id: string; name: string; type: string } & any; };
    
        private startends: string[] = ["", ""];
    
        private sequenceflows: { id: string; sourceRef: string; targetRef: string; conditionExpression?: { value: string; }; }[];
    
        constructor(content: string) {
            // 解析 bpmn xml 文件
            let process = <any>(<any>xml2js(content.replace(/activiti:/g, ""), { compact: true })).definitions.process;
    
            this.id = process._attributes.id;
            this.name = process._attributes.namel
            this.documentation = process.documentation._text;
    
            this.activities = (<any[]>[]).concat.apply([],
                Object.keys(process).filter(type => [
                    "startEvent", "endEvent",
                    "parallelGateway", "exclusiveGateway", "inclusiveGateway", "eventGateway",
                    "userTask", "scriptTask"
                ].indexOf(type) !== -1).map(type => {
                    if (type === "startEvent") {
                        this.startends[0] = process[type]._attributes.id;
                    }
                    if (type === "endEvent") {
                        this.startends[1] = process[type]._attributes.id;
                    }
                    return (process[type] instanceof Array ? <any[]>process[type] : [<any>process[type]]).map(e => {
                        return {
                            ...e._attributes,
                            type
                        };
                    });
                })
            ).reduce((p, c) => { p[c.id] = c; return p; }, {});
    
            this.sequenceflows = (<any[]>process.sequenceFlow).map(({ _attributes, conditionExpression }) => {
                return {
                    ..._attributes,
                    ...(conditionExpression && {
                        conditionExpression: {
                            value: conditionExpression._cdata
                        }
                    })
                };
            });
    
            // 创建数据库连接
            this.db = new DB("bpmn.db");
            // 初始化数据库表
            this.db.execute(`
                create table if not exists process_instance (
                    id integer primary key autoincrement,
                    process_id text not null,
                    status text not null default ('Processing'),
                    variables text
                );
                create table if not exists process_activity_instance (
                    id integer primary key autoincrement,
                    activity_id text not null,
                    process_instance_id integer not null,
                    assignee text,
                    user text,
                    start_time integer not null,
                    end_time integer
                );
            `);
        }
    
        public start(variables: object): number {
            // 保存流程实例
            this.db.query("insert into process_instance (process_id, variables) values (?, ?)", [
                this.id,
                JSON.stringify(variables)
            ]);
            let process_instance_id = this.db.lastInsertRowId;
            
            // 发起流程活动
            this.onCreateTask(this.startends[0], process_instance_id, variables);
    
            return process_instance_id;
        }
    
        public tasks(assignee: string) {
            return this.db.queryEntries<{ id: number, activity_id: string, process_instance_id: number, assignee: string, user: string, start_time: number, end_time: number }>("select * from process_activity_instance where assignee = ? and end_time is null", [assignee]);
        }
    
        public complete(process_activity_instance_id: number) {
            let process_activity_instance = <{ activity_id: string; process_instance_id: number; }>this.db.queryEntries<{ activity_id: string, process_instance_id: number }>("select activity_id, process_instance_id from process_activity_instance where id = ?", [process_activity_instance_id]).pop();
            if (process_activity_instance == null) {
                throw new Error("Task is not found.");
            }
    
            let process_instance = <{ id: number; variables: string; }>this.db.queryEntries<{ id: number, variables: string }>("select id, variables from process_instance where id = ?", [process_activity_instance.process_instance_id]).pop();
    
            // 保存活动实例
            this.db.query("update process_activity_instance set end_time = ? where id = ?", [new Date().getTime(), process_activity_instance_id]);
    
            let variables = <object>JSON.parse(process_instance.variables);
    
            // 查找并执行下一步活动（集合）
            this.sequenceflows.filter(f => f.sourceRef === process_activity_instance.activity_id).map(f => f.targetRef).forEach(id => {
                this.onCreateTask(id, process_instance.id, variables);
            });
        }
    
        public history(process_instance_id: number) {
            return this.db.queryEntries<{ id: number, activity_id: string, process_instance_id: number, assignee: string, user: string, start_time: number, end_time: number }>("select * from process_activity_instance where process_instance_id = ? order by start_time desc", [process_instance_id]);
        }
    
        /**
         * 发起流程活动
         */
        private onCreateTask(activity_id: string, process_instance_id: number, variables: object) {
            let activity = this.activities[activity_id];
            if (activity == null) {
                throw new Error("Can not find the activity.");
            }
    
            switch (activity?.type) {
                case "startEvent": // 开始事件
                    this.sequenceflows.filter(f => f.sourceRef === activity_id).map(f => f.targetRef).forEach(id => {
                        this.onCreateTask(id, process_instance_id, variables);
                    });
                    break;
                case "endEvent": // 结束事件
                    this.db.query("update process_instance set status = 'Completed' where id = ?", [process_instance_id]);
                    break;
                case "userTask": // 用户任务
                    // 保存活动实例
                    this.db.query("insert into process_activity_instance (activity_id, process_instance_id, assignee, start_time) values (?, ?, ?, ?)", [
                        activity_id,
                        process_instance_id,
                        this.resolve(variables, activity.assignee),
                        new Date().getTime()
                    ]);
                    break;
                case "exclusiveGateway": // 排他网关
                    this.sequenceflows.filter(f => f.sourceRef === activity_id)
                        .filter(f => this.resolve(variables, f?.conditionExpression?.value) === "true") // 计算排他网关上的条件表达式
                        .forEach(f => this.onCreateTask(f.targetRef, process_instance_id, variables));
                    break;
                case "parallelGateway": // 平行网关
                    // 如果上一步有并行执行的活动，需要等待这些活动同时执行结束后才能触发
                    let { unfinished_tasks_count } = <{ unfinished_tasks_count: number }>this.db.queryEntries(`select count(id) as unfinished_tasks_count from process_activity_instance where process_instance_id = ? and activity_id in ('${this.sequenceflows.filter(f => f.targetRef === activity_id).map(f => f.sourceRef).join("', '")}') and end_time is null`, [process_instance_id]).pop();
                    if (unfinished_tasks_count === 0) {
                        this.sequenceflows.filter(f => f.sourceRef === activity_id)
                            .forEach(f => this.onCreateTask(f.targetRef, process_instance_id, variables));
                    }
                    break;
                default:
                    throw new Error(`The ${activity?.type} is not implemented.`);
            }
        }
    
        /**
         * 解析表达式
         */
        private resolve(variables: object, expression: string = ""): string {
            return eval("`" + Object.keys(variables).reduce((p, c) => { p = p.replace(c, "variables." + c); return p; }, expression) + "`");
        }
    }
    
    const expense = `<?xml version="1.0" encoding="UTF-8"?>
    <definitions
        xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        xmlns:activiti="http://activiti.org/bpmn"
        xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI"
        xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC"
        xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI"
        typeLanguage="http://www.w3.org/2001/XMLSchema"
        expressionLanguage="http://www.w3.org/1999/XPath"
        targetNamespace="http://www.activiti.org/test">
        
        <process id="expense" name="expense" isExecutable="true">
            <documentation>报销流程</documentation>
            <startEvent id="startevent1" name="Start"></startEvent>
            <userTask id="usertask1" name="填写报销单" activiti:assignee="\${expense.username}"></userTask>
            <sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
            <userTask id="usertask2" name="部门经理审批" activiti:assignee="\${deptManager}"></userTask>
            <sequenceFlow id="flow2" sourceRef="usertask1" targetRef="usertask2"></sequenceFlow>
            <exclusiveGateway id="exclusivegateway2" name="Exclusive Gateway"></exclusiveGateway>
            <sequenceFlow id="flow3" sourceRef="usertask2" targetRef="exclusivegateway2"></sequenceFlow>
            <userTask id="usertask3" name="人事审批" activiti:assignee="\${cho}"></userTask>
            <sequenceFlow id="flow6" sourceRef="exclusivegateway2" targetRef="usertask3">
                <conditionExpression xsi:type="tFormalExpression"><![CDATA[\${expense.amount <= 500}]]></conditionExpression>
            </sequenceFlow>
            <userTask id="usertask4" name="总经理审批" activiti:assignee="\${ceo}"></userTask>
            <sequenceFlow id="flow8" sourceRef="usertask4" targetRef="usertask3"></sequenceFlow>
            <sequenceFlow id="flow10" sourceRef="exclusivegateway2" targetRef="usertask4">
                <conditionExpression xsi:type="tFormalExpression"><![CDATA[\${expense.amount > 500}]]></conditionExpression>
            </sequenceFlow>
            <userTask id="usertask5" name="打印申请单" activiti:assignee="zhangsan"></userTask>
            <userTask id="usertask6" name="粘贴发票" activiti:assignee="lisi"></userTask>
            <parallelGateway id="parallelgateway1" name="Parallel Gateway"></parallelGateway>
            <sequenceFlow id="flow11" sourceRef="usertask3" targetRef="parallelgateway1"></sequenceFlow>
            <sequenceFlow id="flow12" sourceRef="parallelgateway1" targetRef="usertask5"></sequenceFlow>
            <sequenceFlow id="flow13" sourceRef="parallelgateway1" targetRef="usertask6"></sequenceFlow>
            <parallelGateway id="parallelgateway2" name="Parallel Gateway"></parallelGateway>
            <sequenceFlow id="flow15" sourceRef="usertask5" targetRef="parallelgateway2"></sequenceFlow>
            <sequenceFlow id="flow16" sourceRef="usertask6" targetRef="parallelgateway2"></sequenceFlow>
            <userTask id="usertask7" name="财务打款" activiti:assignee="\${cfo}"></userTask>
            <sequenceFlow id="flow17" sourceRef="parallelgateway2" targetRef="usertask7"></sequenceFlow>
            <endEvent id="endevent1" name="End"></endEvent>
            <sequenceFlow id="flow18" sourceRef="usertask7" targetRef="endevent1"></sequenceFlow>
        </process>
    </definitions>`;
    let bpmn = new BusinessProcessModelingNotation(expense); // 在线设计 https://bpmn.io/
    let process_instance_id = bpmn.start({
        expense: {
            username: "luffy",
            amount: 100
        },
        deptManager: "tom",
        ceo: "jerry",
        cho: "rose",
        cfo: "jack"
    });
    let task = bpmn.tasks("luffy").pop();
    bpmn.complete(task?.id || 0);
    task = bpmn.tasks("tom").pop();
    bpmn.complete(task?.id || 0);
    task = bpmn.tasks("rose").pop();
    bpmn.complete(task?.id || 0);
    task = bpmn.tasks("zhangsan").pop();
    bpmn.complete(task?.id || 0);
    task = bpmn.tasks("lisi").pop();
    bpmn.complete(task?.id || 0);
    task = bpmn.tasks("jack").pop();
    bpmn.complete(task?.id || 0);
    console.log(bpmn.history(process_instance_id));
    ```

# 数学公式

- 香农熵（信息熵）  
    对于任意一个随机变量 X（比如得冠军的球队），它的熵定义如下：变量的不确定性越大，熵也就越大，把它搞清楚所需要的信息量也就越大。  
    <!-- ![](https://gss0.bdstatic.com/-4o3dSag_xI4khGkpoWK1HF6hhy/baike/s%3D220/sign=b531b5c5277f9e2f74351a0a2f31e962/0b46f21fbe096b63e29e59730c338744ebf8ac09.jpg)   -->
    $$ H(x)=-\sum_{x}P(x)log_2[P(x)] $$
    > P(x) 表示随机变量某个取值的概率

- 凯利公式

- 圆周率
    - [使用泰勒级数公式计算圆周率](https://www.mk2048.com/blog/blog_h1c0k0ikkkk0j.html)
        ```javascript
        (function (num) {
            let i = 1n,
                x = 3n * (10n ** (20n + num)),
                p = x;
            while (x > 0) {
                x = x * i / ((i + 1n) * 4n);
                p += x / (i + 2n);
                i += 2n;
            }
            return p / (10n ** 20n);
        })(1000n); // 计算包含 1000 位小数的圆周率：31415926...1989n
        ```
    - 使用贝拉公式（衍生自 BBP 公式）计算圆周率 π 的第 n 位数
