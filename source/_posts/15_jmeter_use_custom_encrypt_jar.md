---
title: JMeter 使用自定义的加密库
date: 2020-03-21 22:08:00
tags: [Java, JMeter, JWK, Maven]
categories: [测试]
---

之前我们介绍过使用 JMeter 进行接口测试，这让我们专注于接口本身，而省去了很多编码的工作。但当 JMeter 提供的组件不能满足我们需求时，这个时候该怎么办呢？下面我们以一个登录接口为例，展开对这类场景的处理说明。

<!--more-->

## 提出问题

现在假设我们需要对一个登录接口进行测试，而此接口使用了 JSON Web Key(JWK) 验证(关于 JWK 验证这块可参考<sup>[[1]](#verifying_jwt)</sup>)。根据 JWK 验证，需要客户端进行如下处理：

1. 获取服务端返回的 JWK
2. 将 JWK 从 JSON 格式转换成 PEM 格式的公钥 (SSH 的公钥格式)
3. 根据 JWK 返回加密算法和公钥，对传参进行加密

通过上述处理，将明文的用户名和密码传递给登录接口进行验证。

## 分析问题

为了解决上面的问题，我们需要对服务端返回的 JWK 进行 PEM 格式转化，并使用公钥进行加密。这就需要我们思考 JMeter 目前提供的前置处理器是否能实现这一处理过程？若不能实现，是否社区已提供此类插件？若没有此类插件，自己该如何实现？

通过我们对 JMeter 前置处理器的了解，虽然前置处理器支持 Java, JavaScript 等编程语言，但要实现上述的处理，需要一些外部依赖库，目前前置处理器是处理不了。而在 [JMeter Plugins](https://jmeter-plugins.org/) 上也没有找到此类插件。剩下的只能我们自己实现此功能了。

还记得我们将 JMeter Plugins Manager 这个 jar 导入 JMeter 中，从而实现了插件管理的功能。我们也可以使用此机制， JMeter 支持 jar 包扩展，来实现自己定制的功能。

## 解决问题

### 实现 JWK 加密库

要自己从头实现 JWK 加密库是比较困难的。我们先搜索下社区是否有此类工作的项目。这里我在 GitHub 上搜索到 [joelicious/convertJWKSetToPEMSet](https://github.com/joelicious/convertJWKSetToPEMSet) 这个 Java 项目，它会获取 JWK URL 返回的 JSON，并将其转换成 PEM 格式的公钥。

基于此基础，我们只需要实现对参数的加密即可。以 RSA256 加密算法为例，我们需要实现使用公钥串对参数进行 RSA256 加密，具体加密实现参考：[RSA.java](https://github.com/jinhucheung/convertJWKSetToPEMSet/blob/master/src/main/java/com/redhat/utility/RSA.java)

之后连接转换公钥和加密参数模块，参考文件 [JWKSetEncrypt.java](https://github.com/jinhucheung/convertJWKSetToPEMSet/blob/master/src/main/java/com/redhat/utility/JWKSetEncrypt.java)

最后我们执行 Maven 打包出相应的 jar 包，此 jar 包可以在 [jinhucheung](https://github.com/jinhucheung/convertJWKSetToPEMSet)　获取到。

**Note**: JMeter Java 执行引擎使用 Bean Shell 脚本，不支持部分 Java 特性，如可变长参数。

### 使用 JWK 加密库

打开 JMeter 并载入相应的 `Test Plan`, 点击 `Add directory or jar to classpath` 浏览上一步打包出的 jar，如下图：

![jmeter-add-custom-jar](/images/posts/14_jmeter_use_custom_encrypt_jar/jmeter_add_custom_jar.png)

这里推荐使用相对路径引用 jar 包，避免因不同系统间的路径不同而带来的问题，而使用相应路径需要将 jar 包放在 `JMETER_HOME/lib/ext` 目录下。执行需要重启下 JMeter 以确保 jar 包被引用。

最后我们在登录接口中添加 `JSR223 PreProcessor` 组件，并选择 Java 引擎。之后编写脚本，引用 jar 包并使用，如下：

```java
import com.redhat.utility.JWKSetEncrypt;

log.info(JWKSetEncrypt.ping());

String[] logins = {vars.get("username"), vars.get("password")};
String[] encryptedTexts = JWKSetEncrypt.encrypt(vars.get("jkws_url"), logins);

vars.put("encrypted_username", encryptedTexts[0]);
vars.put("encrypted_password", encryptedTexts[1]);
```

## 参考

1. <a name='verifying_jwt'></a> [验证JSON Web 令牌](https://docs.aws.amazon.com/zh_cn/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-verifying-a-jwt.html)