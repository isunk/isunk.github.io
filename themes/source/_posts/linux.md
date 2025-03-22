---
title: Linux 命令
tags: linux
categories: linux
---

## 文件管理

- 常用文件管理命令
    ```bash
    # # 强制删除目录以及其下所有文件、文件夹（参数 f 表示强制，不推荐使用；参数 r 表示递归删除）
    # rm -rf folder
    # 递归删除目录
    rm -r folder
    # 删除文件 hello.txt
    rm hello.txt

    # 彻底删除、粉碎文件
    shred -u hello.txt

    # 显示当前目录，print working directory(打印工作目录)
    pwd

    # 创建一个空白的文件
    touch file

    # 创建一个临时文件（"tmp.XXX" 为模板，其中的 "X" 数量必须大于等于 3），如 tmp.hKg
    mktemp tmp.XXX

    # 在 test.txt 中查找 money 这个字符串，grep 查找是区分大小写的
    grep money test.txt
    ls -l | grep -Po 'git-[a-z]+.exe' # 参数 “-o” 表示只显示匹配内容

    # 把 file1 的文件内容加上行号后输入 file2 这个文件里
    cat -n file1 > file2

    # 把 file1 和 file2 的文件内容加上行号（空白行不加）之后将内容附加到 file3 里
    cat -b file1 file2 >> file3
     
    # 把 test.log 文件扔进垃圾箱，赋空值 test.log（清空文本 test.log 内容）
    cat /dev/null > test.log

    # 显示 xxx.csv 文件中的第二、第三列
    awk -F ',' '{ print $2, $3 }' xxx.csv

    # 清空文本 test.log 内容
    echo > test.log

    # 实时显示新追加到文件中的信息
    # tail -f test.log # "-f" 表示根据文件描述符进行追踪，当文件改名或被删除，追踪停止
    tail -F test.log # "-F" 表示根据文件名进行追踪，并保持重试，即该文件被删除或改名后，如果再次创建相同的文件名，会继续追踪

    # 以十六进制的方式读取文件 test.log
    od -t x1 test.log
    xxd \
        -u \ # 显示大写字母（默认为小写字母）
        -l 160 \ # 输出 160 个字符后停止
        test.log

    # 以十六进制的方式读取文件 test.txt
    hexdump -C test.txt

    # 查看文件 test.log 的系统属性信息
    stat test.log

    # 以 utf-8 编码显示文件 test.txt 内容  
    iconv -t utf-8 test.txt

    # 查看程序的动态依赖库，如返回 "not a dynamic executable" 则表示该可执行文件不依赖任何动态链接库
    ldd /bin/bash
    # 将 /bin/bash 以及其依赖打包压缩至文件 rootfs.tar.gz 中
    ldd /bin/bash | perl -p -e 's#^\s*(\S+ => )?(/\S+) \(0x[0-9a-z]+\)$#$2#g' | grep -v 'linux-vdso' | sed '$a/bin/bash' | xargs tar zcvf rootfs.tar.gz

    # 打开一个 bash，参数 “--rcfile” 指定预加载的 .bashrc 文件
    bash --rcfile .bashrc

    # 下载文件（"-k" 表示忽略 https 证书校验，"-O" 表示使用 URL 中默认的文件名保存文件到本地，"-s" 表示不显示下载进度，"-L" 表示自动跟踪 URL 重定向跳转，"--retry 3 --retry-delay 2" 表示失败重试 3 次，每次间隔 2 秒）
    curl -k -O -s -L https://github.com/btraceio/btrace/releases/download/v1.3.11.1/btrace-bin-1.3.11.1.zip --retry 3 --retry-delay 2 > /dev/null 2>&1
    # curl -k -s -L https://sourceforge.net/projects/jodconverter/files/JODConverter/2.2.2/jodconverter-2.2.2.zip/download -o jodconverter-2.2.2.zip --retry 3 --retry-delay 2 > /dev/null 2>&1

    # 取某个路径下的目录名
    dirname /home/test.txt # /home
    # 取某个路径下的文件名
    basename /home/test.txt # test.txt
    # 取某个路径下的文件名（并去除文件后缀 .txt）
    basename /home/test.txt .txt # test
    ```

- 根据文件名 "filename" 查找文件  
    `find -name filename`  
    在当前目录下所有txt文件中查找内容 "word"  
    `find -name "*.txt" | grep "word"`  
    `find . -name "*.txt" | grep "word"`  
    删除当前目录及子目录下所有 *.lastUpdated 文件  
    `find -name "*.lastUpdated" | xargs rm`  
    强制删除当前目录及子目录下所有 *.git 目录及其子文件  
    `find -name ".git" | xargs rm -rf`  
    删除 7 天前的文件（可用于定期手工清理 tomcat 日志文件）  
    `find -mtime +7 -exec rm -f {} \;`  
    把 /home/logs/ 目录下所有 *.log 文件中的字符 "abc" 替换成 "def"  
    `find /home/logs/ -name "*.log" | xargs sed -i "s/abc/def/g"`  
    统计当前目录及子目录下所有 *.java 文件的总行数  
    `find -name "*.java" | xargs wc -l`  
    统计当前目录及子目录下所有文件的类型（后缀名）  
    `find -name "*.*" | sed 's/.*\./\./' | sort | uniq`  
    查找当前目录下所有 *. java、*. xml、*. txt文件（参数 "-o" 表示或者）  
    `find -name *. java -o -name *. xml -o -name *. txt`  
    查找当前目录下所有文件，并列出其详细信息  
    `find ./ ! -type d -exec ls -l {} \;`  
    遍历查找当前目录下所有文件内容，查找包含内容 '"id": ' 的行，使用正则替换，筛选出 id 的值并作为数进行排序，取最后一个值（即找出当前目录下所有文件中定义的 id 最大值）
    `grep -rn '"id": ' . | sed 's/^.*"id": \([0-9]\+\),/\1/' | sort -n | tail -n 1`  
    `printf "%d\n" $(($(grep -rn '"id": ' . | sed 's/^.*"id": \([0-9]\+\),/\1/' | sort -n | tail -n 1) + 1))`  
    `printf "%04x\n" $((16#$(grep -rn '"key": ' . | sed 's/^.*"key": "\([a-zA-Z0-9]\+\)",/\1/g' | sort -n | tail -n 1) + 1))`  
    批量重命名文件  
    `find -name '*.json' | cut -c 3- | xargs -i mv {} A_{}`  
    查询当前目录下所有目录（不包含子目录），并删除目录名称中的 B_ 前缀  
    `find . -maxdepth 1 -type d | tail -n +2 | sed 's#^./B_##' | xargs -I{} mv ./B_{} ./{}`  
    查询当前目录下所有 zip 文件，并将文件名中 "_" 去除  
    ```shell
    find *.zip | while read n; do mv $n `echo "$n" | sed 's/_//g'`; done
    ```
    查找当前目录下重复的多版本 jar 包  
    `ls -a | sed 's/-[^-]*\.jar$//g' | sort | uniq -d`  
    从文件中随机选择一行  
    `shuf pokemon.txt -n 1`

- 监听文件 catalina.out，直到文件中出现 "org.apache.catalina.startup.Catalina.start Server startup in [0-9]+ ms" 后退出
    ```bash
    # (tail -f -n0 catalina.out &) | grep -m 1 -E "org.apache.catalina.startup.Catalina(.start)|- Server startup in [0-9]+ ms" && pkill -s 0 tail

    # (tail -f -n0 catalina.out &) | (grep -m 1 -E "org.apache.catalina.startup.Catalina(.start)|- Server startup in [0-9]+ ms" && ( \
	# 	ps | grep tail | grep -oP '^\s*([0-9]+)\s+' | sed 's/ //g' | xargs kill -15 \
	# )) > /dev/null

    tail -F -n 0 catalina.out | grep -m 1 -P "org.apache.catalina.startup.Catalina(.start)|- Server startup in [0-9]+ ms" && (
        echo 'Tomcat has started.'
        # ...
    )
    ```

- 修改文件读写权限（文件或目录的拥有者、文件或目录的所属群组、其他用户。读取权限 'r' 代号为 "4"，写入权限 'w' 代号为 "2"，执行或切换权限 'x' 代号为 "1"，不具任何权限 '-' 代号为 "0"）  
    `chmod 777 filename`  
    `chmod ugo+rwx filename`  
    修改文件读写权限为 "-rwsr-xr-x"（第一个 4 表示给 /bin/cat 加上 setUID 权限，系统可以临时把这个文件的所有者（root）身份角色赋给普通用户）（[特殊权限 "setuid" 代号为 "4"，"setgid" 代号为 "2"，"stick bit" 代号为 "1"](http://www.linuxidc.com/Linux/2013-08/88587.htm)）  
    `chmod 4755 /bin/cat`  
    修改目录 "folder" 以及其下所有文件、文件夹读写权限  
    `chmod -R 777 folder`  
    文件增加执行权限  
    `chmod +x filename`  
    当前用户增加执行权限，组用户增加写权限  
    `chmod u+x g+w filename`  
    当前用户具有读、写、执行权限，组用户具有读写权限，其他用户具有读权限  
    `chmod u=rwx g=rw o=r filename`  
    修改文件的属主为 root  
    `chown root filename`  
    `chown -R root filename`  
    修改文件的属主为 sa，群组为 sagroup  
    `chown sa:sagroup filename`  
    修改文件的群组为 sagroup  
    `chgrp sagroup filename`  
    查看、设置当前用户的 umask（umask 决定目录和文件被创建时得到的初始权限）
    ```bash
    # 查看当前用户的 umask
    umask

    # 设置当前用户的 umask
    umask 022
    # "022" 是八进制格式数据，二进制值为 10010，字符表示为 "----w--w-"。而 umask 为屏蔽的意思，即数值取反，二进制取反值为 "111101101"，八进制取反值为 "755"，字符表示为 "rwxr-xr-x"
    ```
    suid（4）、sgid（2）、sbit（1）  
    ```bash
    # suid
    # 只要一个命令文件在属主权限位上的x变成s的话，那么其它用户在执行这个命令文件时，就会以该命令文件的属主用户身份去执行
    chmod u+s filename # 授权 suid
    chmod u-s filename # 取消授权 suid

    # sgid
    # 一般情况下是设置给目录使用的，主要目的就是为了让别的用户无法删除其它用户所创建的文件或目录
    chmod g+s filename
    chmod g-s filename

    # sbit（sticky bit，即沾滞位）
    # 只作用在目录上，当一个目录的没有设置sticky bit权限时，并且该目录对所有用户都有读、写、执行权限时，普通用户在该目录下所创建的文件或目录都会被其它用户删除
    chmod o+t filename
    chmod o-t filename
    ```
    > [文件信息](https://www.zhihu.com/tardis/sogou/art/29945600)
    > ```
    > drwxr-xr-- 4 root everybody 3488 Mar  2 21:30 src
    > # 第一列，第一位 `d` 表示文件类型为目录，第一组 `rwx` 表示文件的属主 `root` 用户的权限为：可读、可写、可执行，第二组 `r-x` 表示文件所属群组 `everybody` 的权限为：可读、可执行，第三组 `r--` 表示其他用户的权限为：可读。
    > # 第二列，`4` 表示有 4 个文件硬链接到此节点（i-node）：`.`、`..`、该目录下的两 2 个子目录（不计算子目录下的子目录）。当我们新建一个新的目录时，新的目录连接数为 2，而上层目录的连接数会增加 1。
    > # 第三列表示文件属主为 `root` 用户。
    > # 第四列表示文件所属组为 `everybody`。
    > # 第五列表示文件大小为 `3488` bytes。
    > # 第六列表示文件最近修改时间。
    > # 第七列为文件（这里是目录）名称。
    > -rw-rw---- 1 root everybody 2897 Jun  7 23:39 readme.md
    > # 第一列，第一位 `-` 表示文件类型为文件。
    > # 第二列，`1` 表示只有 1 个文件硬链接到此节点，即该文件除了自身，无硬连接。
    > ```

- 保护文件，禁止修改删除移动  
    `chattr +i hello.txt`  
    去保护  
    `chattr -i hello.txt`

- 创建文件或目录 a 的硬连接 b（新建的文件是原文件的一个别名，当原文件被删除，新建的文件仍然可用）  
    `ln a b`  
    创建文件或目录 a 的软连接（又称符号连接，类似于快捷方式，新建的文件可以指向不存在的文件）  
    `ln -s a b`  
    删除连接 b  
    `rm -f b`

- 拷贝本地目录 /home/sa 目录及子文件到远程主机 192.168.1.1 的 /home/sa  
    `scp -r /home/sa root@192.168.1.1:/home/sa`  
    将远程主机 192.168.1.1 上的 /home/sa/info.log 复制到本机的 /tmp 下  
    `scp root@192.168.1.1:/home/sa/info.log /tmp`

- 同步远程主机 192.168.1.1 上目录 /home/sa/dic/ 下除 .* 和 log 之外的文件到本地目录 /home/sa/dic 下  
    `rsync -a --delete --exclude=".*" --exclude="log" root@192.168.1.1:/home/sa/dic/ /home/sa/dic/`

- [使用 GPG 对文件加密](http://www.cnblogs.com/AloneSword/p/4918791.html)
    生成（自己的）密钥  
    `gpg --gen-key`  
    导出（自己的）公钥，其中（自己的）用户 ID 为 "Ke Sun <ke.sun@gmail.com>"  
    `gpg --armor --output public-key.txt --export "Ke Sun <ke.sun@gmail.com>"`  
    导入（他人的）公钥  
    `gpg --import public-key.txt`  
    生成文件 "hello.txt" 的签名 "hello.txt.asc"  
    `gpg --armor --detach-sign hello.txt`  
    校验文件 "hello.txt" 的签名 "hello.txt.asc"  
    `gpg --verify hello.txt.asc hello.txt`

- 计算文件的 sha-1 校验和  
    `sha1sum a.txt`  
    计算文件的 md5 校验和  
    `md5sum a.txt`  
    计算文件的 md5 校验和，并保存至文件 hello.txt.md5 中  
    ~~`md5sum < hello.txt > hello.txt.md5`~~  
    `md5sum hello.txt > hello.txt.md5`  
    根据 hello.txt.md5 校验相应的文件  
    `md5sum -c hello.txt.md5`  
    计算字符串 "hello, world" 的 md5 校验和  
    `echo -n "hello, world" | md5sum`

- 查找命令文件所在位置  
    `which sha1sum`

- 显示 /etc/hosts 的路径  
    `ls /etc/hosts`  
    列出目录 /etc 下所有文件  
    `ls -l /etc`  
    `ls -a /etc`  
    列出当前目录下所有文件和目录，并着色（参数 "--color" 表示对结果进行着色）  
    `ls --color`  
    参数 "-h" 表示友好显示文件大小，如 4.0K、3.2M  
    `ls -lh`

- 使用 unix2dos 将文件 test.txt 由 unix 格式转换为 windows 格式（即将 "\n" 替换为 "\n\r"）  
    `unix2dos test.txt`  
    `/bin/find -name '*.txt' | xargs unix2dos`  
    使用 dos2unix 将文件 test.txt 由 windows 格式转换为 unix 格式（即将 "\n\r" 替换为 "\n"）  
    `dos2unix test.txt`

- 使用 diff 简单比较两个文件 a.txt、b.txt  
    `diff a.txt b.txt`  
    使用 diff 比较两个目录 a、b  
    `diff -ywrNa ./a ./b`  
    使用 diff 比较两个目录 folder1、folder2 下所有文件，但不包括 ".git"、"bin" 目录和 ".classpath" 文件  
    ~~`diff -ywrNa folder1/ folder2/ -x .git -x bin -X .classpath`~~  
    `diff -ywrNa folder1/ folder2/ -x .git -x bin -x .classpath`  
    使用diff比较两个文件 a.txt、b.txt 并计数不相同的行数  
    `diff -yw a.txt b.txt | grep -E '<|>|\|' | wc -l`  
    比较两个文件 f1、f2，并导出补丁脚本（不包括相同的行）  
    `diff -Nu f1 f2 | grep -E '^[^ ]'`  
    比较两个目录 d1、d2，并导出补丁脚本（不包括相同的行）  
    `diff -Nur ./d1 ./d2 | grep -E '^[^ ]'`  
- 比较两个文件 f1、f2，并导出补丁脚本 f2.patch（windows 下可使用 TortoiseUDiff 或 TortoiseGitUDiff 打开 .patch/.diff 文件）  
    `diff -Nu f1 f2 > f2.patch`  
    给文件 f1 打补丁 f2.patch，结果 f1 应与 f2 一致  
    ~~`patch -p0 < f2.patch`~~  
    `patch f1 < f2.patch`  
    给文件 f1 撤销补丁 f2.patch，文件 f1 被还原  
    ~~`patch -RE -p0 < f2.patch`~~  
    `patch -R f1 < f2.patch`
- 比较两个目录、导出补丁脚本、打补丁、撤销补丁
    1. 比较两个目录 d1、d2，并导出补丁脚本 d2.patch  
        `diff -Nur ./d1 ./d2 > d2.patch`
    2. 给目录 d1 打补丁 d2.patch，结果 d1 应与 d2 一致
        - 方法一：  
            `patch --binary --dir ./d1 < d2.patch`
        - 方法二：  
            `cd ./d1 && patch --binary -p1 < d2.patch && cd ../`
    3. 给目录 d1 撤销补丁 d2.patch，目录 d1 被还原
        - 方法一：  
            `patch --binary -R --dir ./d1 < d2.patch`
        - 方法二：  
            `cd ./d1 && patch --binary -R -p1 < d2.patch && cd ../`

- 将 docx 文件转换为 txt 文件  
    `docx2txt *.docx`

- 交集、并集、差集
    ```bash
    # 交集
    #grep -F -f a.txt b.txt
    #cat b.txt | grep -F -f a.txt
    sort a.txt b.txt | uniq -d
    
    # 并集
    #cat a.txt b.txt | sort | uniq
    sort a.txt b.txt | uniq
    
    # 差集（a.txt - b.txt）
    #grep -F -v -f b.txt a.txt
    #cat a.txt | grep -F -v -f b.txt
    sort a.txt b.txt b.txt | uniq -u
    ```

## 磁盘管理

- 返回到上一次的工作目录  
    `cd -`  
    返回上一层目录  
    `cd ../`  
    返回到用户的家目录  
    `cd ~`  
    进入到文件 /etc/sudoers 所在目录  
    `cd $(dirname /etc/sudoers)`

- 统计 / 目录下目录或文件所占用磁盘大小，"-d 1" 表示遍历的目录深度为 1  
    `du -h -d 1 /`

- 快速制作系统盘  
    `dd if=ubuntu-server-amd64.iso of=/dev/sdb`  
    安全擦除硬盘数据  
    `dd if=/dev/urandom of=/dev/sda`

## 文档编辑

- [vim 编辑器](https://www.jianshu.com/p/b5ec0df362e5)
    - 使用 vi 编辑器编辑文本文件 temp.txt  
        `vi temp.txt`  
    - 以二进制模式（binary mode）编辑文件 temp.txt（该模式下，\r 会显示为 ^M，即不可打印字符（non-printable char））  
        `vi -b temp.txt`
    - 同时打开多个文件  
        `vi file1 file2 file3 ...`
    > - Edit 模式（刚启动时 vim 工作于 edit 模式, 主要是用来进行一下文本操作）
    > - Insert 模式（键入 "i" 进入该模式，按下 <Esc> 退出该模式）
    > - Command 模式（使用 ":"，按下 <Esc> 键退出 Insert 模式回到 Normal 模式）
    >   - 输入 `:%!xxd` 可将当前文件转换为 16 进制文本，再次输入 `:%!xxd -r` 可将 16 进制文本转换会原文本
    >   - 先键入 `gg`，再键入 `=`，再键入 `G`，可使全部代码自动对齐
    >   - 输入 `:X` 可设置当前文本加密，按照提示输入密码后保存退出完成加密（若输入密码为空，则表示不加密，即解密）
    >       > 若提示 "E21: Cannot make changes, 'modifiable' is off" 错误信息，需输入 `:set modifiable` 将文件设置为可修改模式，之后再重复上一步骤中的命令
    >   - 输入 `:s/old/new`，表示替换当前行中第一个 old 为 new  
    >       输入 `:%s/old/new/g`，表示替换全文中所有 old 为 new  
    >   - 输入 `/keyword`，表示向下搜索 keyword 关键词（按 <N> 表示查找下一个）
    >   - 输入 `?keyword`，表示向上搜索 keyword 关键词（按 <N> 表示查找上一个）
    >   - 输入 `:set number` 显示行号
    >   - 输入 `:set hlsearch` 开启搜索结果高亮

- 在当前目录下递归查找内容 "word"  
    `grep -rn word ./`  
    在文件 "temp.txt" 中使用正则表达式 "([0-9]+)(\.[0-9]+){3}" 查找 IP 地址（参数 "-E" 表示使用正则表达式，参数 "-o" 表示只显示一行中匹配的部分，<font color=red>grep 无法匹配 CRLF 换行符，即无法跨行匹配</font>）  
    `grep -E '([0-9]+)(\.[0-9]+){3}' temp.txt`  
    ~~`grep -oE '([0-9]+)(\.[0-9]+){3}' *.txt`~~  
    在文件 *.txt 中查找字符串 "hello"，并显示其所在行以及上下 5 行（参数 "-C" 表示匹配字符串所在行以及上下几行，参数 "-B" 表示显示字符串所在行及前几行，参数 "-A" 表示显示字符串所在行及后几行）  
    `grep -C 5 'hello' *.txt`  
    查找 catalina.sh 文件中 webresource 所在行号  
    `grep -n webresource catalina.sh | cut -d : -f1`
    查找 catalina.sh 文件中 webresource 所在行，并打印下一行行号  
    `echo $(($(grep -n webresource catalina.sh | cut -d : -f1) + 1))`  
    使用正则表达式 [a-zA-Z0-9_]+ 查找所有的匹配项（tail -n +7 表示忽略结果中的前7行）  
    `curl http://127.0.0.1:8080/admin/collections?action=List | grep -o -E '[a-zA-Z0-9_]+' | tail -n +7`  
    参数 “-P” 表示使用 perl 正则表达式查询  
    `grep -P '^PermitRootLogin ((?!no)\S)+$' /etc/ssh/sshd_config`  
    参数 “--color=auto” 表示高亮显示结果  
    `grep --color=auto 'hello, world' hello.txt`  
    参数 “--exclude” 表示排除文件  
    `grep -l -r --include '*.csv' --exclude 'res.csv' --exclude 'list.txt' 'hello, world' . | wc -l`
- 就地替换文件 debug.log 中所有 "world" 为 "luffy"（参数 "-i" 表示直接修改文件内容）  
    `sed -i 's/world/luffy/g' debug.log`  
    将文件 debug.log 中所有 "world" 替换为 "luffy" 并保存至 "debug.new.log" 中  
    `sed 's/world/luffy/g' debug.log > debug.new.log`  
    在文件 "temp.txt" 中使用正则表达式 "([0-9]+)(\.[0-9]+){3}" 查找 IP 地址（参数 "-r" 表示使用正则表达式，参数 "-n" 和标志 "p" 表示只打印匹配的行）  
    `sed -nr '/([0-9]+)(\.[0-9]+){3}/p' temp.txt`  
    在文件 "temp.txt" 末尾添加一行 "hello, world"  
    `sed '$ahello, world' hello.txt`  
    将所有的行（以 "|" 为分隔符，拼接为一行）  
    `sed ':a;N;$!ba;s/\n/|/g' xxx.txt`  
    查找 catalina.sh 文件中 webresource 所在行，并在下一行添加一行 "echo hello, world"  
    `sed "$(($(grep -n webresource catalina.sh | cut -d : -f1) + 1))"'iecho hello, world' catalina.sh`  
    或
    ```bash
    sed '/webresource/ {
      n
      i echo hello, world
    }' catalina.sh
    # 其中 /webresource/ 表示匹配 webresource 所在行，n 表示下一行，i 表示插入行， r 表示插入文件内容（如 r hello.txt）
    ```
    筛选第二行内容  
    `echo -e "1\n2\n3"  | sed -n 2p`
    > 其中
    > - 参数，如 sed -i '/.../.../' ...
    >   - `i` 表示将修改替换的内容写入到原文件中
    > - 查找标志，如 sed 's/.../.../' ...
    >   - `s` 表示替换内容
    >   - `n` 表示新增空行
    >   - `i` 表示插入一行，如在第四行插入内容 sed '4ihello, world' ...
    > - 替换标志，如 sed '/.../.../g' ...
    >   - `g` 表示全局替换
    >   - `i` 表示不区分大小写

    替换配置文件中第一次出现的文本内容  
    `sed -i '0,/IP = .*/s/IP = .*/IP = 10.10.10.10/' /usr/share/config.conf`  
    删除包含字符串 "test" 的行  
    `sed -i '/test/d' log.txt`
- awk
    ```bash
    # awk 忽略引号中的分隔符
    echo 'a,b,c,"d,e",f' | awk -v -FPAT '([^,]*)|("[^"]*")' '{print $4}' # "d,e"

    # awk 删除行
    awk -v -FPAT '([^,]*)|("[^"]|""*")' '{ for (i = 1; i <= NF; i++) { if (i != 2) { printf("%s", $i); if (i != NF) printf(","); } } printf("\n") }' test.csv 1<> test.csv

    git diff | grep -P '^(?!---)-' | sed 's/^-//g' | awk -v -FPAT '([^,]*)|("[^"]|""*")' 'function contain(diamond, rough, x, y) { for (x in rough) y[rough[x]]; return diamond in y; } BEGIN { split("20", indexs, ",") } { for (i = 1; i <= NF; i++) { if (!contain(i, indexs)) { printf("%s", $i); if (i != NF) printf(","); } } printf("\n") }' > temp.okf
    
    # 根据文件属主，分组显示文件
    ls -l | tail -n +2 | awk '{ ds[$3] += 1; fs[$3, ds[$3]] = $9; } END { for (x in ds) { print x; for (y = 1; y < ds[x]; y++) { printf("  %s\n", fs[x, y]); } } }'
    
    # 查找以 2019 开头的行记录
    awk '{if(/^2019/) print}' ids.txt
    awk '{if($0~"201709") print}' ids.txt
    ```

- [使用正则表达式替换文本内容（不保存修改到文件）](https://stackoverflow.com/questions/1348124/replace-an-xml-elements-value-sed-regular-expression)  
    `perl -p -0777 -e 's/<Context docBase="solr"[^>]*>//' server.xml`  
    使用正则表达式替换文本内容  
    `perl -p -0777 -i -e 's/<Context docBase="solr"[^>]*>//' server.xml`  
    参数 -0777 会忽略分割符，无法匹配行首和行尾  
    `perl -p -i -e 's/^hello$/world/' hello.txt`
- 使用 cut 截断字符串（截取 当前登录信息中 "(" 和 ")" 之间的内容，即登录者的 ip）  
    `who am i | cut -d '(' -f2 | cut -d ')' -f1`

- 向文件 temp.txt 中写入一行或多行内容，输入结束后，另起一行输入 "EOF" 表示结束写操作
    ```bash
    # cat << EOF # 使用 EOF，需要对特殊字符如 "$"、"`"进行转义 "\$"、"\`"
    # cat << "EOF" # 加引号如 "EOF" 则特殊字符无需转义
    cat >> temp.txt << "EOF"
    hello, world
    EOF
    ```

- 大文件分割，如将文件 catalina.out 分割为多个以 catalina_ 为前缀的文件（单个文件不超过 10 MB），如 catalina_aa、catalina_ab、catalina_ac...  
    `split -C 10M catalina.out catalina_`

- 使用 jq 解析 json
    1. 安装  
        `apt install jq`
    - 格式化显示 package.json 文件中的 json 报文  
        `jq . package.json`
    - 格式化显示 /author/name 属性  
        `jq .author.name package.json`
    - 格式化显示 /contributors 属性  
        `jq -r .contributors[] package.json`
    - 格式化显示 /contributors 数组的第一个元素的 name 属性  
        `jq .contributors[0].name package.json`
    - 压缩 json 报文，并写入到文件 package.min.json 中  
        `jq --compact-output . package.json> package.min.json`
    - 筛选出 /users 数组元素中 id、name 属性  
        `cat users.json | jq '.users[] | { id, name }'`  
        筛选出 name 为 zhangsan 的 user  
        `jq -r '.users[] | select(.name == "zhangsan")' users.json`  
        筛选出 name 为 zhangsan 或 lisi 的 user  
        `jq -r '.users[] | select(.name == "zhangsan", .name == "lisi")' users.json`  
        筛选出 age 大于 18，并且 name 为 zhangsan 的 user，并过滤（只展示）name 属性  
        `jq -r '.users[] | select(.age > 18) | select(.name == "zhangsan") | .name' users.json`
    - 筛选出 /users 数组元素中 id、name 属性，将结果中每两行拼接为一行（最终每行以 "[id]  [name]" 显示 users）  
        `jq -r '.users[] | { id, name }[]' users.json | sed 'N;s/\n/\t/g'`
    - 循环遍历 /users 数组元素中 id、name 属性，以 "id: $id, name: $name" 格式显示  
        `jq -r '.[] | .id + " " + .name ' users.json | while read -r id name; do echo "id: $id, name: $name"; done`

- 使用 xmllint 解析 xml
    - 格式化显示 xml 文档  
        `echo -n "$xml_content" | xmllint --format -`
    - 获取文件 users.xml 中 `//user` 节点下的内容  
        `xmllint --xpath '//user/text()' users.xml`
    - [获取 pom.xml 文件中的 groupId、artifactId、version](https://stackoverflow.com/questions/41114695/get-pom-xml-version-with-xmllint)  
        `xmllint --xpath 'concat(/*[local-name()="project"]/*[local-name()="groupId"]/text(),"  ",/*[local-name()="project"]/*[local-name()="artifactId"]/text(),"  ",/*[local-name()="project"]/*[local-name()="version"]/text())' pom.xml`

## 备份压缩

- 压缩打包目录 "content" 至文件 "file.tar.gz"  
    `tar -zcvf file.tar.gz ./content`  
    查阅压缩包文件 "file.tar.gz"  
    `tar -ztvf file.tar.gz`  
    解压缩文件 "file.tar.gz"  
    `tar -zxvf file.tar.gz`  
    `tar -jxvf ffmpeg-3.2.2.tar.bz2`  
    解压指定文件或目录  
    `tar zxvf tomcat.tar.gz tomcat/conf/server.xml`  
    查找所有包含名称 "\*.usl" 的文件并打包压缩 "usl.tar.gz" 到当前目录（保持目录结构）  
    `find . -name "*.usl" | xargs tar zcvf usl.tar.gz`  
    查找当前目录及子目录下所有包含名称 *.java、 *.xml、 *.jsp 的文件，并将其打包到 p.tar 中  
    `find -name "*.java" -o -name '*.xml' -o -name *.jsp | xargs tar rvf p.tar`
- 使用 bzip2 压缩 messages 文本文件（/var/log/messages，即系统日志文件）为 messages.bz2  
    `bzip2 messages`  
    使用 bzip2 解压缩 messages.bz2 文件为 messages  
    `bzip2 -d messages.bz2`
- 使用 gzip 解压缩 train-labels-idx1-ubyte.gz  
    `gzip -d train-labels-idx1-ubyte.gz`

- 将文件 "hello.txt" 添加至压缩包 "hello.zip" 中  
    `zip hello.zip hello.txt`  
    解压缩 "hello.zip" 至当前目录下  
    `unzip hello.zip`  
    解压缩 "arc.zip" 中的 src\hello.java 至 \tmp 目录下  
    `unzip -j "arc.zip" src\hello.java -d \tmp`  
    查看 "hello.zip" 中的文件  
    `unzip -v hello.zip`  
    将文件 "file1"、"file2" 添加至压缩包 "result.zip" 中  
    `zip result.zip file1 file2`  
    将 "tomcat-native-1.1.34-win32-bin.zip" 中的 bin/x64 目录下的 tcnative-1.dll 解压到 $CATALINA_HOME/bin 目录下  
    `unzip -j -q tomcat-native-1.1.34-win32-bin.zip 'bin/x64/tcnative-1.dll' -d $CATALINA_HOME/bin`

- 将文件（或目录） "hello.txt" 压缩至压缩包 "hello.7z" 中  
    `7z a hello.7z hello.txt`  
    解压缩 "hello.7z" 至当前目录下  
    `7z x hello.7z`  
    解压缩 "hello.rar" 至当前目录下  
    `7z x hello.rar`

## 系统设置

- 查看服务  
    `chkconfig`  
    查看所有服务（ubuntu，the status is [ + ] for running services, [ - ] for stopped services and [ ? ] for services without a 'status' command.）  
    `service --status-all`

- 显示环境变量 HOME  
    `echo $HOME`  
    设置临时环境变量(只对当前进程有效，不会传递给子进程)  
    `path=jdk/bin:${PATH}`  
    设置环境变量（变量在关闭 shell 时失效，不仅对当前进程有效，而且会传递给子进程）（export 用于 bash，setenv 用于 csh）  
    `export name=value`  
    `setenv name value`  
    显示当前用户的变量  
    `env`  
    显示当前 shell 的变量，包括当前用户的变量  
    `set`  
    `export`  
    删除环境变量  
    `unset name`

- 添加全局的环境变量  
    `echo 'export PATH="$PATH:/opt/jdk/bin"' >> ~/.cshrc`  
    添加当前用户环境变量（需要注销后才能生效）  
    `echo 'export PATH="$PATH:/opt/jdk/bin"' >> /etc/profile`

- 安装 deb 软件包 "app.deb"（Debian、Ubuntu 等 Linux 发行版的软件安装包）  
    `sudo dpkg -i app.deb`  
    删除软件包 "app"（删除配置信息）  
    `sudo dpkg -P app`  
    删除软件包 "app"（保留配置信息）  
    `sudo dpkg -r app`  
    显示所有已经安装的 deb 包，同时显示版本号以及简短说明  
    `dpkg -l`

- 安装 rpm 软件包 "app.rpm"（Red Hat、Fedora、CentOS、SuSE 等 Linux 发行版的软件安装包）  
    `sudo rpm -i app.rpm`  
    删除软件包 "app"（删除配置信息）  
    `sudo rpm -e app`  
    显示所有已经安装的 deb 包，同时显示版本号以及简短说明  
    `rpm -qa`

- 定时任务  
    ```bash
    # 查询定时任务
    crontab -l

    # 查询定时任务，不现实注释，并将结果导出到文件 crontab.data
    crontab -l | grep -v '^#' > crontab.data

    # 添加定时任务，每小时的第一分钟会发送广播 "hello, world"
    # 格式为：minute hour day month dayofweek command
    # minute: 每个小时的第几分钟执行该任务，hour: 每天的第几个小时执行该任务，day: 每月的第几天执行该任务， month: 每年的第几个月执行该任务，dayofweek: 每周的第几天执行该任务
    echo "1 * * * * echo \"hello, world\" | wall" >> crontab.data
    crontab crontab.data

    # 删除当前用户下的所有定时任务  
    crontab -r
    ```

- 查看命令别名  
    `alias`  
    设置别名  
    `alias ll=ls -l`

## 系统管理

- 以 root 用户权限执行命令 "touch file1"  
    `sudo touch file1`  
    切换当前用户至 root 用户  
    `su`

- 关机重启  
    ```bash
    # 重启
    sudo reboot
    sudo shutdown -r now
    sudo shutdown -r 10 # 过 10 分钟自动重启
    sudo shutdown -r 20:00 # 在时间为 20:00 时候重启
    # 关机
    sudo halt
    sudo poweroff
    sudo shutdown -h now
    sudo shutdown -h 10 # 10 分钟后自动关机
    sudo shutdown -h 20:00 # 在时间为 20:00 时候关机
    # 注销
    logout
    ```

- 查询当前所有登录的终端用户信息、ip  
    `who`  
    查询当前登录的用户信息  
    `who am i`  
    `whoami`  
    显示目前登入系统的用户信息  
    `w`

- 查看历史命令执行记录  
    `history`  
    > - 设置历史命令执行记录的格式为 "[日期 时间][ip地址][用户名]命令"  
    >   `export HISTTIMEFORMAT="[%F %T][`who -u am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g'`][`whoami`]"`

- 用户管理
    ```bash
    # 创建了一个用户 luffy，其中 -d 和 -m 选项用来为登录名 luffy 产生一个主目录 /home/luffy
    useradd -d /home/luffy -m luffy

    # 修改用户的密码
    passwd luffy

    # 删除用户 luffy 在系统文件（主要是 /etc/passwd，/etc/shadow，/etc/group 等）中的记录，同时 '-r' 删除用户的主目录 /home/luffy
    userdel -r luffy

    # 此命令将用户 luffy 的登录 Shell 修改为 bash，主目录改为 /home/luffy，用户组改为 developer
    usermod -s /bin/bash -d /home/luffy -g developer luffy

    # 查看用户 luffy 密码有效期
    chage -l luffy
    # 设置用户有效期无限
    chage -M 99999 luffy

    # 查看用户 id、组 id，其中 gid（只有一个）为当前工作组 id，groups（可能为多个）为用户所有组）
    id
    id luffy
    # 查询用户所在组
    groups
    groups luffy
    ＃ （如果用户有多个附加组）切换当前用户的工作组为 group2
    newgrp group2
    ＃ 修改用户所属组（主组）
    usermod -g group1 luffy
    ＃ 修改用户所属组（附加组）
    usermod -G group1,group2 luffy
    ```

- 赋予用户 root 权限
    - 方法 1  
        修改 /etc/sudoers 文件，添加用户 sa 信息  
        `sa    ALL=(ALL)    ALL`  
        保存退出，则可以用 sa 帐号登录，然后用命令 su - ，即可获得 root 权限
    - 方法 2  
        修改 /etc/sudoers 文件，添加用户组 admin 信息  
        `%admin  ALL=(ALL)   ALL`  
        保存退出  
        修改用户 sa，使其属于 admin 组  
        `usermod -g admin sa`  
        则可以用 sa 帐号登录，然后用命令 `su` 或 `su -` 或 `su - root`，即可获得 root 权限
    - 方法 3  
        修改 /etc/passwd 文件，找到用户 sa 所在行，把用户 ID 修改为 0  
        如将  
        `sa:x:500:500:sa:/home/sa:/bin/bash`  
        修改为  
        `sa:x:0:500:sa:/home/sa:/bin/bash`  
        保存退出，则可以用 sa 帐号登录，即可获得 root 权限

- 查看磁盘空间占用情况  
    `df`  
    [查看内存占用情况](http://www.cnblogs.com/yinzhengjie/p/6489374.html)（默认单位为 KB，即等价于 `free -k`）  
    `free`
    > 以 MB 为单位显示内存使用情况  
    > `free -m`  
    > 不显示包含 "-/+ buffers/cache" 的这一列，即不显示缓冲区调节列  
    > `free -t -o -m -s 1`

    实时显示系统中各个进程的资源占用状况  
    `top`  
    查看网络设备（网卡、ip、子网掩码等）信息  
    `ifconfig`  
    查看路由表（默认网关等）信息  
    `route`  
    显示系统中所有 PCI 总线设备或连接到该总线上的所有设备  
    （Host bridge 主板芯片，VGA compatible controller 显卡，Audio device 音频设备，PCI bridge 接口插槽，USB Controller USB 控制器，Ethernet controller 网卡）  
    `lspci`  
    查看当前时间  
    `date`  
    格式化输出当前时间，如20180331205217（可用于生成时间戳）  
    `date +%Y%m%d%H%M%S`  
    ```bash
    # https://blog.csdn.net/myweishanli/article/details/26172887
    
    date '+%Y-%m-%d %H:%M:%S' # 2018-06-20 13:17:23

    date '+%Y-%m-%d %H:%M:%S' -d 'next monday' # 下周一的日期
    date '+%Y-%m-%d %H:%M:%S' -d 'next-day' # 明天的日期
    date '+%Y-%m-%d %H:%M:%S' -d 'tomorrow' # 明天的日期
    date '+%Y-%m-%d %H:%M:%S' -d 'last-day' # 昨天的日期
    date '+%Y-%m-%d %H:%M:%S' -d 'yesterday' # 昨天的日期

    date '+%Y-%m-%d %H:%M:%S' -d '30 days ago' # 30天前的日期
    date '+%Y-%m-%d %H:%M:%S' -d '-100 days' # 100天以前的日期
    date '+%Y-%m-%d %H:%M:%S' -d '3 months' # 3个月后的日期
    ```
    显示系统名、节点名称、操作系统的发行版号、操作系统版本、运行系统的机器 ID 号  
    `uname -a`  
    显示当前的各种用户进程限制  
    `ulimit -a`  
    每两秒显示一次系统资源信息（CPU、内存占用）  
    `vmstat 2`  
    每两秒显示一次系统资源信息（CPU、内存占用），一共显示 100 次  
    `vmstat 2 100`  
- 查看 $PID 进程占用的文件句柄数  
    `ls -l /proc/$pid/fd | wc -l`  
    ~~`ls /proc/$pid/fd | sed -n "$="`~~  
    ~~`lsof -n | awk '{ print $2 }' | sort | uniq -c | sort -nr | more | grep $PID`~~  
    ~~`lsof -p $pid | wc -l`~~
    > [lsof 的错误使用场景和查看打开文件数的正确方法](https://www.jianshu.com/p/407c2baef92e)

- [线程和 wait、sleep 命令](http://www.cnblogs.com/xiaofeiIDO/p/6170199.html)
    ```bash
    sleep 10 # 等待 10 秒，再继续下一操作

    sleep 10 & # 当前 shell 不等待，后台子 shell 等待

    # wait 是用来阻塞当前进程的执行，直至指定的子进程执行结束后，才继续执行。
    wait # 等待 wait 所在 bash 上的所有子进程的执行结束，如上一句的 `sleep 10 &`
    ```

- 进程、端口查询
    - 查询当前系统所有进程信息  
        `ps -ef`
    - 杀死进程
        ```shell
        # （推荐使用）杀死 4044 进程，"-15" 表示 SIGTERM 信号编码
        kill -15 4044

        # （不推荐使用）强制杀死 4044 进程，"-9" 表示 SIGKILL 信号编码
        # kill -9 4044
        ```
    - 已知端口号 8080，查询占用该端口号的进程信息（进程号）  
        `netstat -anp | grep ":8080"`  
        `lsof -i :8080`  
        `lsof -i tcp | grep ":8080"`
        > netstat 中查看 tcp 状态
        > - `LISTENING`: 端口处于处于侦听状态
        > - `ESTABLISHED`：已建立连接，即两台机器正在通信
        > - `CLOSE_WAIT`：对方主动关闭连接或者网络异常导致连接中断，这时我方的状态会变成 CLOSE_WAIT 此时我方需要调用 close() 来使得连接正确关闭
        > - `TIME_WAIT`：我方主动调用 close() 断开连接，收到对方确认后状态变为 TIME_WAIT
        > - `SYN_SENT`：表示请求连接（当你要访问其它的计算机的服务时首先要发个同步信号给该端口，此时状态为 SYN_SENT），如果连接成功了就变为 ESTABLISHED
    - 已知进程号 4044，查询该进程的具体信息  
        `ps 4044`  
        查看该进程启动的完整命令行  
        `ps eho command -p 4044`  
        查看该进程启动时候所在的目录  
        `readlink /proc/4044/cwd`  
        查看该进程启动时的完整环境变量  
        `strings -f /proc/4044/environ | cut -f2 -d ' '`
    - 已知进程号 4044，查询该进程所打开的网络连接  
        `netstat -pan | grep 4044`
    - 查询所有 tomcat 进程并强制杀掉进程  
        `ps -ef | grep tomcat | grep -v grep | awk '{print $2}' | xargs kill -s 9`

- 临时修改网卡 eth0 上第一个 ip 地址  
    `ifconfig eth0 192.168.1.104`  
    [修改网卡 eth0 上第二个 ip 地址](http://blog.csdn.net/hzhsan/article/details/44677867)  
    `ifconfig eth0:1 192.168.2.101`
    配置当前主机的 ip 为 192.168.1.101，子网掩码为 255.255.255.0  
    `ifconfig eth0 192.168.1.101 netmask 255.255.255.0 up`  
    开启/关闭网卡混杂模式  
    `ifconfig eth0 promisc`  
    删除 eth0 上的默认路由  
    `route del default eth0`  
    添加默认路由  
    `route add default gw 192.168.1.1`  
    配置当前主机的默认网关为 192.168.1.1  
    `route add default gw 192.168.1.1`
- 查看当前主机的 mac 地址  
    `ifconfig -a | grep -i hw`

- OpenSSL
    - 使用 OpenSSL 生成自签名证书及私钥(无 PEM pass phrase)  
        `openssl req -nodes -new -x509 -keyout key.pem -out cert.crt`  
        <font color=grey>参数 "-nodes" 表示生成的私钥不需要加密，即无需输入无 PEM pass phrase</font>
    - [创建自签名的CA证书，及使用CA签发证书](http://www.cnblogs.com/xcloudbiz/articles/5523815.html) 
        1. 创建自签名根证书密钥 ca.key、证书 ca.crt  
            `openssl req -new -days 3650 -x509 -keyout ca.key -out ca.crt -subj "/C=CN/ST=JiangSu/L=NanJing/O=MyCompany Corp./emailAddress=admin@mycompany.com"`
        2. 创建服务器证书密钥 server.key  
            `openssl genrsa -out server.key 2048`
        3. 创建服务器证书的申请文件 server.csr(Certificate Signing Request)  
            `openssl req -new -key server.key -out server.csr -subj "/C=CN/ST=JiangSu/L=NanJing/O=MyCompany Corp./CN=127.0.0.1/emailAddress=admin@mycompany.com"`  
            <font color=grey>其中 CN(Common Name) 为 "127.0.0.1" (服务器主机名)，即 web 服务访问的域名(如 "www.microsoft.com")，若填写不正确，浏览器会报告证书无效</font>
        4. 使用 CA 证书签发自当前日期起有效期为期两年的服务器证书 server.crt  
            `openssl x509 -req -days 365 -sha256 -extensions v3_req -CA ca.crt -CAkey ca.key -CAserial ca.srl -CAcreateserial -in server.csr -out server.crt`  
            <font color=grey>参数 "-sha256" 表示使用 sha256 签名算法，参数 "-extensions v3_req" 表示需要生成 v3 版的证书</font>
    - 使用 OpenSSL 生成私钥文件 key.pem (2048位)  
        `openssl genrsa -out key.pem 2048`  
        根据私钥文件生成自签名证书 cert.pem  
        `openssl req -new -x509 -key key.pem -out cert.pem`
    - 将证书文件 server.crt 和证书密钥文件 server.key 合并成证书安装包 server.pfx  
        `openssl pkcs12 -export -in server.crt -inkey server.key -out server.pfx`
    - 对字符串进行编码解码、加密解密
        - 对字符串 "hello, world" 进行 base64 编码  
            `openssl base64 <<< "hello, world"`  
        - 对字符串 "hello, world" 进行 aes 加密，使用密钥 123，输出结果以 base64 编码格式给出  
            `echo hello, world | openssl aes-128-cbc -k 123 -base64`  
            对以上结果进行解密处理  
            `echo U2FsdGVkX18ynIbzARm15nG/JA2dhN4mtiotwD7jt4g= | openssl aes-128-cbc -d -k 123 -base64`
        - 对字符串 "hello, world" 进行 aes-128-cbc 加密，使用密钥 "123"、初始化向量 "123"  
            `openssl aes-128-cbc -iv "123" -k "123" <<< "hello, world"`  
            对字符串 "hello, world" 进行 aes-128-cbc 加密，输出结果以 base64 编码格式给出（需要手动输入密钥，如 "123"）  
            `openssl aes-128-cbc <<< "hello, world" | openssl base64`  
            对以上结果进行解密处理（需要手动输入密钥，与加密密钥一致，如 "123"）  
            `openssl base64 -d <<< "U2FsdGVkX1+oPWAlmoa6dn4c5ePWw9zG8MaZ0YCblq4=" | openssl aes-128-cbc -d`
    - [使用 OpenSSL 编码解码文件](http://www.cnblogs.com/AloneSword/p/3480115.html)  
        将文件 hello.txt 编码为 base64 格式，并输出到文件 hello.base64.txt 中  
        `openssl base64 -in hello.txt -out hello.base64.txt`  
        使用 base64 解码文件 hello.base64.txt  
        ~~`openssl base64 -d -in hello.base64.txt -out hello.txt`~~  
        `openssl base64 -d <<< "U2FsdGVkX1+oPWAlmoa6dn4c5ePWw9zG8MaZ0YCblq4=" > hello.txt`  
        对文件 hello.txt 进行 md5 摘要计算  
        `openssl md5 -in hello.txt`
    - 使用 AES 算法加密文件  
        使用 aes-256-ecb 加密算法加密文件 a.txt 并输出到文件 a.ecb.txt 中  
        `openssl enc -aes-256-ecb -in a.txt -out a.ecb.txt -e`  
        使用 aes-256-ecb 加密算法解密文件 a.ecb.txt 并输出到文件 a.ecb.d.txt 中  
        `openssl enc -aes-256-ecb -in a.ecb.txt -out a.ecb.d.txt -d`  
        使用 aes-256-cbc 加密算法加密文件 a.bmp 并输出到文件 a.cbc.bmp 中  
        `openssl enc -aes-256-cbc -in a.bmp -out a.cbc.bmp`
    - 使用 RSA 算法加密解密文件
        1. 生成密钥的长度为 1024 位的私钥文件 pri_key.pem  
            `openssl genrsa -out pri_key.pem 1024`
        2. 从私钥文件 pri_key.pem 中提取公钥文件 pub_key.pem  
            `openssl rsa -in pri_key.pem -pubout -out pub_key.pem`
        3. 使用公钥文件 pub_key.pem 加密文件 a.txt  
            `openssl rsautl -encrypt -in a.txt -inkey pub_key.pem -pubin -out a.e.txt`
        4. 使用密钥文件 pri_key.pem 解密文件 a.e.txt  
            `openssl rsautl -decrypt -in a.e.txt -inkey pri_key.pem -out a.d.txt`
    - 证书转换、导入导出  
        - PKCS12 转换为 PEM
            - 导出私钥  
                `openssl pkcs12 -in client.p12 -passin pass:123456 -passout pass:123456 -nocerts -out client.pri.pem`
            - 导出证书  
                `openssl pkcs12 -in client.p12 -passin pass:123456 -nokeys -out client.pub.pem`
        - PEM 转换为 PKCS12  
            `openssl pkcs12 -export -in client.pub.pem -inkey client.pri.pem -out client.p12 -name client -passin pass:123456 -passout pass:123456`
        - JKS 转换为 PKCS12  
            `keytool -importkeystore -srcstoretype JKS -srckeystore client.jks -srcstorepass 123456 -srcalias client -srckeypass 123456 -deststoretype PKCS12 -destkeystore client.p12 -deststorepass 123456 -destalias client -destkeypass 123456 -noprompt`
        - PKCS12 导出至 JKS  
            `keytool -importkeystore -srckeystore client.p12 -srcstoretype PKCS12 -srcstorepass 123456 -alias client -deststorepass 123456 -destkeypass 123456 -destkeystore client.jks`
        > - [使用 keytool 生成证书](tomcat.md#keytool)
    - 使用 openssl 生成 alipay 商户公私钥文件（merchant_private_key_pkcs8.pem、alipay_public_key.pem）
        1. 生成（商户）私钥  
            `openssl genrsa -out merchant_private_key.pem 2048`
        2. Java 开发者需要将私钥转换成 PKCS8 格式  
            `openssl pkcs8 -topk8 -inform PEM -in merchant_private_key.pem -outform PEM -nocrypt -out merchant_private_key_pkcs8.pem`
        3. 生成（应用）公钥  
            `openssl rsa -in merchant_private_key.pem -pubout -out app_public_key.pem`
    - 使用 openssl 查看网站证书信息  
        `openssl s_client -showcerts -connect www.baidu.com:443 < /dev/null`
    - [使用 openssl 生成 ca 证书、签发中间 ca 证书、签发证书](https://jamielinux.com/docs/openssl-certificate-authority/)
        ```bash
        # root ca certificate
        openssl genrsa -out ca.key 4096
        openssl req -new -x509 -days 7300 -key ca.key -subj "/C=CN/ST=JS/L=NJ/O=Sunke, Inc./CN=Sunke Root CA" -out ca.crt

        # intermediate ca certificate
        openssl req -newkey rsa:2048 -nodes -keyout intermediate.key -subj "/C=CN/ST=JS/L=NJ/O=Sunke, Inc./CN=Sunke Intermediate CA" -out intermediate.csr
        openssl x509 -sha256 -req -extfile <(printf "basicConstraints=critical,CA:true") -days 3650 -in intermediate.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out intermediate.crt

        # server certificate (for multiple domains or ips, change subjectAltName to: DNS:example.com,DNS:www.example.com,IP:192.168.1.100)
        openssl req -newkey rsa:2048 -nodes -keyout server.key -subj "/C=CN/ST=JS/L=NJ/O=Sunke, Inc./CN=www.example.com" -out server.csr
        #openssl x509 -sha256 -req -extfile <(printf "subjectAltName=DNS:www.example.com") -days 365 -in server.csr -CA intermediate.crt -CAkey intermediate.key -CAcreateserial -out server.crt
        openssl x509 -sha256 -req -extfile <(printf "subjectAltName=DNS:www.example.com,DNS:localhost,IP:127.0.0.1,IP:192.168.1.101") -days 365 -in server.csr -CA intermediate.crt -CAkey intermediate.key -CAcreateserial -out server.crt # visit with url https://www.example.com or https://localhost or https://127.0.0.1 or https://192.168.1.101
        cat server.crt intermediate.crt ca.crt > server.b.crt
        
        # deploy server.key server.b.crt
        ```
    - RSA 秘钥格式 PKCS#1（RSA PRIVATE KEY）与 PKCS#8（PRIVATE KEY）互转
        ```bash
        # 生成 PKCS#1 私钥
        openssl genrsa -out private_pkcs1.pem 2048
        # 生成 PKCS#8 私钥
        openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out private_pkcs8.pem

        # PKCS#1 私钥转为 PKCS#8 私钥
        openssl pkcs8 -topk8 -inform PEM -in private_pkcs1.pem -outform pem -nocrypt -out private_pkcs8.pem
        # PKCS#8 私钥转为 PKCS#1 私钥
        openssl rsa -in private_pkcs8.pem -out private_pkcs1.pem

        # PKCS#1 私钥生成 PKCS#8 公钥
        openssl rsa -in private_pkcs1.pem -pubout -out public_pkcs8.pem
        # PKCS#8 私钥生成 PKCS#8 公钥
        openssl rsa -in private_pkcs8.pem -pubout -out public_pkcs8.pem
        openssl pkey -in private_pkcs8.pem -pubout -out public_pkcs8.pem
        
        # PKCS#8 公钥转为 PKCS#1 公钥（在私钥不使用密码的情况下，公钥的 PKCS#1 和 PKCS#8 格式内容是一致的）
        openssl rsa -pubin -in public_pkcs8.pem -RSAPublicKey_out -out public_pkcs1.pem
        # PKCS#1 公钥转为 PKCS#8 公钥
        openssl rsa -RSAPublicKey_in -in public_pkcs1.pem -RSAPublicKey_out -pubout -out public_pkcs8.pem
        ```
    - 公钥/私钥格式处理，单行字符串转换为多行  
        如单行公钥字符串: `MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDYwzcpD6YQQu3ddK147cjBZoqRupP56FJIH2Rr8ShLawh1fbmHRHbrtfKsPE7jJS6UXEI0LncqUydbVKnOt9q1Dt+W8mEXZxzArmv5NuaHI7+Rx2ehphk913bR531CPm/+nUfLQN/9JT+2MVLMRMQ5P2b3fH+8h+ndvKhHQGetXwIDAQAB`  
        依次按每 64 个字符末尾加换行符 `\n`，首尾分别加一行 `-----BEGIN PUBLIC KEY-----` 和 `-----END PUBLIC KEY-----`，最终结果如下
        ```
        -----BEGIN PUBLIC KEY-----
        MIGfMA0GCSqGSIb3DQEBAQUAA4GNA6DCBiQKBgQCwbR2M2efjbuttNoeCjE19ZBqA
        z3Q2BWxdNeQfC7v3434344OI54JB28H5DX1H44CFRgodYn6oLzI4zh3kG2XqwAOGf4
        FOZlkD1nkDhlY8od4mUJvEr1XwSz+V2W0aNyONBe29NdWScMQlTxhxf15NaHHSM1l
        RkiOOOhNXgLLXOXF4QID3AQAB
        -----END PUBLIC KEY-----
        ```
    - 导出网站的公钥证书  
        `openssl s_client -showcerts -connect www.baidu.com:443 < /dev/null 2> /dev/null | openssl x509 -outform PEM > www.baidu.com.pem`
    - 签名
        ```bash
        # SHA256 + RSA/PSS 签名
        openssl dgst -sha256 -sign private.pem -sigopt rsa_padding_mode:pss -out sign.bin 文件名.txt

        # SHA256 + RSA/PSS 验签
        openssl dgst -sha256 -verify public.pem -sigopt rsa_padding_mode:pss -signature sign.bin 文件名.txt # 输出 Verified OK 代表数字签名校验成功
        ```
        ```bash
        # 计算 SHA256 哈希校验和
        openssl dgst -sha256 -binary -out sha256sum.bin 文件名.txt

        # RSA/PSS 签名
        openssl pkeyutl \
            -inkey private.pem -pkeyopt rsa_padding_mode:pss -pkeyopt digest:sha256 \
            -sign -in sha256sum.bin -out sign.bin

        # RSA/PSS 验签
        openssl pkeyutl \
            -pkeyopt rsa_padding_mode:pss -pkeyopt digest:sha256 \
            -pubin -inkey public.pem \
            -verify -in sha256sum.bin -sigfile sign.bin
        # 输出 Signature Verified Successfully 代表数字签名校验成功
        ```
        ```bash
        # SHA1 + RSA 签名
        openssl dgst -sha1 -sign private.pem -out sign.bin 文件名.txt

        # SHA1 + RSA 验签
        openssl dgst -sha1 -verify public.pem -signature sign.bin 文件名.txt
        ```
        ```bash
        # 计算文件 SHA256 哈希值
        openssl dgst -sha256 文件名.txt
        ```

- hmac sha256 签名文件
    ```bash
    hmac256 \
        "" \ # key
        hello.txt # 待签名的文件
    ```

- 实现 /etc/shadow 中的密钥算法（详细请参考 `man crypt`）加密密码
    - OpenSSL passwd 只支持基于 MD5 的加密算法（"xxxxxxxx" 为盐值，"password" 为密码明文，其结果应为 "$1$xxxxxxxx$UYCIxa628.9qXjpQCjM4a."
）  
        `openssl passwd -1 -salt 'xxxxxxxx' 'password'`
    - 使用 python 的 crypt 模块可以使用所有加密算法
        ```python
        import crypt
        assert crypt.crypt('root', '$6$Sguvrc9/') == '$6$Sguvrc9/$657bKmUBEpGUplcKgDhd3UOTy7r71/da6upxGkf1jvX4j22flk8av8oM8IjL2ztxk5xQDKmV3M2Ea5QfcA/6Y0'
        # '$6$Sguvrc9/' 表示使用基于 SHA-512 的加密算法和盐值 'Sguvrc9/'
        ```

- iptables 防火墙
    - 创建规则，将所有通过本机的源地址在 10.0.1.* 网段（网卡接口 1，如 eth0）的数据包转发到 10.0.2.* 网段（网卡接口 2，如 eth1）上，可用作网关不同网络之间的转发  
        `iptables -I FORWARD -s 10.0.1.0/24 -d 10.0.2.0/24 -j ACCEPT`  
        参数 -D 表示删除该规则  
        `iptables -D FORWARD -s 10.0.1.0/24 -d 10.0.2.0/24 -j ACCEPT`
    - 删除规则
        1. 列出 INPUT 链所有规则  
            `iptables -L INPUT --line-numbers`
        2. 删除 INPUT 链规则行号为 3 的规则  
            `iptables -D INPUT 3`
    - 端口转发，将 422 端口的包转发到 22 端口  
        `iptables -t nat -A PREROUTING -p tcp -d 192.168.1.37 --dport 422 -j DNAT --to 192.168.1.37:22`

- 修改主机名，如修改为 luffy
    ```bash
    hostname luffy
    cat > /etc/hostname < EOF
    luffy
    EOF
    ```

- 使用 unshare 命令模拟 docker 实现各种资源隔离（基于 Linux 的 Namespace 技术）
    ```bash
    # 创建一个新的 namespace，该 namespace 的 uts、mount、pid 与全局资源隔离
    unshare \
        --uts \ # 隔离主机名和域名
        --mount \ # 隔离挂载点
        --pid \ # 隔离进程 ID（不继承父进程 pid 命名空间，即在子进程内执行 ps，无法看到父进程原有的进程）
        --fork /bin/bash # fork 一个新的子进程，在子进程里执行 /bin/bash
    ```
    > Linux 内核提供了 8 种类型的 Namespace。在这些独立的 Namespace 中，资源互不影响，相互隔离
    > - Mount/mnt 隔离挂载点
    > - Process ID/pid 隔离进程 ID
    > - Network/net 隔离网络设备，端口号等
    > - Interprocess Communication/ipc 隔离 System V IPC 和 POSIX message queues
    > - UTS Namespace/uts 隔离主机名和域名
    > - User Namespace/user 隔离用户和用户组
    > - Control group Namespace/cgroup 隔离 Cgroups 根目录 (4.6 版本加入)
    > - Time Namespace/time 隔离系统时间 (5.6 版本加入)

## 网络通讯

- 从网络上下载文件  
    `wget http://xxx/xxx.tar.gz`  
    断点续传  
    `wget -c http://xxx/xxx.tar.gz`  
    从 https 站点下载文件  
    `wget https://102.alibaba.com/downloadFile.do?file=1516614343703/AliDouble11.pdf --no-check-certificate`

- [使用 curl](http://www.cnblogs.com/gbyukg/p/3326825.html) 下载单个文件，默认将输出打印到标准输出中(stdout)中  
    `curl http://www.centos.org`  
    `curl -k https://www.baidu.com`  
    参数 "-s" 表示不显示下载进度信息  
    `curl -s http://www.baidu.com`  
    将文件下载到本地并命名为 mygettext.html（参数 "-o" 表示将文件保存为命令行中指定的文件名的文件中）  
    `curl -o mygettext.html http://www.gnu.org/software/gettext/manual/gettext.html`  
    将文件保存到本地并命名为 gettext.html（参数 "-O" 表示使用 URL 中默认的文件名保存文件到本地）  
    `curl -O http://www.gnu.org/software/gettext/manual/gettext.html`  
    “--retry 3” 表示失败重试次数为 3 次，“--retry-delay 3” 表示失败重试间隔时间为 3 秒  
    `curl -v -s -k -O http://www.gnu.org/software/gettext/manual/gettext.html --retry 3 --retry-delay 3`  
    通过添加 -C 选项继续对该文件进行下载，已经下载过的文件不会被重新下载（即断点续传）  
    `curl -C - -O http://www.gnu.org/software/gettext/manual/gettext.html`  
    通过代理服务器 http://proxy.huacai.com:8080（其中认证方式为 NTLM，用户名为 "s00123456"，密码为 "huacai@123"）访问 http://www.baidu.com  
    `curl --proxy-ntlm --proxy-user s00123456:huacai@123 --proxy http://proxy.huacai.com:8080 http://www.baidu.com`  
    `curl -k 'https://139.159.158.150:8443/' --proxy 'http://10.75.203.36:5888'`  
    参数 "-L" 表示自动跟踪 URL 重定向跳转  
    `curl -k -O -s -L https://github.com/btraceio/btrace/releases/download/v1.3.11.1/btrace-bin-1.3.11.1.zip`  
    使用环境变量设置代理服务器  
    ```bash
    export http_proxy=http://10.75.203.36:5888
    export https_proxy=http://10.75.203.36:5888

    curl -k 'https://139.159.158.150:8443/'
    ```
    上传 json 文件  
    `curl http://localhost:8983/xxx --data-binary @/temp/userinfo.json -H 'Content-type:application/json'`  
    `curl -k https://xxx -F "files=@data.zip" -H "filename:data.zip"`  
    上传 jpeg 文件  
    `curl -F "pic=@1.jpg;filename=1.jpg;type=image/jpeg" -F "username=zhangsan" -F "type=2" http://127.0.0.1:8080/upload`  
    使用 post 方法提交 json 格式数据  
    ~~`curl -X POST -H 'Content-Type: application/json' 'http://localhost:8983/xxx' --data-binary '{ "id": "1", "name": "zhangsan" }'`~~  
    `curl -X POST -H 'Content-Type: application/json' http://localhost:8983/xxx -d '{ "id": "1", "name": "zhangsan" }'`  
    `curl -X POST -H 'Content-Type: application/json' -H 'userid: 0012345' http://localhost:8983/xxx -d '{ "id": "1", "name": "zhangsan" }'`  
    自定义 Locale  
    `curl -H 'Accept-Language:zh_CN' 'http://localhost:8983/xxx'`  
    `curl -H 'Accept-Language:en-us;q=0.8,zh-cn' http://localhost:8983/xxx`
    显示消息头和消息体  
    `curl -i http://www.baidu.com`  
    客户端证书认证  
    `curl --cacert ./ca.crt --cert ./client.crt --key ./client.key https://127.0.0.1:8443`

- 抓包并写入到文件 xxx.cap 中  
    `tcpdump -i any port xxxx -s 0 -w xxx.cap`  
    抓包（直接显示 ip，不做 dns 翻译）  
    `tcpdump -n`  
    监听本地 80 端口，抓包并写入到文件 http.cap 中（使用参数 "-s0"，指定抓包长度，防止解析时出现 "Packet size limited during capture: HTTP truncated"）  
    `tcpdump port 80 -s0 -w http.cap`  
    抓包并写入到文件 file.pcap 中（抓取网卡 eth0 的 tcp 协议包，目的主机为 192.168.1.100:3306）  
    `tcpdump -n -i eth0 tcp and host 192.168.1.100 and port 3306 -w file.pcap`  
    抓包并写入到文件 a.cap 中  
    `tcpdump -n -w a.cap`  
    从文件 a.cap 中读取抓包数据  
    `tcpdump -nX -r a.cap`  
    抓包所有与本机 1935 端口通信的数据包，并记录到 rtmp.log 中  
    `tcpdump -nX port 1935 > rtmp.log`  
    参数 -i 指定监听的网卡，如 eth、lo（可用于抓取本地包，即 127.0.0.1）  
    `tcpdump -i lo port 8080`

- 使用文字式网页浏览器打开网页 archive.ubuntu.com  
    `w3m archive.ubuntu.com`

- 发送消息/广播  
    `echo "hello, world" | wall`  
    向指定用户 "root"、指定终端 "pts/9" 发送消息  
    `write root pts/9`

- 连接 ftp  
    `sftp -o Port=20022 username@192.168.1.1`

- 使用 sqlplus 连接 oracle 数据库  
    `sqlplus sysdb/sys_12dagd@10.137.15.11:1526/mdspdb`   

- 使用 netcat 监听本地 3452 端口，并打印详细信息（连接发起者 ip 和端口号）  
    `nc -lvnp 3452`
- 使用 netcat 监听 TCP 的 3452 端口，并将镜像写入文件 cyqimage.dd  
    `nc -l -p 3452 > myimage.dd`
- 使用 netcat 循环监听本地 80 端口，并响应 index.html 内容  
    `while true; do nc -l -p 80 < index.html; done`  
    使用 netcat 监听本地 UDP 协议的 8080 端口，其中参数 "-w1" 表示只接收一行消息，参数 "-u" 表示使用 UDP 协议  
    `nc -u -w1 -l -p 8080`  
    使用 netcat 向本机的 8080 端口发送（一行）UDP 消息 "hello"  
    `echo "hello" | nc -u -w1 127.0.0.1 8080`  
    向本机的 8080 端口发送 UDP 消息 "hello"  
    `echo "hello" > /dev/udp/localhost/8080`
- [使用 netcat 创建一个 TCP/HTTP 代理服务器并监听请求和响应：监听本地 80 端口并转发请求至本地 8080 端口（如 web 服务器），请求报文保存至文件 in 中，响应报文保存至文件 out 中](http://www.linux-france.org/~mdecore/linux/doc/memo2/node168.html)
    ```bash
    mknod backpipe p
    nc -l -p 80 < backpipe | tee -a in | nc localhost 8080 | tee -a out > backpipe
    ```

- 使用 netcat 或 telnet 建立 TCP 连接，建立连接后即可直接相互发送消息
    ```bash
    # 服务端监听本地端口，如 8000
    nc -l -p 8000
    
    # 方法一：客户端使用 telnet 连接服务端
    telnet 127.0.0.1 8000
    # 方法二：客户端使用 netcat 连接服务端
    nc 127.0.0.1 8000
    ```

- 查询域名信息  
    `whois www.baidu.com`  
    查询域名信息，指定端口查询  
    `whois -p 80 www.baidu.com`

- 查询本机公网 ip  
    `curl cip.cc`  
    `curl ip.sb`

## 其他命令

- 显示当前 shell 进程号  
    `echo $$`

- 在命令结尾处添加 `&`，表示在后台运行命令（即进程在后台运行，Control + C 不会结束该进程，关闭 bash 时结束该进程）  
    `python server.py &`  
    在命令的开头加一个 `nohup`，表示忽略所有的挂断信号，如果当前 bash 关闭，则当前进程会挂载到 init 进程下，成为其子进程，这样即使退出当前用户，命令仍然在后台运行
    ```bash
    # nohup python server.py &

    # 使用追加模式（">>"），可方便动态清空 out 日志内容（`cat /dev/null > srv.out`）。否则无非释放 out 文件磁盘空间，见 https://zhidao.baidu.com/question/1821419994894225548.html
    nohup python server.py >> srv.out 2>&1 &
    ```
    - 查看后台执行的程序  
        `jobs`  
        根据 jobs 命令查询的结果，唤起后台程序，如唤起序号为 2 的后台程序  
        `fg 2`

- 编译、链接
    - 自动化编译，需先切换目录至 Makefile 所在目录（即项目根目录），再执行以下命令  
        `make`
    - [使用gcc/g++编译、链接](http://blog.csdn.net/yinjiabin/article/details/7731817)
        - 编译、链接当前目录下所有 "*.c" 文件，并使用用宏定义 "DEBUG" 进行条件编译，生成可执行文件 "main"  
            `gcc *.c -DDEBUG -o main`
        - 编译&链接为可执行文件
            - 编译、链接单个源文件 "main.c" 为可执行文件，默认生成可执行文件 "a"  
                `gcc main.c`
            - 编译、链接单个源文件 "main.c" 为可执行文件，指定生成可执行文件 "main"  
                `gcc main.c -o main`  
                "-static" 表示静态编译（即将动态库的函数和所依赖的任何的东西，都编译进本程序中，因此编译好后，文件会非常大，但是运行时不需要依赖任何动态库）  
                `gcc -static main.c -o main`
            - 编译、链接多个源文件 "main.c"、"test.c"为可执行文件 "main"  
                `gcc main.c test.c -o main`
        - 编译、链接、调试
            1. 编译、链接  
                `gcc -g main.c -o main`
            2. 调试运行  
                `gdb main`
        - 编译、链接、运行程序
            1. 编译  
                `gcc -c main.c -o main.o`
            2. 链接  
                `gcc main.o -o main`  
                ~~将多个目标文件("main.o"、"app.o")链接为可执行文件 "main"  
                `gcc main.o app.o -o main`~~
            3. 运行  
                `./main`
        - [编译链接动态库、静态库并调用](http://blog.sina.com.cn/s/blog_69e96b3701010881.html)
            - 编译 "func.c" 成动态链接库文件 "libfunc.so"  
                `gcc -fPIC -shared func.c -o libfunc.so`  
                链接 "func1.o"、"func2.o" 成动态链接库文件 "libfunc.so"  
                `gcc -o libfunc.so -fPIC -shared func1.o func2.o`
            - 链接 "func1.o"、"func2.o" 成静态链接库文件 "libfunc.a"  
                `ar rc libfunc.a func1.o func2.o`
            - 编译链接 "main.c"，其中 "-lfunc" 中的 "func" 是指寻找当前目录下动态连接库 "libfunc.so"（隐含的命名规则：在给出的名字前面加上 lib，后面加上.so 来确定库的名称）  
                `gcc -L. -lfunc main.c -o main`
    - 其他  
        生成汇编代码 hello.s  
        `gcc -S hello.c`  
        将汇编代码 hello.s 编译成可执行文件 hello  
        `gcc hello.s -o hello`

- 使用 script 录制终端会话中所有输入输出结果，并保存时序信息至文件 "record.timing"，保存命令信息至文件 "record.script"（命令行提示 "Script started, file is record.script" 后开始录制，输入 "exit" 或键入 "Ctrl + D" 后提示 "Script done, file is record.script" 结束录制）  
    `script -t 2> record.timing record.script`  
    使用 scriptreplay 重放终端会话中所有输入输出结果  
    `scriptreplay record.timing record.script`

- 使用 qemu 启动一个虚拟机，内存为 128 mb，镜像为 dsl.iso  
    `qemu -L . -m 128 -cdrom dsl.iso -enable-audio -localtime -user-net`  
    > 使用 qemu-img 创建一个虚拟磁盘  
    > `qemu-img create -f vhdx vdisk.vhdx 1G`

- 执行 shell 脚本
    - fork 方式（由当前进程创建一个子进程）  
        `./hello.sh`
    - source 方式（不另外创建子进程，而是在当前的的 Shell 环境中执行）（可用于在脚本中集中设置环境变量，如引用 hello.sh 中的定义的环境变量）  
        `source ./hello.sh`  
        > 该命令通常用命令 "." 来替代  
        > 如：`source .bash_rc` 与 `. .bash_rc` 是等效的

    - exec 方式（不另外创建子进程，但是会终止当前的 shell 执行）  
        `exec ./hello.sh`

- 使用 python 正则查找所有 /etc/sudoers 中引用的 shell 脚本文件，并计算各脚本文件的 md5 值  
    `python -c "import re; print ' '.join(re.compile('/[^,\s]+\.sh').findall(open('/etc/sudoers').read())).replace('\'', '');" | xargs md5sum`

- 执行命令  
    `eval "echo hello"`  
    `exec echo hello` # 只能在脚本中使用

- 生成随机数
    - 随机生成 10 个字符的字符串  
        `tr -dc 'A-Za-z0-9!?%=' < /dev/urandom | head -c 10`  
        `tr -cd '[:alnum:]' < /dev/urandom | fold -w10 | head -n1`  
        <font color=grey># [:alnum:] 表示匹配当前归类中的数字、大写和小写字母字符</font>
    - 使用 OpenSSL 随机生成 10 个字节，并将其转换为十六进制数  
        `openssl rand -hex 10`  
        随机生成 8 个字节，并将其转换为 base64 编码  
        `openssl rand -base64 8`  

- 格式化打印到控制台  
    `printf "hello, %5s" "hello"`  
    `printf "\e[1;31;40mRed\e[m"`  
    `echo -e "\e[1;31;40mRed\e[m"`
    > 终端的字符颜色由转义序列（Escape Sequence）控制，是文本模式下的系统显示功能，与具体语言无关  
    > 通过转义序列设置终端显示属性时，可采用格式：\033[ Param {;Param;...}m 或 \e[ Param {;Param;...}m（其中，'\033['或'\e['引导转义序列，'m'表示设置属性并结束转义序列）  
    > 转义序列相关的常用参数如下（通过 man console_codes 命令可查看更多的参数描述）：
    > - 显示：0(默认)、1(粗体/高亮)、22(非粗体)、4(单条下划线)、24(无下划线)、5(闪烁)、25(无闪烁)、7(反显、翻转前景色和背景色)、27(无反显)
    > - 颜色：0(黑)、1(红)、2(绿)、3(黄)、4(蓝)、5(洋红)、6(青)、7(白)  
    >   前景色为 30+ 颜色值，如 31 表示前景色为红色；背景色为 40+ 颜色值，如 41 表示背景色为红色。
    > 
    > 如 `\e[1;31;40mRed\e[m` 或 `\033[1;31;40mRed\033[m` 表示使用粗体、前景色为红色、背景色为黑色显示字符串 Red

- 在 ~/.ssh 目录下生成当前用户的密钥 id_rsa、公钥 id_rsa.pub  
    `ssh-keygen -t rsa`  
    将公钥复制(替换或追加)到远程主机的 ~/.ssh 目录下的文件 authorized_keys，则远程主机信任当前主机用户，当前主机用户无需密码，可 ssh 直接登录远程主机  
    `ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.101`
    <!-- `ssh-agent bash`
    `ssh-add id_rsa` -->
- 使用 ssh 远程登录服务器  
    `ssh 192.168.1.100`  
    通过私钥登录 ssh 服务器  
    `ssh -i ~/.ssh/id_rsa root@192.168.1.200`  
    使用 ssh 向远程主机发送命令（设置主机信任后，可用于自动化脚本）  
    ```bash
    name=$(date +%Y%m%d%H%M%S)
    ssh root@192.168.1.200 << "EOF"
    mkdir ~/${name}
    echo 'hello, world'
    EOF
    ```
    ```bash
    # 自动登录远程主机 192.168.1.101，并登录该主机中的 docker 容器（参数 -t 表示强制伪终端分配，即使标准输入不是终端）
    ssh -t root@192.168.1.101 "docker exec -it xxxxxxxx bash"
    ssh -t root@192.168.1.101 "docker exec -it \`docker ps | grep tomcat: | awk \"{ print \\\\\$1 }\"\` /bin/bash"

    # 登录远程主机 192.168.1.101 并进入 /home 路径
    ssh -t root@192.168.1.101 "cd /home; bash"

    alias mysql='ssh -t root@192.168.1.101 mysql'

    function kubectl()
    {
        ssh -t root@192.168.1.101 "bash -ic \"kubectl $@\""
    }

    # 查看函数 kubectl 的定义
    # type kubectl
    ```
    保持连接:每60秒发送一次空包  
    `ssh -o ServerAliveInterval=60 192.168.1.100`
- 判断远程主机是否信任当前主机  
    `ssh -o NumberOfPasswordPrompts=0 -o StrictHostKeyChecking=no root@192.168.1.101 "date" && echo $?`  
    结果返回 0 则信任，否则为不信任（如 255）
- 使用 ssh 进行端口转发
    - 监听本地8001端口（并登录远程主机192.168.1.102），将监听的tcp报文发送至（远程主机192.168.1.102的）localhost的8080端口，可用于正向端口转发  
        `ssh -L 8001:localhost:8080 root@192.168.1.102`
    - （通过主机192.168.1.102的ssh服务）将本机192.168.1.101的80端口转发至主机192.168.1.103的8080端口  
        `ssh -L 192.168.1.101:80:192.168.1.103:8080 192.168.1.102`  
        ~~`ssh -L 192.168.1.101:80:192.168.1.103:8080 192.168.1.101`~~
    - （登录远程主机192.168.1.101并）监听远程主机的8001端口，将监听的tcp报文发送至本地localhost的8080端口，可用于反向端口转发  
        `ssh -R 8001:localhost:8080 192.168.1.101`
    - 监听本地8001端口，（登录远程主机192.168.1.103）并动态开启端口，将监听的tcp报文通过该端口转发至目的服务器，可用于sockets代理服务器  
        `ssh -D 8001 root@192.168.1.103`
