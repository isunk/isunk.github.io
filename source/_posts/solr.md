---
title: Solr
tags: solr
categories: solr
---

# Solr

## [Document（官方文档）](https://lucene.apache.org/solr/guide/7_5/)

## Usage

- [下载、部署与运行](http://lucene.apache.org/solr/guide/7_3/solr-tutorial.html)
    1. 下载二进制包（需安装 java 1.8 版本）  
        `wget http://mirrors.hust.edu.cn/apache/lucene/solr/7.3.1/solr-7.3.1.zip`
        > 下载源码包  
        >   `wget http://mirrors.hust.edu.cn/apache/lucene/solr/7.3.1/solr-7.3.1-src.tgz`

    2. 解压缩  
        `unzip -q solr-7.3.1.zip`
    3. 启动
        - 单机模式
            1. 方法一：使用 jetty 启动 solr  
                `cd solr-7.3.1/`  
                `./bin/solr start`  
                ~~`./bin/solr start -p 8984`~~  
                浏览器中访问 http://localhost:8983/solr/ 进入管理页面  
                > 查看 solr 服务状态  
                > `./bin/solr status`  
                > 停止 solr 服务  
                > `./bin/solr stop`  
                > 创建 solr 库 core4  
                > `.bin/solr create -c core4`

            2. 方法二：<span id="single_run_with_tomcat">[使用 tomcat 运行](http://blog.csdn.net/happyzwh/article/details/51741204)</span>
                1. 搭建 tomcat 环境
                    1. 下载 tomcat  
                        `curl -O http://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.5.31/bin/apache-tomcat-8.5.31.zip`
                    2. 解压到当前目录下  
                        `unzip -q apache-tomcat-8.5.31.zip`
                    3. 创建 tomcat 软连接  
                        `ln -s apache-tomcat-8.5.31 tomcat`
                    4. 删除 tomcat/webapps 无用的工程  
                        `rm -rf tomcat/webapps/*`
                    5. 设置启动、停止脚本可执行权限  
                        `chmod +x tomcat/bin/*.sh`
                2. 创建 solrhome  
                    `mkdir tomcat/solrhome`  
                    `cp solr-7.3.1/server/solr/solr.xml tomcat/solrhome/`  
                    ~~复制 solr-7.3.1/server/solr/ 目录下所有文件到 tomcat/solrhome/ 目录下~~  
                    ~~`cp -r solr-7.3.1/server/solr/* tomcat/solrhome/`~~
                3. <span id="deploy_solr_in_tomcat_webapp">部署 solr 工程至 tomcat 中</span>
                    1. 在 tomcat/webapps/ 目录下创建文件夹 solr  
                        `mkdir tomcat/webapps/solr`
                    2. 将 solr-7.3.1/server/solr-webapp/webapp 下的所有文件拷贝到 tomcat/webapps/solr/ 目录下  
                        `cp -r solr-7.3.1/server/solr-webapp/webapp/* tomcat/webapps/solr/`
                    3. 复制 solr 依赖的 jar 包到 WEB-INF/lib 目录下  
                        `cp solr-7.3.1/server/lib/metrics-* tomcat/webapps/solr/WEB-INF/lib/`  
                        `cp solr-7.3.1/server/lib/ext/* tomcat/webapps/solr/WEB-INF/lib/`
                    4. 复制日志配置文件到 WEB-INF/classes 目录下  
                        `mkdir tomcat/webapps/solr/WEB-INF/classes`  
                        `cp solr-7.3.1/server/resources/log4j.properties tomcat/webapps/solr/WEB-INF/classes/`
                    5. 配置 solrhome 路径
                        - 方法一：在 tomcat 中配置 solrhome 路径  
                            修改文件 context.xml，在 `\\Context` 节点下添加 Environment 配置项
                            ```xml
                            <Environment name="solr/home" type="java.lang.String" value="../solrhome" override="false" />
                            ```
                        - 方法二：在 web.xml 中配置 solrhome 路径  
                            修改文件 web.xml，在 `\web-app` 节点下，添加（或取消注释并修改 env-entry-value）配置如下
                            ```xml
                            <env-entry>
                                <env-entry-name>solr/home</env-entry-name>
                                <env-entry-value>../solrhome</env-entry-value>
                                <env-entry-type>java.lang.String</env-entry-type>
                            </env-entry>
                            ```
                            <!-- 修改文件 web.xml，取消配置项 "\<env-entry>" 的注释  
                            `sed -i '40d;46d' tomcat/webapps/solr/WEB-INF/web.xml`  
                            修改文件 web.xml，将配置项 "\<env-entry-value>" 中的值修改为 solrhome 目录所在位置  
                            ~~`sed -i "s#\/put\/your\/solr\/home\/here#$(pwd)\/tomcat\/solrhome#" tomcat/webapps/solr/WEB-INF/web.xml`~~  
                            `sed -i "s#\/put\/your\/solr\/home\/here#..\/solrhome#" tomcat/webapps/solr/WEB-INF/web.xml` -->
                        - 方法三：在启动命令中使用 jvm 启动参数 `-Dsolr.solr.solrhome=../solrhome`
                    6. 修改文件 web.xml，注释节点 `\web-app\security-constraint`（该配置限制了对 solr 资源的访问，若不注释，则抛出异常 "HTTP Status 403 - Access to the requested resource has been denied"）  
                        `sed -i 's/<security-constraint>/<!-- <security-constraint>/' tomcat/webapps/solr/WEB-INF/web.xml`  
                        `sed -i 's/<\/security-constraint>/<\/security-constraint> -->/' tomcat/webapps/solr/WEB-INF/web.xml`
                        <!-- `sed -i '161i<!--' tomcat/webapps/solr/WEB-INF/web.xml`  
                        `sed -i '177i--\>' tomcat/webapps/solr/WEB-INF/web.xml` -->
                    7. （可选）配置安全认证  
                        1. 编辑文件 tomcat/conf/tomcat-users.xml，在 `\tomcat-users` 节点下添加用户、角色配置如下
                            ```xml
                            <role rolename="solr_admin" />
                            <role rolename="core3_admin" />
                            <user username="solr_admin" password="solr_admin" roles="solr_admin" />
                            <user username="core3_admin" password="core3_admin" roles="core3_admin" />
                            ```
                        2. [配置 SSL](tomcat.md#tomcat_setup_ssl)
                        3. 编辑文件 web.xml，在 `\web-app` 节点下添加 security-constraint 配置如下  
                            ```xml
                            <!-- 查询接口无需认证 -->
                            <security-constraint>
                                <web-resource-collection>
                                    <web-resource-name>Solr Select Interface</web-resource-name>
                                    <url-pattern>/core3/select</url-pattern>
                                    <http-method>GET</http-method><!-- 仅允许（未认证用户）使用 GET 方法访问该接口 -->
                                </web-resource-collection>
                            </security-constraint>

                            <!-- core 管理接口需 core admin 角色认证 -->
                            <security-constraint>
                                <web-resource-collection>
                                    <web-resource-name>Core3 Admin</web-resource-name>
                                    <url-pattern>/core3/*</url-pattern>
                                </web-resource-collection>
                                <auth-constraint>
                                    <role-name>core3_admin</role-name>
                                </auth-constraint>
                                <user-data-constraint><!-- 需转换成 https 访问，即禁止以 http 访问该接口 -->
                                    <transport-guarantee>CONFIDENTIAL</transport-guarantee>
                                </user-data-constraint>
                            </security-constraint>

                            <!-- solr admin 管理接口、页面需 solr admin 角色认证 -->
                            <security-constraint>
                                <web-resource-collection>
                                    <web-resource-name>Solr Admin</web-resource-name>
                                    <!-- <url-pattern>/admin/*</url-pattern> -->
                                    <url-pattern>/*</url-pattern>
                                </web-resource-collection>
                                <auth-constraint>
                                    <role-name>solr_admin</role-name>
                                </auth-constraint>
                                <user-data-constraint><!-- 需转换成 https 访问，即禁止以 http 访问该接口 -->
                                    <transport-guarantee>CONFIDENTIAL</transport-guarantee>
                                </user-data-constraint>
                            </security-constraint>
                            <login-config>
                                <auth-method>DIGEST</auth-method>
                            </login-config>
                            ```
                4. 启动 tomcat  
                    `tomcat/bin/startup.sh`
                5. 访问 http://127.0.0.1:8080/solr/index.html 可进入 solr 管理页面
                6. 创建一个 Core 实例（Core 实例配置可参考[创建、配置 Core 实例 core3](#create_core3)）
                    - 快速创建一个 Core 实例 core1
                        1. 创建一个空文件夹 core1  
                            `mkdir tomcat/solrhome/core1`
                        2. 复制示例配置文件  
                            `cp -r solr-7.3.1/server/solr/configsets/sample_techproducts_configs/conf/ tomcat/solrhome/core1`
                        3. 登录进入 solr 管理页面添加 Core
                    - 创建一个极简配置的 Core 实例 core2
                        ```bash
                        # 递归创建一个空文件夹 core2/conf
                        mkdir -p tomcat/solrhome/core2/conf

                        # 创建配置文件 solrconfig.xml
                        cat > tomcat/solrhome/core2/conf/solrconfig.xml << EOF
                        <config>
                            <luceneMatchVersion>7.3.1</luceneMatchVersion>
                            <requestHandler name="/select" class="solr.SearchHandler" >
                                <lst name="defaults">
                                    <str name="df">text</str>
                                </lst>
                            </requestHandler>
                        </config>
                        EOF

                        # 创建配置文件 managed-schema
                        cat > tomcat/solrhome/core2/conf/managed-schema << EOF
                        <schema name="example" version="1.6">
                            <field name="text" type="text_general" indexed="true" stored="true" multiValued="true" />
                            <dynamicField name="*" type="string" indexed="true" stored="false"  multiValued="true" />
                            <copyField source="*" dest="text" />
                            <fieldType name="string" class="solr.StrField" />
                            <fieldType name="text_general" class="solr.TextField">
                                <analyzer>
                                    <tokenizer class="solr.LowerCaseTokenizerFactory" />
                                </analyzer>
                            </fieldType>
                        </schema>
                        EOF
                        ```
                        登录进入 solr 管理页面添加 Core

        - 集群模式（SolrCloud）
            1. 方法一：使用 jetty 启动 solr  
                `cd solr-7.3.1/`  
                `./bin/solr start -e cloud`  
                访问 http://localhost:8983/solr/ 进入管理页面
            2. 方法二：[使用 tomcat 运行](https://www.jianshu.com/p/4e00db4f8c47)
                1. 部署 zookeeper（原则上，zookeeper 节点数量应大于 tomcat 节点数量的一半。如 zookeeper 有 3 个，tomcat 有 4 个）
                    1. 分别在不同的主机上[部署 zookeeper](zookeeper.md)
                        1. 下载 zookeeper  
                            `wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.11/zookeeper-3.4.11.tar.gz`
                        2. 解压到 zookeeper 目录下  
                            `tar -zxvf zookeeper-3.4.11.tar.gz && mv zookeeper-3.4.11 zookeeper`
                        3. 新建配置文件 zoo.cfg
                            ```bash
                            cat > zookeeper/conf/zoo.cfg << EOF
                            tickTime=2000
                            initLimit=10
                            syncLimit=5
                            dataDir=/tmp/zookeeper
                            clientPort=2181
                            server.1=192.168.1.101:2888:3888
                            server.2=192.168.1.102:2888:3888
                            server.3=192.168.1.103:2888:3888
                            EOF
                            ```
                        4. ~~为每个主机建立一个 myid 文件，文件内容不应相同，可以为 1、2、3...  
                            `echo 1 > ./zookeeper/data/myid`~~
                    2. 分别启动各主机上的 zookeeper 实例  
                        ~~`./zookeeper/bin/zkServer.sh start`~~  
                        `./zookeeper/bin/zkServer.sh`
                2. 部署 solr
                    1. 分别在不同的主机上部署 solr，部署方法同[使用 tomcat 运行单机模式下的 solr](#single_run_with_tomcat)（需要配置 solrhome，但不需要创建 Core 实例）
                    2. 修改每个主机中 solrhome 目录下的 solr.xml  
                        编辑 solrhome/solr.xml 文件，修改 `/solr/solrcloud` 节点下的配置 host、hostPort （即当前主机提供的 solr 服务的 ip 和端口号）如下
                        ```xml
                        <solrcloud>
                            <str name="host">192.168.1.101</str>
                            <int name="hostPort">8080</int>
                            ...
                        </solrcloud>
                        ```
                        > 各主机应根据自身实际 ip 设置 host 内容。伪分布式情况下，各 solr 实例的 host、hostPort 不应相同

                    3. 分别在不同的主机上设置 tomcat 启动参数  
                        编辑 tomcat/bin/catalina.sh，添加以下 jvm 参数  
                        `JAVA_OPTS="-DzkHost=192.168.1.101:2181,192.168.1.101:2181,192.168.1.101:2181"`
                    4. 将 solr 配置文件上传到 zookeeper 中，进行统一管理  
                        <!-- `./zookeeper/bin/zkCli.sh -zkhost 192.168.1.101:2181,192.168.1.101:2181,192.168.1.101:2181 -cmd upconfig -confdir tomcat/solrhome/configsets/_default/conf/ -confname collection1` -->
                        `java -cp tomcat/webapps/solr/WEB-INF/lib/* org.apache.solr.cloud.ZkCLI -zkhost 192.168.1.101:2181,192.168.1.101:2181,192.168.1.101:2181 -cmd upconfig -confdir tomcat/solrhome/configsets/_default/conf/ -confname collection1`
                        > 删除配置文件  
                        > `java -cp tomcat/webapps/solr/WEB-INF/lib/* org.apache.solr.cloud.ZkCLI -zkhost 192.168.1.101:2181,192.168.1.101:2181,192.168.1.101:2181 -cmd clear /configs/collection1`

                    4. 分别启动各主机上的 solr 实例，参考[使用 tomcat 运行单机模式下的 solr](#single_run_with_tomcat)中的 "启动 tomcat" 步骤
                    5. 访问 http://192.168.1.101:8080/solr/index.html 可进入 solr 管理页面
                    6. 创建 Collection 实例  
                        `curl http://192.168.1.101:8080/solr/admin/collections?action=CREATE&name=collection1&numShards=2&replicationFactor=2`  
                        删除 Collection 实例  
                        `curl http://192.168.1.101:8080/solr/admin/collections?action=DELETE&name=collection1`
                        > - 实测，linux 环境中 solr 以 cloud 模式运行时，collection 名称（如 test_test）中带有 "_" 的需要指定 collection.configName  
                        > `curl http://127.0.0.1:8080/solr/admin/collections?action=CREATE&name=test_test&numShards=2&replucationFactor=2&collection.configName=test_test`

                > [伪分布式情况](http://www.cnblogs.com/xiazh/articles/4084025.html)下，ip 相同，zookeeper 的 dataDir、clientPort 不应相同，server 的路径和端口号不应相同，其他操作步骤同以上的集群模式搭建过程。伪分布式 solr cloud 环境自动化搭建脚本见 [setup_fake_solr_cloud.sh](#/Source/shell/setup_fake_solr_cloud.sh)。

- <span id="create_core3">创建、配置 Core 实例 core3</span>
    1. 在 solrhome 目录下创建配置文件  
        `mkdir -p tomcat/solrhome/core3/conf && cd tomcat/solrhome/core3/conf`  
        `touch solrconfig.xml managed-schema`
    2. <span id="config_core3">编辑配置文件</span>
        - solrconfig.xml
            ```xml
            <config>
                <luceneMatchVersion>7.3.1</luceneMatchVersion>
                <requestHandler name="/select" class="solr.SearchHandler" >
                    <lst name="defaults">
                        <str name="df">text</str>
                    </lst>
                </requestHandler>
            </config>
            ```
        - managed-schema
            ```xml
            <schema name="example" version="1.6">
                <field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false" />
                <uniqueKey>id</uniqueKey>

                <field name="text" type="text_general" indexed="true" stored="true" multiValued="true" />
                <dynamicField name="*" type="string" indexed="true" stored="false"  multiValued="true" />

                <fieldType name="string" class="solr.StrField" />
                <fieldType name="text_general" class="solr.TextField">
                    <analyzer>
                        <tokenizer class="solr.LowerCaseTokenizerFactory" />
                    </analyzer>
                </fieldType>
            </schema>
            ```
    - [配置拼写检查（纠错）](http://www.cnblogs.com/blog-zuo/p/4819058.html)
        1. 编辑 solrconfig.xml，在 `/config` 节点下，新增 searchComponent 配置项并替换 requestHandler 配置项，如下
            ```xml
            <config>
                ...
                <searchComponent name="spellcheck" class="solr.SpellCheckComponent">
                    <!-- 将输入关键词当做text_general类型进行处理 -->
                    <str name="queryAnalyzerFieldType">text_general</str>
                    <!-- a spellchecker built from a field of the main index -->
                    <lst name="spellchecker">
                        <!-- 拼写检查模块名 -->
                        <str name="name">default</str>
                        <!-- 对索引中的哪个字段进行拼写检查 -->
                        <str name="field">text</str>
                        <!-- 自定义拼写检查，可以用自定义拼写检查类代替默认类 -->
                        <str name="classname">solr.DirectSolrSpellChecker</str>
                        <!-- 拼写检查编辑距离, 默认使用internal levenshtein -->
                        <str name="distanceMeasure">internal</str>
                        <!-- minimum accuracy needed to be considered a valid spellcheck suggestion -->
                        <float name="accuracy">0.5</float>
                        <!-- 最大编辑距离，与输入字符串编辑距离小于等于2的字符串被检索出来作为纠错结果 -->
                        <int name="maxEdits">2</int>
                        <!-- 与输入字符串最少有一个字符相等，才能被检索出来 -->
                        <int name="minPrefix">1</int>
                        <!-- maximum number of inspections per result. 一次纠错探测最大数量 -->
                        <int name="maxInspections">5</int>
                        <!-- 纠错元词最小长度，元词长度小于4不进行纠错 -->
                        <int name="minQueryLength">4</int>
                        <!-- maximum threshold of documents a query term can appear to be considered for correction -->
                        <float name="maxQueryFrequency">0.01</float>
                    </lst>
                    <lst name="spellchecker">
                        <str name="name">wordbreak</str>
                        <str name="classname">solr.WordBreakSolrSpellChecker</str>
                        <str name="field">text</str>
                        <str name="combineWords">true</str>
                        <str name="breakWords">true</str>
                        <int name="maxChanges">10</int>
                    </lst>
                    <!-- 拼写检查器，使用 JaroWinklerDistance 距离算法 -->
                    <!-- <lst name="spellchecker">
                        <str name="name">jarowinkler</str>
                        <str name="classname">solr.IndexBasedSpellChecker</str>
                        <str name="field">spell</str>
                        <str name="distanceMeasure">org.apache.lucene.search.spell.JaroWinklerDistance</str>
                        <str name="spellcheckIndexDir">./spellchecker2</str>
                        <str name="buildOnCommit">true</str>
                    </lst> -->
                </searchComponent>
                ...
                <requestHandler name="/select" class="solr.SearchHandler" startup="lazy">
                    <lst name="defaults">
                        <str name="df">text</str>
                        <!-- 下边配置了两个拼写检查子模块，是前边定义好的default和wordbreak，solr会分别用两个模块对输入进行拼写检查，最终将结果整合到一块 -->
                        <str name="spellcheck.dictionary">default</str>
                        <str name="spellcheck.dictionary">wordbreak</str>
                        <str name="spellcheck">on</str>
                        <!-- 为纠错后的提示词添加额外信息，如在索引中的频率 -->
                        <str name="spellcheck.extendedResults">true</str>
                        <!-- 一次纠错返回结果数量 -->
                        <str name="spellcheck.count">10</str>
                        <!-- The maximum number of suggestions to return for terms that exist in the index -->
                        <str name="spellcheck.alternativeTermCount">5</str>
                        <!-- The maximum number of results the query can return while still triggering spelling suggestions -->
                        <str name="spellcheck.maxResultsForSuggest">5</str>
                        <!-- 是否添加校验结果 -->
                        <str name="spellcheck.collate">true</str>
                        <!-- 是否添加校验拓展结果 -->
                        <str name="spellcheck.collateExtendedResults">true</str>
                        <!-- The maximum # of collation possibilities to try before giving up. -->
                        <str name="spellcheck.maxCollationTries">10</str>
                        <!-- 返回校验结果的最大数目 -->
                        <str name="spellcheck.maxCollations">5</str>
                    </lst>
                    <!-- 必须将拼写检查控件添加到搜索控件序列中，若无此项则不进行拼写检查 -->
                    <arr name="last-components">
                        <str>spellcheck</str>
                    </arr>
                </requestHandler>
                ...
            </config>
            ```
        2. 重启 solr 服务，执行查询操作，查询结果的 spellcheck.suggestions.suggestion 下将会显示检查纠正的单词 word 和频率 freq
        > 参考 http://wiki.apache.org/solr/SpellCheckComponent

    - [配置 solr 自带的中文分词（和 iK 的区别是不能自己添加词库）](http://www.cnblogs.com/wander1129/p/6658828.html)
        1. <span id="setup_smartcn_analyzer">复制 lucene-analyzers-smartcn-7.3.1.jar 到 tomcat/webapps/solr/WEB-INF/lib/ 目录下</span>  
            `cp solr-7.3.1/contrib/analysis-extras/lucene-libs/lucene-analyzers-smartcn-7.3.1.jar tomcat/webapps/solr/WEB-INF/lib/`
        1. 编辑 managed-schema，在 `/schema/fieldType[@name="text_general"]` 节点下，添加 analyzer 配置项，如下
            ```xml
            <schema ...>
                ...
                <fieldType name="text_general" class="solr.TextField" positionIncrementGap="0">
                    ...
                    <analyzer type="index">
                        <tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory"/>
                    </analyzer>
                    <analyzer type="query">
                        <tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory"/>
                    </analyzer>
                </fieldType>
                ...
            </schema>
            ```
    - [配置 ik 中文分词器](http://www.cnblogs.com/wander1129/p/6658828.html)<!-- http://www.cnblogs.com/zhuxiaojie/p/5764680.html -->
        1. <span id="setup_ik_analyzer">下载 [IKAnalyzer](http://files.cnblogs.com/files/wander1129/ikanalyzer-solr6.5.zip)</span>
            1. 复制 [ik-analyzer-solr5-5.x.jar](https://github.com/EugenePig/ik-analyzer-solr5) 至 tomcat/webapps/solr/WEB-INF/lib 目录下
            2. 将 IKAnalyzer.cfg.xml、ext.dic、stopword.dic 复制到 tomcat/webapps/solr/WEB-INF/classes 目录下
            > 其中，文件  
            > - ext.dic 为扩展字典，可增加自己的扩展词典，如：唯品会 聚美优品  
            > - stopword.dic 为停止词字典  
            > - IKAnalyzer.cfg.xml 为配置文件  
            > - solr-analyzer-ik-5.1.0.jar、ik-analyzer-solr5-5.x.jar 为分词 jar 包  
            > stopword.dic、ext.dic 的编码方式为 UTF-8 无 BOM 的编码方式
        2. 编辑 managed-schema，在 `/schema/fieldType[@name="text_general"]` 节点下，添加 analyzer 配置项，如下
            ```xml
            <schema ...>
                ...
                <fieldType name="text_ik" class="solr.TextField">
                    <analyzer type="index">
                        <tokenizer class="org.wltea.analyzer.lucene.IKTokenizerFactory" useSmart="true"/>
                    </analyzer>
                    <analyzer type="query">
                        <tokenizer class="org.wltea.analyzer.lucene.IKTokenizerFactory" useSmart="true"/>
                    </analyzer>
                </fieldType>
                ...
            </schema>
            ```
        > 测试：
        > 1. 建立字段如 summary，字段类型为 text_ik
        > 2. 建立索引数据如 { "id": 1, "summary": "明月装饰了你的窗子，你装饰了别人的梦。" }
        > 3. 执行查询语句如 [http://localhost:8080/solr/xxx/select?q=summary:明月](http://localhost:8080/solr/xxx/select?q=summary:%E6%98%8E%E6%9C%88)
    - 配置拼音检索&简拼检索
        1. 复制 pinyin4j-2.5.0.jar、[pinyinAnalyzer.jar](https://github.com/liangbaolin/pinyinAnalyzer)（可从 [pinyin.zip](http://files.cnblogs.com/files/wander1129/pinyin.zip) 中解压获取）至 tomcat/webapps/solr/WEB-INF/lib 目录下
        2. 下载并安装分词器 [ik 中文分词器](#setup_ik_analyzer) ~~或 [lucene-analyzers-smartcn 中文分词器](#setup_smartcn_analyzer)~~
        3. 编辑 managed-schema，在 `/schema/fieldType[@name="text_pinyin"]` 节点下，添加 analyzer 配置项，如下
            ```xml
            <schema ...>
                ...
                <fieldType name="text_pinyin" class="solr.TextField" positionIncrementGap="0">
                    <analyzer type="index">
                        <tokenizer class="org.apache.lucene.analysis.ik.IKTokenizerFactory"/><!-- 依赖 ik 中文分词器 -->
                        <!-- <tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory"/> --><!-- 依赖 lucene-analyzers-smartcn 中文分词器 -->
                        <filter class="com.shentong.search.analyzers.PinyinTransformTokenFilterFactory" minTermLenght="2" />
                        <filter class="com.shentong.search.analyzers.PinyinNGramTokenFilterFactory" minGram="1" maxGram="20" />
                    </analyzer>
                    <analyzer type="query">
                        <tokenizer class="org.apache.lucene.analysis.ik.IKTokenizerFactory"/>
                        <!-- <tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory"/> -->
                        <filter class="com.shentong.search.analyzers.PinyinTransformTokenFilterFactory" minTermLenght="2" />
                        <filter class="com.shentong.search.analyzers.PinyinNGramTokenFilterFactory" minGram="1" maxGram="20" />
                    </analyzer>
                </fieldType>
                ...
            </schema>
            ```
        > 测试：
        > 1. 建立字段如 summary，字段类型为 text_pinyin
        > 2. 建立索引数据如 { "id": 1, "summary": "明月装饰了你的窗子，你装饰了别人的梦。" }
        > 3. 执行查询语句如 [http://localhost:8080/solr/xxx/select?q=summary:minyue](http://localhost:8080/solr/xxx/select?q=summary:minyue)、[http://localhost:8080/solr/xxx/select?q=summary:my](http://localhost:8080/solr/xxx/select?q=summary:my)（简拼暂只适用于 ik 分词器）

    - <span id="config_uuid">配置自动生成 id</span>
        1. 编辑 managed-schema，在 `/schema` 节点下，新增 uuid 字段类型
            ```xml
            <fieldType name="uuid" class="solr.UUIDField" indexed="true" />
            ```
            在 `/schema/field[@name="id"]` 节点中，修改字段 id 的类型为 uuid  
            `sed -i 's/<field name="id" type="string"/<field name="id" type="uuid"/g' managed-schema`  
        2. 编辑 solrconfig.xml，在 `/config` 节点下，新增更新策略配置，调用 Solr 中的 UUIDUpdateProcessorFactory 生成全局唯一的 UUID
            ```xml
            <updateRequestProcessorChain name="uuid">
                <processor class="solr.UUIDUpdateProcessorFactory">
                    <str name="fieldName">id</str>
                </processor>
                <processor class="solr.LogUpdateProcessorFactory" />
                <processor class="solr.DistributedUpdateProcessorFactory" />
                <processor class="solr.RunUpdateProcessorFactory" />
            </updateRequestProcessorChain>
            ```
            在 `/config/requestHandler/lst` 节点下（配置 requestHandler 如 /dataimport、/update 等，使其能够自动生成 UUID），新增<span id="chain_uuid">自动生成 UUID 的策略</span>如下
            ```xml
            <str name="update.chain">uuid</str>
            ```
    - 从 mysql 数据库中导入数据
        1. 添加依赖 jar 包
            1. 复制 solr-dataimporthandler-7.3.1.jar、solr-dataimporthandler-extras-7.3.1.jar 到 solr 的 lib 中  
                `cp solr-7.3.1/dist/solr-dataimporthandler-*.jar tomcat/webapps/solr/WEB-INF/lib/`
            2. 下载 mysql 驱动包  
                `curl -O http://repo1.maven.org/maven2/mysql/mysql-connector-java/6.0.6/mysql-connector-java-6.0.6.jar`
            3. 编辑 solrconfig.xml，在 `/config` 节点下，新增 DataImport、mysql-connector 的 lib 配置项如下
                ```xml
                <lib dir="~/tomcat/webapps/solr/WEB-INF/lib" regex="solr-dataimporthandler-\d.*\.jar" />
                <lib dir="~/tomcat/webapps/solr/WEB-INF/lib" regex="mysql-connector-java-\d.*\.jar" />
                ```
        2. 配置数据源
            > 例如，远程主机 192.168.128.137 上安装有 mysql 数据库服务，其中用户名为 test，密码为 test，现需要获取数据库 mysql 下数据表 user 中的数据，且取该表中列 Host、User、Password 数据并将其导入到索引库中。
            > 默认情况下，mysql 中的 root 用户禁止远程访问，需要远程访问可参考配置 [mysql 设置允许 root 用户远程访问](mysql.md#allow_root_remote)

            1. 在 solrconfig.xml 同级目录下，新建配置文件 data-config.xml 并配置 dataConfig 信息
                - <span id="config_solr_dataConfig">方法一：直接配置数据源，配置内容如下</span>
                    ```xml
                    <dataConfig>
                        <dataSource driver="com.mysql.jdbc.Driver" url="jdbc:mysql://192.168.128.137:3306/mysql" user="test" password="test" />
                        <document>
                            <entity name="user" query="select Host, User, Password as Pass from user;">
                                <field column="Host" name="host" />
                                <field column="User" name="user" />
                                <field column="Pass" name="pass" />
                            </entity>
                        </document>
                    </dataConfig>
                    ```
                - [方法二：使用 tomcat 数据源，即 jndi](https://www.cnblogs.com/edwinchen/p/3975004.html)
                    1. [在 tomcat 中配置数据源](tomcat.md#config_datasource)
                    2. 配置 dataConfig 信息同[方法一](#config_solr_dataConfig)，其中节点 `/dataConfig/dataSource` 替换为以下内容
                        ```xml
                        <dataSource name="solr" jndiName="java:comp/env/jdbc/solr" type="JdbcDataSource"/>
                        ```
                        可配置多个 dataSource 节点可实现多数据源配置
            2. 编辑 managed-schema，在 `/schema` 节点下，新增字段（用于映射第一步中配置的 field）如下
                ```xml
                <field name="host" type="string" indexed="true" stored="true" multiValued="false" />
                <field name="user" type="string" indexed="true" stored="true" multiValued="false" />
                <field name="pass" type="string" indexed="true" stored="true" multiValued="false" />
                ```
            3. 编辑 solrconfig.xml 文件，在 `/config` 节点下，新增 requestHandler 配置项如下
                ```xml
                <requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
                    <lst name="defaults">
                        <str name="config">data-config.xml</str>
                    </lst>
                </requestHandler>
                ```
            4. 参照[配置自动生成 id](#config_uuid)，在文件 solrconfig.xml 的 `/config/requestHandler[@name="/dataimport"]/lst` 节点下，新增[自动生成 UUID 的策略](#chain_uuid)
            5. 进入页面 http://localhost:8080/solr/index.html#/core3/dataimport//dataimport 执行导入操作

- [solr 配置说明](https://www.jianshu.com/p/2ac7dac7f6f2)
    - schema
        - <span id="schema_field">[field](http://lucene.apache.org/solr/guide/6_6/defining-fields.html)</span>  
            用于字段的定义（位于 `/schema` 节点下），如
            ```xml
            <field name="title" type="string" indexed="true" stored="true" multiValued="false" />
            ```
            > 其中，属性  
            > - `name` 标示这个字段的名称
            > - `type` 标示字段的类型，值对应 [fieldType](#schema_fieldType) 元素的 name 属性值 `/schema/fieldType/@name`
            > - `indexed` 定义文档是否被索引（也可以理解为是否在此字段上进行搜索），值为 true 或 false
            > - `stored` 定义这个字段内容是否存储（也可以理解为是否可以作为返回结果的一部分），值为 true 或 false  
            >   [字段 field 定义中的 indexed、stored 配置](#field_config_with_indexed_and_stored)
            > - `multiValued` 这个字段是否存在多个值，如果存在多个值设置为 true
        
        - <span id="schema_fieldType">fieldType</span>

- solr 接口调用
    1. [创建配置基本 Core 实例 core3](#config_core3)
    2. 接口调用
        - [使用 solrj 调用 solr 接口](http://lucene.apache.org/solr/guide/7_2/using-solrj.html)
            1. 引入 maven 依赖
                ```xml
                <dependency>
                    <groupId>org.apache.solr</groupId>
                    <artifactId>solr-solrj</artifactId>
                    <version>7.3.1</version>
                </dependency>
                ```
            3. 调用（完整实例可参考 [SolrJExample.java](https://gitee.com/904243852/Source/tree/master/java/snippets/3rdparty/SolrJExample.java)）
                - 添加或修改索引
                    ```java
                    SolrClient client = new HttpSolrClient.Builder("http://127.0.0.1:8080/solr").build();
                    
                    SolrInputDocument doc = new SolrInputDocument();
                    
                    doc.addField("id", "0a8f4e36-0f20-42d5-9d2d-c68343f6861f");
                    doc.addField("text", "hello, world");
                    
                    client.add("core3", doc);
                    client.commit("core3");

                    client.close();
                    ```
                - 查询
                    ```java
                    SolrClient client = new HttpSolrClient.Builder("http://127.0.0.1:8080/solr").build();
                    
                    Map<String, String> queryParamMap = new HashMap<String, String>();
                    queryParamMap.put("q", "*:*");
                    queryParamMap.put("fl", "id, text");
                    MapSolrParams params = new MapSolrParams(queryParamMap);

                    QueryResponse response = client.query("core3", params);
                    SolrDocumentList documents = response.getResults();
                    
                    for (SolrDocument document : documents) {
                        assertTrue(document.getFieldNames().contains("id"));

                        List<String> text = (List<String>) document.getFieldValue("text");
                        assertEquals("hello, world", text.get(0));
                    }

                    client.close();
                    ```
                - 删除索引
                    ```java
                    SolrClient client = new HttpSolrClient.Builder("http://127.0.0.1:8080/solr/core3").build();

                    client.deleteById("0a8f4e36-0f20-42d5-9d2d-c68343f6861f");
                    // client.deleteByQuery("id:0a8f4e36-0f20-42d5-9d2d-c68343f6861f");
                    // client.deleteByQuery("*:*");

                    client.commit();
                    client.close();
                    ```
        - 使用 python 调用 solr 接口
            ```python
            import json
            import requests

            # 查询
            r = requests.get('http://127.0.0.1:8080/solr/core3/select?q=*:*&wt=json', verify = False)
            assert 0 < r.json()['response']['numFound']
            print r.json()

            # 索引
            r = requests.post('http://127.0.0.1:8080/solr/core3/update?wt=json', json = {
                "add": {
                    "doc": { "id": "doc_2", "text": "hello, doc 2" }
                }
            }, params = { "boost": 1.0, "overwrite": "true", "commitWithin": 1000 }, headers = { "Content-Type": "application/json" })
            print r.text
            ```
        - [使用 pysolr 调用 solr 接口](https://www.cnblogs.com/shaosks/p/7845576.html)
            > 安装 pysolr 模块  
            >   `python -m pip install pysolr`

            ```python
            import pysolr

            solr = pysolr.Solr('http://127.0.0.1:8080/solr/core3/', timeout = 10)

            # 索引
            result = solr.add([
                { "id": "doc_1", "text": "hello, doc 1" },
                { "id": "doc_2", "text": "hello, doc 2" }
            ])
            print result
            ```
        - 直接调用 solr 接口
            - 单机（standalone）模式下添加/删除 core
                - 卸下 core 实例的同时，删除对应的 core 实例目录  
                    `curl http://127.0.0.1:8080/solr/admin/cores?action=UNLOAD&deleteInstanceDir=true&core=test`
            - 新增/修改
                - 使用 post 方法提交 json 格式数据  
                    `curl -X POST -H 'Content-Type: application/json' http://localhost:8080/solr/test/update?commit=true -d '[{ "name": "zhangsan", "age": 19, "sex": "male" }]'`  
                    ~~`curl -X POST -H 'Content-Type: application/json' http://localhost:8983/solr/xxx/update/json/docs -d '{ "id": "1", "name": "zhangsan" }'`~~
                - 上传 json 文件  
                    `curl http://localhost:8080/solr/test/update?commit=true --data-binary @/temp/userinfo.json -H 'Content-type: application/json'`
            - 删除
                - 删除所有数据  
                    `curl -X POST -H 'Content-type: application/json' http://localhost:8080/solr/test/update?commit=true -d '{ delete: { query: "*:*" } }'`  
                    ~~`curl -X POST -H 'Content-type: application/json' http://localhost:8080/solr/test/update -d '{ delete: { query: "name:zhangsan" }, commit: {} }'`~~
                - 根据 id 删除数据  
                    `curl -X POST -H 'Content-type: application/json' http://localhost:8080/solr/test/update?commit=true -d '{ delete: { "id": "0a8f4e36-0f20-42d5-9d2d-c68343f6861f" }, delete: { "id": "550e8400-e29b-41d4-a716-446655440000" } }'`
            - 查询
                > [查询参数](http://www.cnblogs.com/zhangweizhong/p/5056884.html)
                > - `q` 指定查询的关键字。例如，q=id:1，默认为 q=*:*
                > - `fl` 指定返回哪些字段，用逗号或空格分隔，字段区分大小写。例如，fl=id,title,sort
                > - `start` 返回结果的第几条记录开始，一般分页用，默认 0 开始
                > - `rows` 指定返回结果最多有多少条记录，默认值为 10，配合 start 实现分页
                > - `sort` 排序方式，例如 id desc 表示按照 "id" 降序
                > - `wt` 指定输出格式，有 xml、json、php 等
                > - `fq` 过虑查询，提供一个可选的筛选器查询。返回在 q 查询符合结果中同时符合的 fq 条件的查询结果。例如，q=id:1&fq=sort:[1 TO 5]，找关键字 id 为 1 的，并且 sort 是 1 到 5 之间的
                > - `df` 默认的查询字段，一般默认指定
                > - `qt` 指定那个类型来处理查询请求，一般不用指定，默认是 standard
                > - `indent` 返回的结果是否缩进，默认关闭，用 indent=true|on 开启，一般调试 json、php、phps、ruby 输出才有必要用这个参数
                > - `version` 查询语法的版本，建议不使用它，由服务器指定默认值

                - 常用查询  
                    查询 name 字段为 zhangsan 的数据，且在返回结果中只展示 name 字段  
                    http://localhost:8080/solr/test/select?name=zhangsan&fl=name  
                    其他  
                    http://localhost:8080/solr/test/select?q=name:zhang
                    http://localhost:8080/solr/test/select?q=name:"zhang san"
                    http://localhost:8080/solr/test/select?q=zhang san&df=name

                - 词权重查询  
                    使用 dismax 查询解析器查询关键词 zhangsan  
                    [http://localhost:8080/solr/test/select?defType=dismax&fl=name,age&q=zhangsan&qf=name^0 age^1.0](http://localhost:8080/solr/test/select?defType=dismax&fl=name,age&q=zhangsan&qf=name%5E0+age%5E1.0)  
                    使用 edismax 查询解析器查询关键词 zhangsan  
                    [http://localhost:8080/solr/test/select?bq=sex:female^3.0&defType=editmax&fl=name,age&q=zhangsan&qf=name^0 age^1.0](http://localhost:8080/solr/test/select?bq=sex:female%5E3&defType=edismax&fl=name,age&q=zhangsan&qf=name%5E0+age%5E1.0)
                    > 其中，查询参数  
                    > - `defType` 指定查询（词权重）解析器，值可以为 dismax、edismax  
                    > - `qf` 即 query field，指定 q 中的词项要在哪些字段上执行查询。可以设置多列以及每一列的权重。如 name^0.1 age^2.0
                    > - `bq`（edismax）接受一个和 q 一样的查询，它和 q 的区别是不影响返回的结果集，只会影响排名
                    > - `bf`（edismax）提升函数，通过数学公式来影响评分，而且不局限在 qf 中的字段  

                    > 通过附加参数 `fl=name, age, [explain]`，可以直接在结果中查看 name、age、权重计算详细值
                
                - 其他查询参数
                    - 使用 facet 聚合查询  
                        `facet=on&facet.field=color&facet.field=memory`
                    - 开启 debug 调试  
                        `debugQuery=on`
                
                - DateField 类型的查询  
                    例如，字段 publishTime 为 DateField 类型
                    ```javascript
                    // [] 包含范围检索，如检索某时间段记录，包含头尾，date:[200707 TO 200710]，{} 不包含范围检索，如检索某时间段记录，不包含头尾

                    // 查询 2015 年 8 月 28 日当天的数据
                    publishTime:[publishTime:[2015-08-28T00:00:00Z TO 2015-08-28T23:59:59.999Z]
                    // 查询 2015 年 8 月 28 日以后的数据，包含当天的数据
                    publishTime:[publishTime:[2015-08-28T00:00:00Z TO *]

                    // 查询 2015 年 8 月 28 日以前的数据
                    publishTime:[publishTime:[* TO 2015-08-28T00:00:00Z]
                    // 或
                    publishTime:[publishTime:[* TO 2015-08-27T23:59:59.999Z]
                    ```
            
            - Schema API  
                使用 curl 发送 post 请求，格式如下  
                `curl -X POST -H 'Content-type:application/json' http://localhost:8080/solr/test/schema --data-binary '$JSON_REQUEST'`  
                其中请求的正文 $JSON_REQUEST 内容可参考以下
                - field（field 各属性可参考 [field 定义](#schema_field)）
                    - 创建 field
                        ```json
                        { "add-field": { "name":"sell-by", "type":"tdate", "stored":true } }
                        ```
                    - 删除 field
                        ```json
                        { "delete-field": { "name":"sell-by" } }
                        ```
                    - 替换 field
                        ```json
                        { "replace-field": { "name":"sell-by", "type":"tdate", "stored":true } }
                        ```
                - dynamic field
                    - 创建 dynamic field
                        ```json
                        { "add-dynamic-field": { "name":"*_s", "type":"string", "stored":true } }
                        ```
                    - 删除 dynamic field
                        ```json
                        { "delete-dynamic-field": { "name":"*_s" } }
                        ```
                    - 替换 dynamic field
                        ```json
                        { "replace-dynamic-field": { "name":"*_s", "type":"text_general", "stored":false  } }
                        ```
                - field type
                    - 创建 field type
                        ```json
                        {
                            "add-field-type" : {
                                "name":"myNewTxtField",
                                "class":"solr.TextField",
                                "positionIncrementGap":"100",
                                "analyzer" : {
                                    "charFilters":[{
                                    "class":"solr.PatternReplaceCharFilterFactory",
                                    "replacement":"$1$1",
                                    "pattern":"([a-zA-Z])\\\\1+" }],
                                    "tokenizer":{
                                    "class":"solr.WhitespaceTokenizerFactory" },
                                    "filters":[{
                                    "class":"solr.WordDelimiterFilterFactory",
                                    "preserveOriginal":"0" }]}}
                        }
                        ```
                    - 删除 field type
                        ```json
                        { "delete-field-type":{ "name":"myNewTxtField" } }
                        ```
                    - 替换 field type
                        ```json
                        {
                            "replace-field-type":{
                                "name":"myNewTxtField",
                                "class":"solr.TextField",
                                "positionIncrementGap":"100",
                                "analyzer":{
                                    "tokenizer":{
                                    "class":"solr.StandardTokenizerFactory" }}}
                        }
                        ```
                - copy field
                > 多个 json 格式请求正文也可同时/批量发送，如
                > ```json
                > {
                >     "add-field":{
                >         "name":"shelf",
                >         "type":"myNewTxtField",
                >         "stored":true },
                >     "add-field":{
                >         "name":"location",
                >         "type":"myNewTxtField",
                >         "stored":true },
                >     "add-copy-field":{
                >         "source":"shelf",
                >         "dest":[ "location", "catchall" ]}
                > }
                > ```
            
            - Config API
                - 查询 requestHandler 配置  
                    `curl http://localhost:8080/solr/test/config/requestHandler`

                使用 curl 发送 post 请求，格式如下  
                `curl http://localhost:8080/solr/test/config -H 'Content-type:application/json'  -d '$JSON_REQUEST'`  
                其中请求的正文 $JSON_REQUEST 内容可参考以下
                - 修改 requesthandler 配置  
                    ```json
                    {
                        "update-requesthandler": {
                            "name": "/select",
                            "class": "solr.SearchHandler",
                            "defaults": {
                                "defType": "dismax",
                                "qf": "name^10 summary^1" } } }
                    ```
            - cloud 模式下，新增、删除副本  
                删除一个副本  
                `curl http://localhost:8080/admin/collections?action=DELETEREPLICA&collection=prodIndex&shard=shard1&replica=core_node3`  
                新增一个副本  
                `curl http://localhost:8080/admin/collections?action=ADDREPLICA&collection=prodIndex&shard=shard1&node=192.168.1.102:8080_`
            - 集群模式下，查询指定的 shard 或 replica
                ```bash
                curl http://localhost:8080/solr/test?q=*:*&shards=shard1 # 指定 shard，多个分片名称以“,”分隔
                curl http://localhost:8080/solr/test?q=*:*&shards=localhost:8081/solr/test # 指定 replica，多个副本地址以“|”分隔
                ```
            - 检测索引文件  
                `java -cp $CATALINA_HOME/webapps/solr/WEB-INF/lib/lucene-core-*.jar -ea:org.apache.lucene... org.apache.lucene.index.CheckIndex $CATALINA_HOME/solrhome/prodIndex`

- solr 源码解析
    ```java
    // 获取 solrHome 路径
    Path solrHome = org.apache.solr.core.SolrResourceLoader.locateSolrHome();
    ```

## Troubleshooting

- [Recovery 失败造成的宕机原因分析](https://www.cnblogs.com/rcfeng/p/4287030.html)  
    [solrcloud Recovery 原理及无法选举分片 leader](http://i.ifeng.com/lady/vnzq/news?cid=0&aid=120274197)

- <span id="field_config_with_indexed_and_stored">字段 field 定义中的 indexed、stored 配置</span>
    > stored、docValues、indexed 未指定为 false 时，默认值均为 true

    1. indexed = false  
        `curl -X POST -H 'Content-type:application/json' http://localhost:8080/solr/test/schema --data-binary '{ "replace-field": { "name":"sex", "type":"string", "indexed":false } }'`
        - 新增数据  
            `{ "name": "zhangsan", "age": 18, "sex": "male" }`
        - 查询
            - `q=*:*` 同 4
            - `q=sex:male`
                ```json
                { "name":"zhangsan", "age":18, "sex":"male", "id":"...", "_version_":... },
                { "name":"zhaoliu", "age":21, "id":"...", "_version_":..., "sex":"male" }
                ```
    2. stored = false, docValues = false  
        `curl -X POST -H 'Content-type:application/json' http://localhost:8080/solr/test/schema --data-binary '{ "replace-field": { "name":"sex", "type":"string", "stored":false, "docValues":false } }'`
        - 新增数据  
            `{ "name": "lisi", "age": 19, "sex": "female" }`
        - 查询
            - `q=*:*` 同 4
            - `q=sex:male` 同 4
    3. stored = false, docValues = false, indexed = false  
        `curl -X POST -H 'Content-type:application/json' http://localhost:8080/solr/test/schema --data-binary '{ "replace-field": { "name":"sex", "type":"string", "stored":false, "docValues":false, "indexed":false } }'`
        - 新增数据  
            `{ "name": "wangwu", "age": 20, "sex": "unknow" }`
        - 查询
            - `q=*:*` 同 4
            - `q=sex:male` 同 4
    4. stored = false  
        `curl -X POST -H 'Content-type:application/json' http://localhost:8080/solr/test/schema --data-binary '{ "replace-field": { "name":"sex", "type":"string", "stored":false } }'`
        - 新增数据  
            `{ "name": "zhaoliu", "age": 21, "sex": "male" }`
        - 查询
            - `q=*:*`
                ```json
                { "name":"zhangsan", "age":18, "sex":"male", "id":"...", "_version_":... },
                { "name":"lisi", "age":19, "id":"...", "_version_":..., "sex":"female" },
                { "name":"wangwu", "age":20, "id":"...", "_version_":... },
                { "name":"zhaoliu", "age":21, "id":"...", "_version_":..., "sex":"male" }
                ```
            - `q=sex:male`
                ```json
                { "name":"zhaoliu", "age":21, "id":"...", "_version_":..., "sex":"male" }
                ```
    5. indexed = true, stored = false  
        可搜索（到这条记录），但不返回结果（不能返回该字段的值或原始文档）