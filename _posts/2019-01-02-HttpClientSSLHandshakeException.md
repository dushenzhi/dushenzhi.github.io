---
layout:     post
title:      https证书问题导致HttpClient访问异常的踩坑事故
subtitle:   HttpClient访问一些https站点报SSLHandshakeException问题解决方案
date:       2018-12-04
author:     dushenzhi
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - HttpClient
    - java
    - SSLHandshakeException
    - ssl
    - 异常
---

## 问题

最近在写网页爬虫，发现爬虫爬取数据时会漏一些站点，排查发现在用HttpClient访问部分https网站（如：https://www.gucci.cn/zh/）时，由于证书信任问题，会报如下错误：

``` java
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
	at sun.security.ssl.Alerts.getSSLException(Alerts.java:192)
	at sun.security.ssl.SSLSocketImpl.fatal(SSLSocketImpl.java:1949)
	at sun.security.ssl.Handshaker.fatalSE(Handshaker.java:302)
	at sun.security.ssl.Handshaker.fatalSE(Handshaker.java:296)
	at sun.security.ssl.ClientHandshaker.serverCertificate(ClientHandshaker.java:1509)
	at sun.security.ssl.ClientHandshaker.processMessage(ClientHandshaker.java:216)
	at sun.security.ssl.Handshaker.processLoop(Handshaker.java:979)
	at sun.security.ssl.Handshaker.process_record(Handshaker.java:914)
	at sun.security.ssl.SSLSocketImpl.readRecord(SSLSocketImpl.java:1062)
	at sun.security.ssl.SSLSocketImpl.performInitialHandshake(SSLSocketImpl.java:1375)
	at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1403)
	at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1387)
	at org.apache.http.conn.ssl.SSLConnectionSocketFactory.createLayeredSocket(SSLConnectionSocketFactory.java:394)
	at org.apache.http.conn.ssl.SSLConnectionSocketFactory.connectSocket(SSLConnectionSocketFactory.java:353)
	at org.apache.http.impl.conn.DefaultHttpClientConnectionOperator.connect(DefaultHttpClientConnectionOperator.java:141)
	at org.apache.http.impl.conn.PoolingHttpClientConnectionManager.connect(PoolingHttpClientConnectionManager.java:353)
	at org.apache.http.impl.execchain.MainClientExec.establishRoute(MainClientExec.java:380)
	at org.apache.http.impl.execchain.MainClientExec.execute(MainClientExec.java:236)
	at org.apache.http.impl.execchain.ProtocolExec.execute(ProtocolExec.java:184)
	at org.apache.http.impl.execchain.RetryExec.execute(RetryExec.java:88)
	at org.apache.http.impl.execchain.RedirectExec.execute(RedirectExec.java:110)
	at org.apache.http.impl.client.InternalHttpClient.doExecute(InternalHttpClient.java:184)
	at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:82)
	at com.mogujie.spiderman.fetch.http.HttpUtil.get(HttpUtil.java:171)
	at com.mogujie.spiderman.fetch.http.HttpUtil.get(HttpUtil.java:155)
	at com.mogujie.spiderman.fetch.http.HttpUtil.main(HttpUtil.java:335)
Caused by: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
	at sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:387)
	at sun.security.validator.PKIXValidator.engineValidate(PKIXValidator.java:292)
	at sun.security.validator.Validator.validate(Validator.java:260)
	at sun.security.ssl.X509TrustManagerImpl.validate(X509TrustManagerImpl.java:324)
	at sun.security.ssl.X509TrustManagerImpl.checkTrusted(X509TrustManagerImpl.java:229)
	at sun.security.ssl.X509TrustManagerImpl.checkServerTrusted(X509TrustManagerImpl.java:124)
	at sun.security.ssl.ClientHandshaker.serverCertificate(ClientHandshaker.java:1491)
	... 21 more
Caused by: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
	at sun.security.provider.certpath.SunCertPathBuilder.build(SunCertPathBuilder.java:146)
	at sun.security.provider.certpath.SunCertPathBuilder.engineBuild(SunCertPathBuilder.java:131)
	at java.security.cert.CertPathBuilder.build(CertPathBuilder.java:280)
	at sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:382)
	... 27 more
```



## 解决方案

环境是Apache HttpClient 4.5+ & Java8，可以采用如下方式解决这个问题:

``` java
SSLContext sslContext = SSLContexts.custom()
        .loadTrustMaterial((chain, authType) -> true).build();

SSLConnectionSocketFactory sslConnectionSocketFactory =
        new SSLConnectionSocketFactory(sslContext, new String[]
        {"SSLv2Hello", "SSLv3", "TLSv1","TLSv1.1", "TLSv1.2" }, null,
        NoopHostnameVerifier.INSTANCE);
CloseableHttpClient client = HttpClients.custom()
        .setSSLSocketFactory(sslConnectionSocketFactory)
        .build();
```



但是如果你的`HttpClient `是通过`ConnectionManager` 来创建连接的话，如下：

``` java
 PoolingHttpClientConnectionManager connectionManager = new 
         PoolingHttpClientConnectionManager();

 CloseableHttpClient client = HttpClients.custom()
            .setConnectionManager(connectionManager)
            .build();
```

这时配置**HttpClients.custom().setSSLSocketFactory(sslConnectionSocketFactory)**还是不能解决问题，因为这个配置是不生效的。

因为`HttpClient `是通过指定`connectionManager `来获取`connection `的，而`connectionManager`并没有指定`SSLConnectionSocketFactory`，解决这个问题的方法是就是在`connectionManager`中指定`SSLConnectionSocketFactory `就可以了，正确的方式代码如下所示：

``` java
PoolingHttpClientConnectionManager connectionManager = new 
    PoolingHttpClientConnectionManager(RegistryBuilder.
                <ConnectionSocketFactory>create()
      .register("http",PlainConnectionSocketFactory.getSocketFactory())
      .register("https", sslConnectionSocketFactory).build());

CloseableHttpClient client = HttpClients.custom()
            .setConnectionManager(connectionManager)
            .build();
```



参考链接：[https://stackoverflow.com/questions/1828775/how-to-handle-invalid-ssl-certificates-with-apache-httpclient](https://stackoverflow.com/questions/1828775/how-to-handle-invalid-ssl-certificates-with-apache-httpclient)