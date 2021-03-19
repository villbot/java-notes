# Tomcat是怎么样支持HTTPS

当然既然是要支持HTTPS，那么首先我们肯定需要证书（关于HTTPS可以参考 [HTTP和HTTPS的关系.md](../network/HTTP和HTTPS的关系.md) ）

一般我们在生产环境使用的证书，都是通过CA机构进行购买的

当然我们也可以使用Jdk中的keytool工具生成免费的证书（这样生成的证书是不被浏览器所信任，HTTPS在进行TSL/SSL交互时会验证证书签发机构）

```sh
keytool -genkey -alias lagou -keyalg RSA -keystore lagou.keystore
```



拥有证书之后，在tomcat/conf/server.xml

```java
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"maxThreads="150" schema="https" secure="true" SSLEnabled="true">
    <SSLHostConfig>
        <Certificate certificateKeystoreFile="/Users/yingdian/workspace/servers/apache-tomcat-8.5.50/conf/lagou.keystore" certificateKeystorePassword="lagou123"type="RSA"/>
    </SSLHostConfig>
</Connector>
```

