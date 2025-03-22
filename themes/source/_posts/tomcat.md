---
title: Tomcat
tags: tomcat
categories: tomcat
---

# Tomcat

- 环境搭建
    1. 下载 tomcat-8.5.31.zip  
        `curl -O http://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.5.31/bin/apache-tomcat-8.5.31.zip`
    2. 解压 tomcat-8.5.31.zip  
        `unzip -q apache-tomcat-8.5.31.zip`
    3. 创建 tomcat 软连接  
        `ln -s apache-tomcat-8.5.31 tomcat`
    
- <span id="start-or-stop-tomcat">停止、启动 Tomcat 服务</span>
    1. 切换至 $CATALINA_HOME/bin 目录  
        `cd tomcat/bin`
    2. 停止、启动服务
        - 停止服务  
            `./shutdown.sh`  
            `ps -ef | grep tomcat | grep -v grep | awk '{print $2}' | xargs kill -s 9`
        - 启动服务  
            `./startup.sh`
    3. 返回目录  
        `cd -`

- <span id="tomcat-setup-ssl">开启 SSL</span>
    1. [创建 keystore 文件](java.md#createKeystore)(如 tomcat.keystore)  
        `keytool -genkeypair -alias tomcat -keyalg RSA -keystore keystore.jks`  
        根据提示输入证书信息，如密码为 123456
    2. 将 keystore.jks 移动至 $CATALINA_HOME/conf 目录下  
        `mv keystore.jks tomcat/conf`
    3. 编辑文件 $CATALINA_HOME/conf/server.xml，在 `/Server/Service/Connector[@port="8443"]` 节点中，配置 keystoreFile 和 keystorePass，如下
        ```xml
        <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
                    maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
                    clientAuth="false" sslProtocol="TLS"
                    keystoreFile="${catalina.home}/conf/keystore.jks" keystorePass="123456"/>
        ```
    3. [重新启动 Tomcat 服务](#start-or-stop-tomcat)
    4. 在浏览器中访问如：https://localhost:8443/xxx/

- [配置 SSL 双向认证](https://www.cnblogs.com/Before/p/4244167.html)
    1. <span id="keytool">（在 $CATALINA_HOME/conf 目录下）生成密钥库和证书</span>
        1. 生成服务器证书库 server.jks  
	        `keytool -validity 365 -genkey -v -alias server -keyalg RSA -keystore server.jks -dname "CN=127.0.0.1,OU=NA,O=NA,L=NA,ST=NA,c=NA" -storepass 123456 -keypass 123456`
        2. 从服务器证书库中导出服务器证书 server.cer  
	        `keytool -export -v -alias server -keystore server.jks -storepass 123456 -rfc -file server.cer`
        3. 生成客户端证书库 client.p12  
	        `keytool -validity 365 -genkeypair -v -alias client -keyalg RSA -storetype PKCS12 -keystore client.p12 -dname "CN=client" -storepass 123456 -keypass 123456`
        4. 从客户端证书库中导出客户端证书 client.cer  
	        `keytool -export -v -alias client -keystore client.p12 -storetype PKCS12 -storepass 123456 -rfc -file client.cer`
        5. 将客户端证书导入到服务器证书库（使得服务器信任客户端证书）  
	        `keytool -import -v -alias client -file client.cer -keystore server.jks -storepass 123456 -noprompt`
        6. ~~生成客户端信任证书库（由服务端证书生成的证书库）  
	        `keytool -import -v -alias server -file server.cer -keystore client.truststore -storepass 123456 -noprompt`~~
        > - 查看证书库中的全部证书  
	    >   `keytool -list -keystore server.jks -storepass 123456`
    2. 编辑文件 $CATALINA_HOME/conf/server.xml，在 `/Server/Service/Connector[@port="8443"]` 节点中，配置如下
        ```xml
        <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol" SSLEnabled="true"
                    maxThreads="150" scheme="https" secure="true"
                    clientAuth="true" sslProtocol="TLS"
                    keystoreFile="${catalina.base}/conf/server.jks" keystorePass="123456"
                    truststoreFile="${catalina.base}/conf/server.jks" truststorePass="123456"/>
        ```
        > - [servlet 中获取客户端证书](https://www.cnblogs.com/tianjifa/p/9533092.html)
        >   ```java
        >   // HttpServletRequest request = ...
        >   X509Certificate[] chain = (X509Certificate[]) request.getAttribute("javax.servlet.request.X509Certificate");
        >   ```
    3. （可选）配置角色权限
        1. 编辑文件 $CATALINA_HOME/conf/server.xml，开启 8080 端口（此处用于释放普通权限，对比区别于第二步中配置的权限），配置如下
            ```xml
            <Connector port="8080" protocol="HTTP/1.1"
                        connectionTimeout="20000"
                        redirectPort="8443" />
            ```
        2. 编辑 $CATALINA_HOME/webapps/*/WEB-INF/web.xml 文件，在 `/web-app` 节点下，添加 security-constraint、login-config 配置如下
            ```xml
            <security-constraint>
                <web-resource-collection>
                    <web-resource-name>SSL</web-resource-name>
                    <url-pattern>/*</url-pattern>
                </web-resource-collection>
                <user-data-constraint>
                    <transport-guarantee>CONFIDENTIAL</transport-guarantee>
                </user-data-constraint>
                <security-role>
                    <role-name>admin4cert</role-name>
                </security-role>
            </security-constraint>
            <login-config>
                <auth-method>CLIENT-CERT</auth-method>
            </login-config>
            ```
        3. 编辑 tomcat-users.xml 文件，在 `/tomcat-users` 节点下，配置角色、用户如下
            ```xml
            <role rolename="admin4cert" />

            <user username="CN=client" password="null" roles="admin4cert" />
            ```
    4. [重新启动 Tomcat 服务](#start-or-stop-tomcat)
    5. 安装证书 client.p12
    6. 在浏览器中访问如：https://localhost:8443/

- [配置多个虚拟主机与 HTTPS 证书](https://www.jianshu.com/p/2b4a587db5cb)
    1. 编辑 $CATALINA_HOME/conf/server.xml 文件，配置 `/Server/Service/Connector` 节点如下
        ```xml
        <Connector port="443" protocol="org.apache.coyote.http11.Http11NioProtocol"
                   maxThreads="150" SSLEnabled="true" URIEncoding="UTF-8" defaultSSLHostConfigName="domain.com">

            <SSLHostConfig hostName="domain.com" caCertificateFile="conf/ca.pem">
                <Certificate certificateKeyFile="conf/domain.key"
                             certificateFile="conf/domain.crt"
                             type="RSA" />
            </SSLHostConfig>

            <SSLHostConfig hostName="x1.domain.top" caCertificateFile="conf/ca.pem">
                <Certificate certificateKeyFile="conf/x1.key"
                             certificateFile="conf/x1.crt"
                             type="RSA" />
            </SSLHostConfig>
        </Connector>
        ```

- 全局/局部 url 转换成 https 访问配置（需 [Tomcat 安装 SSL 证书](#tomcat-setup-ssl)）  
    编辑 $CATALINA_HOME/webapps/*/WEB-INF/web.xml 文件，在 `/web-app` 节点下，添加 security-constraint 配置如下
    ```xml
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>SSL</web-resource-name>
            <url-pattern>/*</url-pattern><!-- 全局转换成 https 访问 -->
            <!-- <url-pattern>/test/*</url-pattern> --><!-- 局部使用 https 访问，即当访问路径包括 test 的时候，就会强制转换为 https -->
        </web-resource-collection>
        <user-data-constraint>
            <transport-guarantee>CONFIDENTIAL</transport-guarantee>
        </user-data-constraint>
    </security-constraint>
    ```
    > web-resource-collection 节点可以配置多个，url-pattern 为配置需要强制 https 的 url

- [配置 http 2.0（需 tomcat 版本 >= 8.5）](https://stackoverflow.com/questions/30855281/tomcat-support-for-http-2-0)  
    1. [安装 Tomcat Native Connector](#setup-native-connector)
    2. 配置证书（http 2.0 需强制开启 https）  
        `openssl req -nodes -new -x509 -keyout $CATALINA_HOME/conf/localhost-rsa-key.pem -out $CATALINA_HOME/conf/localhost-rsa-cert.pem`
    3. 编辑 $CATALINA_HOME/conf/server.xml 文件，配置 `/Server/Service/Connector` 节点如下
        ```xml
        <Connector port="8443" protocol="org.apache.coyote.http11.Http11AprProtocol"
               maxThreads="150" SSLEnabled="true" >
            <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
            <SSLHostConfig>
                <Certificate certificateKeyFile="conf/localhost-rsa-key.pem"
                            certificateFile="conf/localhost-rsa-cert.pem"
                            type="RSA" />
            </SSLHostConfig>
        </Connector>
        ```

- 开启远程调试
    - windows 环境下，在 $CATALINA_HOME/bin/catalina.bat 中添加  
        `SET CATALINA_OPTS=-server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8899` 
    - linux 环境下，在 $CATALINA_HOME/bin/catalina.sh 中添加  
        `CATALINA_OPTS="-server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8899"`

- 监控资源文件，文件修改了，则自动重新加载该应用  
    编辑配置文件（即[在 Tomcat 中配置数据源](#config-datasource)中涉及的配置文件），在 `//Context` 节点下，添加 WatchedResource 配置如下
    ```xml
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    ```
    可配置多个 WatchedResource 节点以监听不同文件，当 WatchedResource 中的文件发生修改变动时，当前的 webapp 将自动重新加载

- <span id="tomcat-protocol">[Tomcat 在 Linux 服务器上的 BIO、NIO、APR 模式设置](https://www.cnblogs.com/dijia478/p/8085143.html)</span>
    - BIO 模式  
        BIO（blocking I/O），顾名思义，即阻塞式 I/O 操作，表示Tomcat使用的是传统的 Java I/O 操作（即 java.io 包及其子包）。Tomcat 在默认情况下，就是以 bio 模式运行的。遗憾的是，就一般而言，bio 模式是三种运行模式中性能最低的一种。我们可以通过 Tomcat Manager 来查看服务器的当前状态。
        ```xml
        <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
        ```
    - NIO 模式  
        NIO 是 Java SE 1.4 及后续版本提供的一种新的 I/O 操作方式（即 java.nio 包及其子包）。Java nio 是一个基于缓冲区、并能提供非阻塞 I/O 操作的 Java API，因此nio也被看成是 non-blocking I/O 的缩写。它拥有比传统 I/O 操作（bio）更好的并发运行性能。
        ```xml
        <Connector port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol"
               connectionTimeout="20000"
               redirectPort="8443" />
        ```
    - APR 模式  
        APR（Apache Portable Runtime/Apache 可移植运行库），是 Apache HTTP 服务器的支持库。你可以简单地理解为，Tomcat 将以 JNI 的形式调用 Apache HTTP 服务器的核心动态链接库来处理文件读取或网络传输操作，从而大大地提高 Tomcat 对静态文件的处理性能。Tomcat apr 也是在 Tomcat 上运行高并发应用的首选模式。（需依赖安装 apr、apr-util、tomcat-native）
        1. <span id="setup-native-connector">下载并安装 [Tomcat Native Connector](https://tomcat.apache.org/download-native.cgi)</span>
            - windows  
                `curl -O http://mirror.bit.edu.cn/apache/tomcat/tomcat-connectors/native/1.1.34/binaries/tomcat-native-1.1.34-win32-bin.zip`  
                `unzip -j -q tomcat-native-1.1.34-win32-bin.zip 'bin/x64/tcnative-1.dll' -d $CATALINA_HOME/bin`
        2. 编辑 $CATALINA_HOME/conf/server.xml 文件，修改属性 `/Server/Service/Connector@protocol` 为 `org.apache.coyote.http11.Http11AprProtocol` 如下
            ```xml
            <Connector port="8080" protocol="org.apache.coyote.http11.Http11AprProtocol"
                connectionTimeout="20000"
                redirectPort="8443" />
            ```

- <span id="config-datasource">在 Tomcat 中配置数据源</span>
    1. 编辑配置文件
        - <span id="config-datasource-method-1">方法一：在 $CATALINA_HOME/conf/Catalina/localhost 目录下创建配置文件（文件名称应与 webapp 同名，如 solr.xml，对应 webapps 目录下的 solr 应用），配置信息如下</span>
            ```xml
            <?xml version='1.0' encoding='utf-8'?>
            <Context path="/solr" docBase="solr" workDir="solr">
                <Resource name="jdbc/solr" type="javax.sql.DataSource" auth="Container" driverClassName="com.mysql.jdbc.Driver" url="jdbc:mysql://127.0.0.1:3306/solr" validationQuery="select 1 from dual" username="test" password="123456" />
            </Context>
            ```
        - 方法二：编辑 $CATALINA_HOME/conf/context.xml 文件（区别与 server.xml，该文件修改后，tomcat 将会自动重新加载该文件，而无需重启 tomcat）  
            在 `/` 节点下，添加 Context 配置信息（同[方法一](#config-datasource-method-1)的 Context 配置信息）
        - 方法三：编辑 $CATALINA_HOME/conf/server.xml 文件（不推荐，server.xml 是不可动态重加载的资源，即修改后必须重启 tomcat 才会生效）  
            在 `/Server/Service/Engine` 节点下，添加 Context 配置信息（同[方法一](#config-datasource-method-1)的 Context 配置信息）
    2. 添加数据库驱动引用（如 mysql-connector-java-6.0.6.jar）至 $CATALINA_HOME/lib 中

- 在 webapp 中引用 Tomcat 中配置的数据源
    1. [在 Tomcat 中配置数据源](#config-datasource)
    2. 编辑 $CATALINA_HOME/webapps/*/WEB-INF/web.xml 文件，在 `/web-app` 节点下，添加 resource-ref 配置如下
        ```xml
        <resource-ref>
            <res-ref-name>jdbc/solr</res-ref-name>
            <res-type>javax.sql.DataSource</res-type>
            <res-auth>Container</res-auth>
        </resource-ref>
        ```
    3. 添加数据库驱动引用（如 mysql-connector-java-6.0.6.jar）至 $CATALINA_HOME/webapps/*/WEB-INF/lib 目录下
    4. 使用
        ```java
        <%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
        <%@ page import="java.sql.*,javax.sql.*,javax.naming.*" %>

        <!DOCTYPE HTML>
        <html>
            <head>
                <title>JNDI Test</title>
            </head>
            <body>
                <%
                    Connection connection = null;
                    try {
                        // 初始化名称查找上下文
                        Context ctx = new InitialContext();

                        // 通过 JNDI 名称找到 DataSource，格式为 java:comp/env/DataSourceName
                        DataSource ds = (DataSource) ctx.lookup("java:comp/env/jdbc/solr");

                        // 通过 DataSource 取得一个连接
                        connection = ds.getConnection();

                        // 操作数据库，实现增删改查等
                        // ...
                    } catch (NamingException e) {
                        e.printStackTrace();
                    } catch (SQLException e) {
                        e.printStackTrace();
                    } finally {
                        // 关闭数据库，关闭的时候是将连接放回到连接池之中
                        connection.close();
                    }
                %>
            </body>
        </html>
        ```

- 认证配置  
    Tomcat 支持四种认证方式：[BASIC（基本验证）、DIGEST（摘要验证）、FORM（基于表单的验证）、CLIENT-CERT（客户证书验证）](http://www.cnblogs.com/dengyungao/p/7542241.html)
    - BASIC（基本验证）
        1. <span id="config-security-constraint-for-basic">配置 security-constraint</span>  
            编辑 $CATALINA_HOME/webapps/*/WEB-INF/web.xml 文件，在 `/web-app` 节点下，添加配置如下
            ```xml        
            <security-constraint>
                <web-resource-collection>
                    <web-resource-name>auth field</web-resource-name>
                    <url-pattern>/*</url-pattern>
                    <!-- <url-pattern>/ps/*</url-pattern> -->
                    <!-- <http-method>GET</http-method> -->
                </web-resource-collection>
                <auth-constraint>
                    <role-name>admin</role-name>
                </auth-constraint>
            </security-constraint>
            <login-config>
                <auth-method>BASIC</auth-method>
            </login-config>
            ```
            > 其中
            > - url-pattern 配置匹配的 url，可配置多个进行细化管理
            > - http-method 配置 http 请求的方法，可配置多个，值可为 GET、POST、PUT、DELETE
            > - role-name 配置角色名称
            > - auth-method 配置认证方式，值可为 BASIC（基本验证）、DIGEST（摘要验证，只支持 md5 算法？）以及（不常用）FORM（基于表单的验证）、CLIENT-CERT（客户证书验证）
        2. <span id="config-username-and-passwd-for-basic">配置用户名、密码</span>
            - 方法一：使用配置文件 $CATALINA_HOME/conf/tomcat-users.xml 管理用户名、密码、角色  
                1. 配置 Resource（默认情况下，tomcat 已存在该配置，可跳过）  
                    编辑 $CATALINA_HOME/conf/server.xml 文件，在 `/Server/GlobalNamingResources` 节点下，配置 Resource 如下
                    ```xml
                    <Resource name="UserDatabase" auth="Container"
                        type="org.apache.catalina.UserDatabase"
                        description="User database that can be updated and saved"
                        factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
                        pathname="conf/tomcat-users.xml" />
                    ```
                2. <span id="config-tomcat-users-for-basic">配置用户角色、用户名、密码</span>  
                    编辑 tomcat-users.xml 文件，在 `/tomcat-users` 节点下，配置角色、用户如下
                    ```xml
                    <role rolename="admin" />

                    <!-- 默认未配置 CredentialHandler，则使用明文密码，不推荐 -->
                    <!-- <user username="admin" password="admin" roles="admin" /> -->

                    <!-- 配置 MessageDigestCredentialHandler 指定使用 MD5 摘要算法 -->
                    <!-- <user username="admin" password="21232f297a57a5a743894a0e4a801fc3" roles="admin" /> -->
                    <!-- <user username="admin" password="03552ca004c4d197a06f769e36825b3666b27b10a74a035089f27cf48d680f27$1$9299434ea39a82833950ef4c6d8d81af" roles="admin" /> -->

                    <!-- 配置 MessageDigestCredentialHandler 指定使用 SHA-256 摘要算法 -->
                    <!-- <user username="admin" password="8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918" roles="admin" /> -->

                    <!-- 配置 SecretKeyCredentialHandler 指定使用 PBKDF2WithHmacSHA512 算法 -->
                    <user username="admin" password="9fac50c23deceebc5884015289b111c9$100000$2087629b338449c8446e11ad5d0f8a0a8264c641af10b0ba4ee7a3a5f2df5ae5" roles="admin" />
                    ```
                    > 其中，若指定摘要算法，则 password 需要进行摘要处理，可使用 `digest.(sh|bat) -a $algorithm (-s 0)? $password` 命令获取密码（如 "admin"）的摘要值
                    >
                    > - 指定使用 MD5 摘要算法  
                    >   `./$CATALINA_HOME/bin/digest.sh -a md5 -s 0 admin`  
                    >   <font color="grey">返回结果如 admin:21232f297a57a5a743894a0e4a801fc3</font>
                    >   `./$CATALINA_HOME/bin/digest.sh -a md5 admin`  
                    >   <font color="grey">返回结果如 admin:03552ca004c4d197a06f769e36825b3666b27b10a74a035089f27cf48d680f27$1$9299434ea39a82833950ef4c6d8d81af</font>
                    >
                    > - 指定使用 SHA-256 摘要算法  
                    >   `./$CATALINA_HOME/bin/digest.sh -a sha-256 -s 0 admin`  
                    >   <font color="grey">返回结果如 admin:8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918</font>
                    >
                    > - 指定使用 PBKDF2WithHmacSHA512 算法  
                    >   `./$CATALINA_HOME/bin/digest.sh -a PBKDF2WithHmacSHA512 -i 100000 -s 16 -k 256 -h org.apache.catalina.realm.SecretKeyCredentialHandler admin`  
                    >   <font color="grey">返回结果如 admin:9fac50c23deceebc5884015289b111c9$100000$2087629b338449c8446e11ad5d0f8a0a8264c641af10b0ba4ee7a3a5f2df5ae5</font>
                3. <span id="config-credentialHandler-for-basic">[配置 CredentialHandler（可选，仅适用于指定使用摘要算法，其他方式可跳过该步骤）](https://stackoverflow.com/questions/32178822/tomcat-understanding-credentialhandler)</span>  
                    编辑 server.xml 文件，在 `/Server/Service/Engine/Realm[@className="org.apache.catalina.realm.LockOutRealm"]/Realm[@className="org.apache.catalina.realm.UserDatabaseRealm"]` 节点下，配置 CredentialHandler 如下
                    ```xml
                    <!-- 配置 MessageDigestCredentialHandler 指定使用 MD5 摘要算法 -->
                    <!-- <CredentialHandler className="org.apache.catalina.realm.MessageDigestCredentialHandler" algorithm="MD5" /> -->
                    
                    <!-- 配置 MessageDigestCredentialHandler 指定使用 SHA-256 摘要算法 -->
                    <!-- <CredentialHandler className="org.apache.catalina.realm.MessageDigestCredentialHandler" algorithm="SHA-256" /> -->

                    <!-- 配置 SecretKeyCredentialHandler 指定使用 PBKDF2WithHmacSHA512 算法 -->
                    <CredentialHandler className="org.apache.catalina.realm.SecretKeyCredentialHandler"
                        algorithm="PBKDF2WithHmacSHA512"
                        iterations="100000"
                        keyLength="256"
                        saltLength="16" />
                    ```
            - 方法二：[使用数据库管理用户名、密码、角色](http://lsz1023-126-com.iteye.com/blog/2072066)  
                1. <span id="config-db-users">准备数据库，如下</span>
                    ```sql
                    create database db_users;
                    
                    use db_users;

                    create table t_users (
                        username varchar(20) not null,
                        password varchar(64) not null,
                        primary key(username)
                    );

                    create table t_roles (
                        username varchar(20) not null,
                        rolename varchar(20) not null,
                        primary key(username, rolename)
                    );

                    insert into t_users values('zhangsan', '123456');
                    insert into t_users values('lisi', '123456');

                    insert into t_roles values('zhangsan', 'admin');
                    insert into t_roles values('zhangsan', 'tomcat');
                    insert into t_roles values('lisi', 'tomcat');

                    commit;
                    ```
                2. <span id="config-realm">配置 Context</span>  
                    如编辑 $CATALINA_HOME/conf/context.xml 文件，在 `/Context` 节点下添加 Realm 配置如下
                    ```xml
                    <Realm className="org.apache.catalina.realm.JDBCRealm"
                        connectionName="test" connectionPassword="123456"
                        connectionURL="jdbc:mysql://192.168.128.137:3306/db_users"
                        driverName="com.mysql.jdbc.Driver"
                        userTable="t_users"
                        userRoleTable="t_roles"
                        userNameCol="username"
                        userCredCol="password"
                        roleNameCol="rolename"/>
                    ```
    - DIGEST（摘要验证，当前 tomcat 版本仅支持 md5 算法）
        1. 配置 security-constraint  
            参考配置 [BASIC](#config-security-constraint-for-basic)  
            其中 auth-method 需要修改为 DIGEST，且需要定义 realm-name，如下
            ```xml
            <security-constraint>
                ...
            </security-constraint>
            <login-config>
                <auth-method>DIGEST</auth-method>
                <realm-name>app-realm</realm-name>
            </login-config>
            ```
        2. 配置用户名、密码  
            参考配置 [BASIC](#config-username-and-passwd-for-basic)  
            其中**方法一：使用配置文件**中的  
            - 步骤 [**配置用户角色、用户名、密码**](#config-tomcat-users-for-basic)，配置用户信息需要修改如下
                ```xml
                <user username="admin" password="817440541acc5a8e3ea991b2d43533f3" roles="admin" />
                ```
                > 其中，若指定摘要算法，则 password 需要进行摘要处理，区别于 BASIC 方式，DIGEST 方式需使用 `digest.(sh|bat) -a $algorithm (-s 0)? $username:$realm-name:$password` 命令获取密码（如 "admin"，同时用户名为 admin、realm-name 为 app-realm）的摘要值
                > - 指定使用 MD5 摘要算法  
                >   `./$CATALINA_HOME/bin/digest.sh -a md5 -s 0 admin:app-realm:admin`  
                >   <font color="grey">返回结果如 admin:app-realm:admin:817440541acc5a8e3ea991b2d43533f3</font>
            - 步骤 [**配置 CredentialHandler**](#config-credentialHandler-for-basic)，需要使用 MD5 摘要算法，如下
                ```xml
                <CredentialHandler className="org.apache.catalina.realm.MessageDigestCredentialHandler" algorithm="MD5" />
                ```
    - FORM（基于表单的验证）
        1. 配置 security-constraint  
            参考配置 [BASIC](#config-security-constraint-for-basic)  
            其中 login-config 配置需要指定登录页面如下  
            ```xml
            <security-constraint>
                ...
            </security-constraint>
            <login-config>
                <auth-method>FORM</auth-method>
                <form-login-config>
                    <form-login-page>/form/login.html</form-login-page>
                    <form-error-page>/form/error.html</form-error-page>
                </form-login-config>
            </login-config>
            ```
        2. 配置用户名、密码  
            参考配置 [BASIC](#config-username-and-passwd-for-basic)
    - CLIENT-CERT（客户证书验证）
        1. 生成证书
            1. 生成服务器证书容器  
                `keytool -validity 36500 -genkey -v -alias server -keyalg RSA -keystore server.jks -dname "CN=192.168.0.104,OU=xx,O=xx,L=shanghai,ST=shanghai,c=cn" -storepass passwd01 -keypass passwd01`
            2. 生成信赖的客户端证书容器  
                `keytool -validity 36500 -genkey -v -alias server_trust -keyalg RSA -keystore server_trust.jks -dname "CN=192.168.0.104,OU=xx,O=xx,L=shanghai,ST=shanghai,c=cn" -storepass passwd02 -keypass passwd02`
            3. 生成客户端用秘钥对  
                `keytool -validity 36500 -genkeypair -v -alias client_01 -keyalg RSA -storetype PKCS12 -keystore client_01.p12 -dname "CN=client_01,OU=xx,O=xx,L=shanghai,ST=shanghai,c=cn" -storepass passwd03 -keypass passwd03`  
                导出客户端用证书  
                `keytool -export -v -alias client_01 -keystore client_01.p12 -storetype PKCS12 -storepass passwd03 -rfc -file client_01.cer`
            4. 将客户端用的证书导入至服务端信赖的客户端的证书容器  
                `keytool -import -v -alias client_01 -file client_01.cer -keystore server_trust.jks -storepass passwd02`
        2. 配置 https
            编辑 $CATALINA_HOME/conf/server.xml，配置 `/Server/Service/Connector[@port="8443"]` 节点如下  
            ```xml
            <Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
                       maxThreads="150" scheme="https" secure="true"
                       clientAuth="true" sslProtocol="TLS"
                       keystoreFile="${catalina.base}/conf/server.jks" keystorePass="passwd01"
                       truststoreFile="${catalina.base}/conf/server_trust.jks" truststorePass="passwd02" />
            ```
        3. 设置强制 ssl 访问  
            编辑 $CATALINA_HOME/*/web.xml，在 `/web-app` 节点下添加配置 security-constraint 如下
            ```xml
            <!-- 强制SSL，即http请求自动跳转成https --> 
            <security-constraint>
                <web-resource-collection>
                    <web-resource-name>SSL</web-resource-name>
                    <url-pattern>/*</url-pattern><!-- 全站使用SSL -->
                </web-resource-collection>
                <user-data-constraint>
                    <description>SSL required</description>
                    <!-- CONFIDENTIAL: 确保传输数据不被修改，不能被查看 -->
                    <!-- INTEGRAL: 确保传输数据不被修改 -->
                    <!-- NONE: 不做特殊限制-->
                    <transport-guarantee>CONFIDENTIAL</transport-guarantee>
                </user-data-constraint>
            </security-constraint>
            ```
        4. 客户端PC导入上面生成的 client_01.cer 以及 client_01.p12 后，重启浏览器，即可访问。（初次访问会弹出选择证书的对话框，选中导入的cer证书即可）

- 黑白名单配置
    1. 编辑 $CATALINA_HOME/conf/server.xml 文件，在 `/Server/Service/Engine` 节点下、或在 `/Server/Service/Engine/Host` 节点下、或在 `/Server/Service/Engine/Host/Context` 节点下，添加 Valve 配置如下
        ```xml
        <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="" deny="10.17.23.19[1-3]|10.54.233.1(1[6-9]|2[0-6])|62.138.2.(209|12(2|3)|21(3|4))|10.151.(149.222|42.61)|10.239.228.202" denyStatus="403" />
        ```
        其中 allow 为白名单，deny 为黑名单，支持正则配置匹配，denyStatus 为黑名单拜访的 http 状态码

- 虚拟主机配置
    1. 编辑 $CATALINA_HOME/conf/server.xml 文件，在 `/Server/Service/Engine` 节点下，添加一个或多个 Host 配置如下
        ```xml
        <Host name="www.simple.com"  appBase="webapps" unpackWARs="false" autoDeploy="false">
        </Host>

        <Host name="localhost"  appBase="webapps" unpackWARs="false" autoDeploy="false">
            <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs" prefix="localhost_access_log" suffix=".txt" pattern="%h %l %u %t &quot;%r&quot; %s %b" />

            <Context path="/solr" docBase="solr" reloadable="true"/>
        </Host>
        ```
    > - 启动时项目重复加载的问题及解决办法  
    >     <font color="grey">Tomcat 在加载工程时首先会根据 `//Context` 配置内容生成第一个 StandardContext 对象，并加载一次项目。加载完成后会根据 > `//Host/appBase`（默认不填则为 webapps）配置内容生成第二个 StandardContext，并再加载一次相应的工程（或 war 包）。</font>  
    >     如配置
    >     ```xml
    >     <Host name="localhost"  appBase="webapps" unpackWARs="false" autoDeploy="false">
    >         <Context path="/" docBase="solr" reloadable="true"/>
    >     </Host>
    >     ```
    >     既配置 Context，又在 webapps 目录下放置了工程目录，Tomcat 针对同一项目 solr 生成两个 StandardContext 是因为其名称不同，因而当作是两个 Context，> 所以加载了两次。第一个 StandardContext 所对应的名称是由标签 Context 的配置来决定的，而第二个 StandardContext 的名称是由项目名称 "/solr" 决定的。>   
    >     <font color="green">解决方法：
    >     - 方法一：
    >         将对应的工程目录如 $CATALINA_HOME/webapps/solr 重命名为 $CATALINA_HOME/webapps/ROOT
    >     - 方法二：
    >         编辑 server.xml，配置 `//Context@path` 为任意名称如 /solr
    >     - 方法三：
    >         编辑 server.xml，配置 `//Context@name` 为项目名称如 /solr（其中根目录符号 "/" 为必须）
    >
    >     </font>

- [配置并使用内置的集群功能](https://tomcat.apache.org/tomcat-8.5-doc/cluster-howto.html)
    1. 配置 web.xml，开启集群功能（web 工程中的 web.xml 文件中需要配置有 <distributable/> 这个元素才能使用 tomcat 内置的 cluster 功能）
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <web-app ...>
            ...
            <distributable/>
        </web-app>
        ```
    2. 配置 server.xml，在 Engine 节点下配置集群组播地址（见 Cluster 节点及其子节点配置，注意两个 tomcat 中的 path 必须一致，不然会报错(Context manager doesn't exist)。只有 path 一致，两个 tomcat 才能同步 session）
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <Server ...>
            ...
            <Service name="Catalina">
                ...
                <Engine name="Catalina" defaultHost="localhost">
                    <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
                                channelSendOptions="8">
                        <Manager className="org.apache.catalina.ha.session.DeltaManager"
                                expireSessionsOnShutdown="false"
                                notifyListenersOnReplication="true"/>
                        <Channel className="org.apache.catalina.tribes.group.GroupChannel">
                        <Membership className="org.apache.catalina.tribes.membership.McastService"
                                    address="228.0.0.4"
                                    port="45564"
                                    frequency="500"
                                    dropTime="3000"/>
                        <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
                                    address="auto"
                                    port="4000"
                                    autoBind="100"
                                    selectorTimeout="5000"
                                    maxThreads="6"/>
                        <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">
                            <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
                        </Sender>
                        <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>
                        <Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatchInterceptor"/>
                        </Channel>
                        <Valve className="org.apache.catalina.ha.tcp.ReplicationValve"
                                filter=""/>
                        <Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>
                        <Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"
                                tempDir="/tmp/war-temp/"
                                deployDir="/tmp/war-deploy/"
                                watchDir="/tmp/war-listen/"
                                watchEnabled="false"/>
                        <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>
                    </Cluster>
                    ...
                </Engine>
            </Service>
        </Server>
        ```

<details>
<summary>安全加固</summary>

- <span id="security-datasource">[安全加固：Tomcat 数据源连接池加密 password](http://www.cnblogs.com/benefitworld/p/5832668.html)</span>
    1. 构建 DataSourceFactory 包
        1. 切换目录至 $CATALINA_HOME/temp 目录下，创建目录  
            `mkdir -p ./pers/sunke/tomcat/dbcp`
        2. 新建文件 EncryptedDataSourceFactory.java  
            `touch ./pers/sunke/tomcat/dbcp/EncryptedDataSourceFactory.java`  
            编辑内容如下
            ```java
            package pers.sunke.tomcat.dbcp;

            import java.security.InvalidAlgorithmParameterException;
            import java.security.InvalidKeyException;
            import java.security.NoSuchAlgorithmException;
            import java.util.Base64;
            import java.util.Enumeration;
            import java.util.Hashtable;
            import java.util.ResourceBundle;

            import javax.crypto.BadPaddingException;
            import javax.crypto.Cipher;
            import javax.crypto.IllegalBlockSizeException;
            import javax.crypto.NoSuchPaddingException;
            import javax.crypto.spec.IvParameterSpec;
            import javax.crypto.spec.SecretKeySpec;
            import javax.naming.Context;
            import javax.naming.Name;
            import javax.naming.RefAddr;
            import javax.naming.Reference;
            import javax.naming.StringRefAddr;

            import org.apache.tomcat.dbcp.dbcp2.BasicDataSourceFactory;

            public class EncryptedDataSourceFactory extends BasicDataSourceFactory {

                private static final String ALGORITHM = "AES/CBC/PKCS5Padding";
                private static final String KEYALGORITHM = "AES";

                // length of key should be 16 byte for 128 bit, 32 byte for 256 bit.
                private static final String KEY = "2QKH3XfuIKALMwnk";
                // length of iv must be 16 byte.(equal to the length of a data block in aes: 16 byte or 128 bit)
                private static final String IV = "RAk0ygQXIX2ZpMQH";

                private static final String PROP_PASSWORD = "password";

                @Override
                public Object getObjectInstance(Object obj, Name name, Context nameCtx, Hashtable<?, ?> environment) throws Exception {
                    try {
                        if (obj instanceof Reference) {
                            Reference ref = (Reference) obj;
                            StringRefAddr passwordRefAddr = (StringRefAddr) ref.get(PROP_PASSWORD);
                            if (null != passwordRefAddr) {
                                String encryptedPwd = (String) passwordRefAddr.getContent();
                                String cleartextPwd = decrypt(encryptedPwd);
                                int index = find(PROP_PASSWORD, ref);
                                if (index >= 0) {
                                    ref.remove(index);
                                    ref.add(index, new StringRefAddr(PROP_PASSWORD, cleartextPwd));
                                }
                            }
                        }
                    } catch (Exception e) {
                        System.err.println("Failed to decrypt password. Please check DataSource definition.");
                        throw e;
                    }
                    return super.getObjectInstance(obj, name, nameCtx, environment);
                }

                private int find(String addrType, Reference ref) throws Exception {
                    Enumeration<RefAddr> enu = ref.getAll();
                    for (int i = 0; enu.hasMoreElements(); i++) {
                        RefAddr addr = (RefAddr) enu.nextElement();
                        if (addr.getType().equals(addrType))
                            return i;
                    }
                    return -1;
                }

                public static String encrypt(String source) throws NoSuchAlgorithmException, NoSuchPaddingException, InvalidKeyException, IllegalBlockSizeException, BadPaddingException, InvalidAlgorithmParameterException {
                    Cipher cipher = Cipher.getInstance(ALGORITHM);
                    cipher.init(Cipher.ENCRYPT_MODE, new SecretKeySpec(KEY.getBytes(), KEYALGORITHM), new IvParameterSpec(IV.getBytes()));
                    return new String(Base64.getEncoder().encode(cipher.doFinal(source.getBytes())));
                }

                public static String decrypt(String encryptSource) throws NoSuchAlgorithmException, NoSuchPaddingException, InvalidKeyException, IllegalBlockSizeException, BadPaddingException, InvalidAlgorithmParameterException {
                    Cipher cipher = Cipher.getInstance(ALGORITHM);
                    cipher.init(Cipher.DECRYPT_MODE, new SecretKeySpec(KEY.getBytes(), KEYALGORITHM), new IvParameterSpec(IV.getBytes()));
                    return new String(cipher.doFinal(Base64.getDecoder().decode(encryptSource.getBytes())));
                }

                public static void main(String[] args) throws InvalidKeyException, NoSuchAlgorithmException, NoSuchPaddingException, IllegalBlockSizeException, BadPaddingException, InvalidAlgorithmParameterException {
                    try {
                        if (args.length == 1) {
                            System.out.println(encrypt(args[0]));
                        } else if (args.length == 2 && "-e".equals(args[0])) {
                            System.out.println(encrypt(args[1]));
                        } else if (args.length == 2 && "-d".equals(args[0])) {
                            System.out.println(decrypt(args[1]));
                        } else {
                            ResourceBundle bundle = ResourceBundle.getBundle("messages");
                            String usageDescription = bundle.getString("usage.description");
                            System.out.println(usageDescription);
                        }
                    } catch (Exception e) {
                        System.out.println("Failed to encrypt or decrypt.");
                        e.printStackTrace();
                    }
                }
            }
            ```
        3. 编译  
            `javac -cp ../lib/tomcat-dbcp.jar ./pers/sunke/tomcat/dbcp/EncryptedDataSourceFactory.java`
        4. 打包  
            `jar cvf ../lib/tomcat-dbcp-patch.jar ./pers/sunke/tomcat/dbcp/EncryptedDataSourceFactory.class`
        5. 切换目录至 $CATALINA_HOME/lib  
            `cd ../lib`
        5. 生成密文  
            `java -cp "tomcat-dbcp-patch.jar;tomcat-dbcp.jar;../bin/tomcat-juli.jar" pers.sunke.tomcat.dbcp.EncryptedDataSourceFactory -e 123456`
            > 解密密文  
            > `java -cp "tomcat-dbcp-patch.jar;tomcat-dbcp.jar;../bin/tomcat-juli.jar" pers.sunke.tomcat.dbcp.EncryptedDataSourceFactory -d UkLthpB/fB7z6yZFUnAE1A==`

        6. 配置数据源如下
            ```xml
            <?xml version='1.0' encoding='utf-8'?>
            <Context path="/ds01" docBase="app01">
                <Resource name="jdbc/ds01" type="javax.sql.DataSource" auth="Container" driverClassName="com.mysql.jdbc.Driver" factory="pers.sunke.tomcat.dbcp.EncryptedDataSourceFactory" url="jdbc:mysql://127.0.0.1:3306/db01" validationQuery="select 1 from dual" username="test" password="UkLthpB/fB7z6yZFUnAE1A==" />
            </Context>
            ```

- 安全加固：Tomcat 鉴权数据源中的用户密码加密（PBKDF2WithHmacSHA256）
    1. [准备数据源](#config-db-users)，修改字段长度
        ```sql
        alter table t_users modify password varchar(600) not null;

        update t_users set password = '10000:97b10f4dd3f7d5444072b7e57175f069ec1e3b25abaf71386a99fe532212544f:e99cbfcd15d23e93687ccf80d2639746fec83a908062f8b48385908665c8f6c07e65dbdd9b5db62861047a15b35c8653b03f16ac72cef661e4777193056d12c19ec68106fd9f5a9f7d4a6d83029c69ffa54fe795e6ff0b7c3bab97f5c029704e08a6cc4567685c7a8ba42315a4937bc4471a6b280669b9b44e12719beee92e934ff8c0064ef7d08e268c95b73bb506edb484b670e144d25be759485f2be6f50dc4fbe3c119ce8524da7d30a689b8c3a633619acc41e2ebcc431a5ccaca1084910f165812dfcc71889367f8abc1c91c1cc0fcf9330d804a19ace77699190233a24bf87f5126ebc80ef2ef67302ed37fc3bd217aa3d1c7a3e24a0e80fa7b36e530' where username = 'zhangsan';
        ```
    2. 继承重写 org.apache.catalina.realm.JDBCRealm，如下
        ```java
        package pers.sunke.tomcat.realm;
        
        import java.math.BigInteger;
        import java.security.NoSuchAlgorithmException;
        import java.security.Principal;
        import java.security.SecureRandom;
        import java.security.spec.InvalidKeySpecException;
        import java.security.spec.KeySpec;

        import javax.crypto.SecretKeyFactory;
        import javax.crypto.spec.PBEKeySpec;

        import org.apache.catalina.realm.JDBCRealm;

        public class EncryptedJDBCRealm extends JDBCRealm {

            private static final String ALGORITHM = "PBKDF2WithHmacSHA256";
            private static final int ITERATIONCOUNT = 10000;
            private static final int KEYLENGTH = 256;
            private static final int SALTLENGTH = 32;

            @Override
            public synchronized Principal authenticate(String username, String credentials) {
                try {
                    byte[] salt = getSalt(getPassword(username));
                    String credentialsHashed = hash(credentials, salt);
                    return super.authenticate(username, credentialsHashed);
                } catch (NoSuchAlgorithmException | InvalidKeySpecException e) {
                    e.printStackTrace();
                    return null;
                }
            }

            public byte[] generateSalt() {
                SecureRandom random = new SecureRandom();
                byte[] salt = new byte[SALTLENGTH];
                random.nextBytes(salt);
                return salt;
            }

            public byte[] getSalt(String code) {
                String[] params = code.split(":");
                if (3 == params.length) {
                    byte[] salt = fromHex(params[1]);
                    return salt;
                } else {
                    System.err.println("Error cipher code format: " + code);
                    return null;
                }
            }

            public String hash(String password, byte[] salt) throws NoSuchAlgorithmException, InvalidKeySpecException {
                SecretKeyFactory factory = SecretKeyFactory.getInstance(ALGORITHM);
                KeySpec spec = new PBEKeySpec(password.toCharArray(), salt, ITERATIONCOUNT, KEYLENGTH * 8);
                byte[] key = factory.generateSecret(spec).getEncoded();
                return String.format("%d:%s:%s", ITERATIONCOUNT, toHex(salt), toHex(key));
            }

            private String toHex(byte[] array) {
                BigInteger bi = new BigInteger(1, array);
                String hex = bi.toString(16);
                int paddingLength = (array.length * 2) - hex.length();
                if (paddingLength > 0)
                    return String.format("%0" + paddingLength + "d", 0) + hex;
                else
                    return hex;
            }

            private byte[] fromHex(String hex) {
                byte[] binary = new byte[hex.length() / 2];
                for (int i = 0; i < binary.length; i++) {
                    binary[i] = (byte) Integer.parseInt(hex.substring(2 * i, 2 * i + 2), 16);
                }
                return binary;
            }

            public static void main(String[] args) {
                EncryptedJDBCRealm realm = new EncryptedJDBCRealm();
                try {
                    if (args.length == 1) {
                        System.out.println(realm.hash(args[0], realm.generateSalt()));
                    } else if (args.length == 2 && "-h".equals(args[0])) {
                        System.out.println(realm.hash(args[1], realm.generateSalt()));
                    } else if (args.length == 3 && "-h".equals(args[0])) {
                        System.out.println(realm.hash(args[1], realm.fromHex(args[2])));
                    } else {
                        System.out.println("Usage:");
                        System.out.println("\tjava [...] [password]");
                        System.out.println("\tjava [...] -h [password]");
                        System.out.println("\tjava [...] -h [password] [salt]");
                    }
                } catch (Exception e) {
                    System.out.println("Failed to hash.");
                    e.printStackTrace();
                }
            }
        }
        ```
    3. 打包部署，可参考[数据源安全加固](#security-datasource)中相关步骤，其中编译、打包、生成密钥 code 命令如下    
        `javac -cp "../lib/catalina.jar" ./pers/sunke/tomcat/realm/EncryptedJDBCRealm.java`  
        `jar cvf ../lib/catalina-patch.jar ./pers/sunke/tomcat/realm/EncryptedJDBCRealm.class`  
        `java -cp "catalina-patch.jar;catalina.jar;tomcat-util.jar;../bin/tomcat-juli.jar" pers.sunke.tomcat.realm.EncryptedJDBCRealm -h 123456`
    4. 修改[相关配置文件中 Realm 节点](#config-realm)的 className 为 pers.sunke.tomcat.realm.EncryptedJDBCRealm
    > - 配置文件中 connectionPassword 的安全加固，需重写 getConnectionPassword 方法，方法的实现可参考[数据源安全加固](#security-datasource) 中相关实现
    > - 使用 tomcat-users.xml 数据源鉴权的，需继承重写 org.apache.catalina.users.MemoryUserDatabaseFactory 对象，可参考[数据源安全加固](#security-datasource) 中相关实现

- 安全加固：Tomcat 基本安全配置
    1. 删除多余的工程项目  
        删除 $CATALINA_HOME/webapps 目录下所有不需要的工程项目（包括示例工程、webapp 管理工程，如 docs、example、host-manager、manager、ROOT）等项目（目录）
    2. 停止 Tomcat shutdown 端口  
        编辑 $CATALINA_HOME/conf/server.xml，修改 `/Server/@port` 的值为 -1，如下  
        ```xml
        <Server port="-1" shutdown="SHUTDOWN">
        ```
        > - 该端口（默认配置的端口号为 8005）用于远程放送命令（命令内容对应 `/Server/@shutdown` 的值，默认配置的命令内容为 "SHUTDOWN"）给 Tomcat 以停止服务，可通过配置端口号为 -1 来禁用该端口。  
        > - 通过 tcp 连接该端口并发送命令来停止 tomcat 服务
        >   - 方法一：使用 telnet 连接  
        >       `telnet 127.0.0.1 8005`  
        >       输入命令 SHUTDOWN 并回车，则 tomcat 停止服务
        >   - 方法二：使用 netcat 连接  
        >       `nc 127.0.0.1 8005`  
        >       输入命令 SHUTDOWN 并回车，则 tomcat 停止服务
    3. 禁用 AJP  
        编辑 $CATALINA_HOME/conf/server.xml，注释或删除 `/Server/Service/Connector[@protocol="AJP/1.3"]` 节点（即注释或删除使用 AJP/\*.\* 的 Connector 节点）
    4. Tomcat 降权启动  
        禁止以 root 特权用户启动 Tomcat，即 Tomcat 应以非 root 权限启动，应用部署目录权限和 Tomcat 服务启动用户分离，如 Tomcat 以 tomcat 用户启动，而部署应用的目录设置为 nobody 用户 750 读写权限
    5. 屏蔽 Tomcat 版本信息
        1. [自定义服务版本信息](https://jingyan.baidu.com/article/d2b1d102a9dc955c7e37d487.html)  
            编辑 $CATALINA_HOME/lib/catalina.jar!/org/apache/catalina/util/ServerInfo.properties，自定义修改 server.info 配置项如 "Web Server"，修改 server.number 配置项如 "1.0"
        2. 在 http 响应头中屏蔽 Tomcat 版本信息  
            编辑 $CATALINA_HOME/conf/server.xml，设置 `/Server/Service/Connector/@server` 的值为与 Tomcat 版本信息无关的名称，如 "Web Server"
    6. 关闭自动部署  
        编辑 $CATALINA_HOME/conf/server.xml，设置 `/Server/Service/Engine/Host/@unpackWARs` 和 `/Server/Service/Engine/Host/@autoDeploy` 的值为 false
    7. 自定义错误页面  
        编辑 $CATALINA_HOME/conf/web.xml 或 $CATALINA_HOME/webapps/*/WEB-INF/web.xml，在 `/web-app/` 节点下添加错误页面 error-page 配置，如下
        ```xml
        <error-page>
            <error-code>404</error-code>
            <location>/404.html</location>
        </error-page>
        <error-page>
            <error-code>500</error-code>
            <location>/500.html</location>
        </error-page>
        ```
    8. 启用 Cookie 的 HttpOnly 属性
        编辑 $CATALINA_HOME/conf/context.xml，设置 `/Context/@useHttpOnly` 为 true，如下
        ```xml
        <Context useHttpOnly="true">
            ...
        </Context>
        ```
    9. 开启日志审核，即 Tomcat 的访问日志（高版本默认已开启）  
        编辑 $CATALINA_HOME/conf/server.xml，设置或取消注释 `/Server/Service/Engine/Host/Valve[@className="org.apache.catalina.valves.AccessLogValve"]` 节点，如下
        ```xml
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
        ```
        > 其中，参数 pattern 指定记录日志的格式，各项的含义如下  
        > `%h` 远程主机名或 IP 地址。如果有 nginx 等反向代理服务器进行请求分发，该主机名或 IP 地址代表的是 nginx，否则代表的是客户端  
        > `%l` 远程逻辑用户名，一律是 "-"  
        > `%u` 授权的远程用户名，如果没有，则是 "-"  
        > `%t` 访问的时间  
        > `%r` 请求的第一行，即请求方法（get、post 等）、url、协议  
        > `%s` 响应状态，200、404 等  
        > `%b` 响应的数据量，不包括请求头，如果为 0，则是 "-"

    10. 禁止列目录（高版本默认已禁止）  
        编辑 web.xml，找到 `/web-app/servlet/init-param/param-name` 参数名为 listings 的节点，设置其参数值为 false，如下
        ```xml
        ...
        <init-param>
            <param-name>listings</param-name>
            <param-value>false</param-value>
        </init-param>
        ...
        ```
    11. 删除用户权限相关配置（高版本默认已禁止）  
        编辑 $CATALINA_HOME/conf/tomcat-users.xml，在 `/tomcat-users` 节点下，删除或注释所有 role、user 节点（即删除或注释用户、角色配置）

</details>

<details>
<summary>性能优化</summary>

- ~~性能优化：Tomcat 内存使用优化~~
    > 原则上应尽量避免 Full GC

    1. 编辑 $CATALINA_HOME/bin/catalina.sh（Linux），添加以下配置
        `JAVA_OPTS=$JAVA_OPT -server -Xms2048m -Xmx2048m` ~~-XX:PermSize=256m -XX:MaxPermSize=512m~~ `-Djava.awt.headless=true`
        > 其中，参数  
        > - `-Xms`、`-Xmx` 指定 jvm 占用最小和最大物理内存。一般两者配置一样大，可以避免内存不够用时申请内存的耗时
        > - `-XX:PermSize`、`-XX:MaxPermSize` 指定 Perm 内存大小  
        >   <font color="red">Java 8 中 PermSize 被 MetaspaceSize 代替，MetaspaceSize 共享 heap，不会再有"java.lang.OutOfMemoryError：PermGen space"，可以不设置</font>  
        >   <font color="grey">jvm 的内存分为两大类型：perm 和 generation。perm 区域存放的是 class 这些静态信息，一般默认为 64 MB。如果工程很大，有可能一启动就报错："out of memory permsize"；如果用 spring 框架的话很多类是动态反射加载的，运行一段时间也有可能出现此异常。此类情况下，设置 permsize 的大小即可。</font>

- 性能优化：Tomcat 启动参数优化（堆内存、栈内存、GC 日志、堆转储等）
    ```bash
    JAVA_OPTS="$JAVA_OPTS -Xms6144m -Xmx6144m -Xmn2384m -Xss512k -XX:MetaspaceSize=512M -XX:MaxMetaspaceSize=1024M -XX:+HeapDumpOnOutOfMemoryError -Djava.protocol.handler.pkgs=org.apache.catalina.webresources"
    JAVA_OPTS="$JAVA_OPTS -Xloggc:$CATALINA_BASE/logs/gc.log -XX:HeapDumpPath=$CATALINA_BASE/logs"
    JAVA_OPTS="$JAVA_OPTS -XX:+PrintGCDetails -XX:+UseParallelGC -XX:ParallelGCThreads=32"
    JAVA_OPTS="$JAVA_OPTS -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=10M"
    ```

- 性能优化：Tomcat 并发优化
    1. 编辑 $CATALINA_HOME/conf/server.xml，在 `/Server/Service/Connector` 节点中，配置如下
        ```xml
        <Connector ... 
                protocol="org.apache.coyote.http11.Http11NioProtocol"
                maxThreads="500"
                minSpareThreads="100"
                maxSpareThreads="200"
                acceptCount="200"
                enableLookups="false"
                ... />
        ```
        > 其中，参数  
        > - `protocol` 指定协议。[Http11NioProtocol（即 NIO 模式）较其它模式效率更高](#tomcat-protocol)
        > - `maxThreads` 指定当前可以同时处理的最大用户访问数
        > - `minSpareThreads`、`maxSpareThreads` 指定最小、最大空闲线程连接数，用于优化线程池
        > - `acceptCount` 指定当所有的线程已分配，仍然允许连接进来，但是处于等待状态的用户数。等待线程数 + 工作线程数 = 总的可最大连接数，如果超过此数，新的连接将不会被接受，并且将会产生一个 http 错误
        > - `enableLookups` 指定是否允许 DNS 反查，如果为 true，request.getRemoteHost 会执行 DNS 查找，反向解析 ip 对应域名或主机名，当没有这样需要的时候，可以将这个功能关闭，在一定程度上提高了 Tomcat 服务器的性能

- 性能优化：使用 APR 提高可伸缩性和性能

- 性能优化：[使用 log4j 管理 Tomcat 日志，并实现日志分割、备份清理](http://tomcat.apache.org/tomcat-8.0-doc/logging.html#Using-Log4j)
    1. 下载 log4j-1.2.17.jar  
        `curl -O https://mirrors.tuna.tsinghua.edu.cn/apache/logging/log4j/1.2.17/log4j-1.2.17.tar.gz && tar zxvf log4j-1.2.17.tar.gz apache-log4j-1.2.17/log4j-1.2.17.jar`  
        将 log4j-1.2.17.jar 复制到 $CATALINA_HOME/lib 目录中  
        `cp apache-log4j-1.2.17/log4j-1.2.17.jar $CATALINA_HOME/lib`
    2. 下载 tomcat-juli-adapters.jar  
        `curl -O http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.0.52/bin/extras/tomcat-juli-adapters.jar`  
        将 tomcat-juli-adapters.jar 复制到 $CATALINA_HOME/lib 目录中  
        `cp tomcat-juli-adapters.jar $CATALINA_HOME/lib`  
        下载 tomcat-juli.jar  
        `curl -O http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.0.52/bin/extras/tomcat-juli.jar`  
        将 tomcat-juli.jar 复制并替换到 $CATALINA_HOME/bin 目录中  
        `cp -f tomcat-juli.jar $CATALINA_HOME/bin`
    3. 在 $CATALINA_HOME/lib 目录下创建配置文件 log4j.properties  
        `vi $CATALINA_HOME/lib/log4j.properties`  
        编辑配置内容如下
        ```ini
        log4j.rootLogger = INFO, CATALINA
        # log4j.rootLogger = INFO, CATALINA, CONSOLE

        # Define catalina appender
        log4j.appender.CATALINA = org.apache.log4j.RollingFileAppender
        log4j.appender.CATALINA.File = ${catalina.base}/logs/catalina.out
        log4j.appender.CATALINA.Append = true
        log4j.appender.CATALINA.Encoding = UTF-8
        log4j.appender.CATALINA.MaxFileSize = 10MB
        log4j.appender.CATALINA.MaxBackupIndex = 10
        log4j.appender.CATALINA.layout = org.apache.log4j.PatternLayout
        log4j.appender.CATALINA.layout.ConversionPattern = %d [%t] %-5p %c- %m%n

        # Define catalina appender
        log4j.appender.LOCALHOST = org.apache.log4j.RollingFileAppender
        log4j.appender.LOCALHOST.File = ${catalina.base}/logs/localhost.log
        log4j.appender.LOCALHOST.Append = true
        log4j.appender.LOCALHOST.Encoding = UTF-8
        log4j.appender.LOCALHOST.MaxFileSize = 10MB
        log4j.appender.LOCALHOST.MaxBackupIndex = 10
        log4j.appender.LOCALHOST.layout = org.apache.log4j.PatternLayout
        log4j.appender.LOCALHOST.layout.ConversionPattern = %d [%t] %-5p %c- %m%n

        # Define console appender
        log4j.appender.CONSOLE = org.apache.log4j.ConsoleAppender
        log4j.appender.CONSOLE.Encoding = UTF-8
        log4j.appender.CONSOLE.layout = org.apache.log4j.PatternLayout
        log4j.appender.CONSOLE.layout.ConversionPattern = %d [%t] %-5p %c- %m%n

        # Configure which loggers log to which appenders
        log4j.logger.org.apache.catalina.core.ContainerBase.[Catalina].[localhost] = INFO, LOCALHOST
        ```
    4. 删除配置文件 logging.properties  
        `rm $CATALINA_HOME/conf/logging.properties`
    5. 启动/重启 Tomcat

</details>

<details>
<summary>常用第三方 war 应用</summary>

- <span id="setup-jenkins">使用 Jenkins 搭建持续集成环境</span>
    1. 下载  
        `wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war`
    2. 部署至 tomcat 中  
        `mv jenkins.war $CATALINA_HOME/webapps/`
    > Jenkins 将会在 user.home 目录下，即 ~/ 目录下创建 .jenkins 目录。可通过修改启动参数来指定 user.home，如  
    > `SET CATALINA_OPTS=-Duser.home="E:\apache-tomcat-8.0.47\temp"`

- 使用 [Probe](https://github.com/psi-probe/psi-probe) [监控 Tomcat](http://www.cnblogs.com/dancser/p/4450963.html)
    1. 下载 probe.war  
        `curl -L https://oss.sonatype.org/content/repositories/snapshots/com/github/psi-probe/psi-probe-web/3.0.0-SNAPSHOT/psi-probe-web-3.0.0-20180326.001732-103.war -o probe.war`
    2. 将 probe.war 复制到 webapps 目录下  
        `mv probe.war $CATALINA_HOME/webapps/`
    3. 添加用户账号  
        编辑 $CATALINA_HOME/conf/tomcat-users.xml，在 `/tomcat` 节点下添加用户配置如下
        ```xml
        <role rolename="manager" />
        <user username="admin" password="admin" roles="manager" />
        ```

</details>

- 参考
    > [详解 Tomcat 配置文件 server.xml](https://www.cnblogs.com/kismetv/p/7228274.html#title3-1)