---
title: 谷粒商城之MD5加密
date: 2024-01-26 22:30:00
author: Mr zhu
categories: 谷粒商城
tags:
- 谷粒商城
---
## MD5加密概述

### MD5概述：

MD5消息摘要算法，属Hash算法一类。MD5算法对输入任意长度的消息进行运行，产生一个128位的消息摘要(32位的数字字母混合码)。

### MD5主要特点:

不可逆，相同数据的MD5值肯定一样，不同数据的MD5值不一样

(一个MD5理论上的确是可能对应无数多个原文的，因为MD5是有限多个的而原文可以是无数多个。比如主流使用的MD5将任意长度的“字节串映射为一个128bit的大整数。也就是一共有2^128种可能，大概是3.4*10^38，这个数字是有限多个的，而但是世界上可以被用来加密的原文则会有无数的可能性)

### MD5的性质：

1、压缩性：任意长度的数据，算出的MD5值长度都是固定的(相当于超损压缩)。

2、容易计算：从原数据计算出MD5值很容易。

3、抗修改性：对原数据进行任何改动，哪怕只修改1个字节，所得到的MD5值都有很大区别。

4、弱抗碰撞：已知原数据和其MD5值，想找到一个具有相同MD5值的数据（即伪造数据）是非常困难的。

5、强抗碰撞：想找到两个不同的数据，使它们具有相同的MD5值，是非常困难的。

虽说MD5有不可逆的特点

但是由于某些MD5破解网站，专门用来查询MD5码，其通过把常用的密码先MD5处理，并将数据存储起来，然后跟需要查询的MD5结果匹配，这时就有可能通过匹配的MD5得到明文，所以有些简单的MD5码是反查到加密前原文的。

为了让MD5码更加安全，涌现了很多其他方法，如加盐。 盐要足够长足够乱 得到的MD5码就很难查到。

### MD5用途：

1.防止被篡改：

1）比如发送一个电子文档，发送前，我先得到MD5的输出结果a。然后在对方收到电子文档后，对方也得到一个MD5的输出结果b。如果a与b一样就代表中途未被篡改。

2）比如我提供文件下载，为了防止不法分子在安装程序中添加木马，我可以在网站上公布由安装文件得到的MD5输出结果。

3）SVN在检测文件是否在CheckOut后被修改过，也是用到了MD5.

2.防止直接看到明文：

现在很多网站在数据库存储用户的密码的时候都是存储用户密码的MD5值。这样就算不法分子得到数据库的用户密码的MD5值，也无法知道用户的密码。（比如在UNIX系统中用户的密码就是以MD5（或其它类似的算法）经加密后存储在文件系统中。当用户登录的时候，系统把用户输入的密码计算成MD5值，然后再去和保存在文件系统中的MD5值进行比较，进而确定输入的密码是否正确。通过这样的步骤，系统在并不知道用户密码的明码的情况下就可以确定用户登录系统的合法性。这不但可以避免用户的密码被具有系统管理员权限的用户知道，而且还在一定程度上增加了密码被破解的难度。）

3.防止抵赖（数字签名）：

这需要一个第三方认证机构。例如A写了一个文件，认证机构对此文件用MD5算法产生摘要信息并做好记录。若以后A说这文件不是他写的，权威机构只需对此文件重新产生摘要信息，然后跟记录在册的摘要信息进行比对，相同的话，就证明是A写的了。这就是所谓的“数字签名”。

## 加密算法---`BCryptPasswordEncoder`的使用及原理

### 一 、介绍

spring security中的`BCryptPasswordEncoder`方法采用`SHA-256 +随机盐+密钥`对密码进行加密。SHA系列是Hash算法，不是加密算法，使用加密算法意味着可以解密（这个与编码/解码一样），但是采用Hash处理，其过程是不可逆的。

不可逆加密SHA：

基本原理：加密过程中不需要使用密钥，输入明文后由系统直接经过加密算法处理成密文，这种加密后的数据是无法被解密的，无法根据密文推算出明文。

RSA算法历史：底层-欧拉函数

1）加密(encode)：注册用户时，使用SHA-256+随机盐+密钥把用户输入的密码进行hash处理，得到密码的hash值，然后将其存入数据库中。

2）密码匹配(matches)：用户登录时，密码匹配阶段并没有进行密码解密（因为密码经过Hash处理，是不可逆的），而是使用相同的算法把用户输入的密码进行hash处理，得到密码的hash值，然后将其与从数据库中查询到的密码hash值进行比较。如果两者相同，说明用户输入的密码正确。

### 二、示例

#### 1、添加依赖

```xml
      <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-core</artifactId>
             <version>5.7.6</version>
        </dependency>
```

#### 2、`PasswordConfig`

为了防止有人能根据密文推测出salt，我们需要在使用`BCryptPasswordEncoder`时配置随即密钥，创建一个`PasswordConfig`配置类，注册`BCryptPasswordEncoder`对象：

```java
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

import java.security.SecureRandom;

/**
 * @author zyj
 * @create 2023-12-07-10:47
 */
@Data
@Configuration
@ConfigurationProperties(prefix = "encoder.crypt")
public class PasswordConfig {
    /**
     * 加密强度
     */
    private int strength;
    /**
     * 干扰因子
     */
    private String secret;

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        //System.out.println("secret = " + secret);
        //对干扰因子加密
        SecureRandom secureRandom = new SecureRandom(secret.getBytes());
        //对密码加密
        return new BCryptPasswordEncoder(strength, secureRandom);
    }
}
```

#### 3、application.yaml

```yaml
encoder:
  crypt:
    secret: ${random.uuid} # 随机的密钥，使用uuid
    strength: 6 # 加密强度4~31，决定盐加密时的运算强度，超过10以后加密耗时会显著增加
```

#### 4、测试

```java

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@SpringBootTest
class ApplicationTests {
    final static private String password = "123456";
    @Autowired
    private BCryptPasswordEncoder encoder;

    @Test
    void savePassword() {
        // encode()：对明文字符串进行加密
        //注册用户时，使用SHA-256+随机盐+密钥把用户输入的密码进行hash处理，得到密码的hash值，然后将其存入数据库中。
        //每次加密结果都不一样
        String encode1 = encoder.encode(password);
        System.out.println("encode1:" + encode1);
        String encode2 = encoder.encode(password);
        System.out.println("encode2:" + encode2);

        // matches()：对加密前和加密后是否匹配进行验证
        //用户登录时，密码匹配阶段并没有进行密码解密（因为密码经过Hash处理，是不可逆的），
        // 而是使用相同的算法把用户输入的密码进行hash处理，得到密码的hash值，然后将其与从数据库中查询到的密码hash值进行比较。
        // 如果两者相同，说明用户输入的密码正确。
        boolean matches1 = encoder.matches(password, encode1);
        System.out.println("matches1:" + matches1);
        boolean matches2 = encoder.matches(password, encode2);
        System.out.println("matches2:" + matches2);

    }
}
```

#### 5、测试结果

![9](..\100-单点登录和社交登录\图片\9.png)
