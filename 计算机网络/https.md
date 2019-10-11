<http://km.oa.com/group/33341/articles/show/362631?kmref=search&from_page=1&no=1>

<https://github.com/youngwind/blog/issues/108>

# 概括

用非对称加密交换 key, 用对称加密（使用key作为密钥）传输数据

## https中的非对称加密

非对称加密指的是 客户端传递 Pre-Master 给服务器，先用 服务器证书上的公钥加密，服务器接收后使用对应的私钥解密，但是 Pre-Master  并非最终用于对称加密的key.

## https中的对称加密



对称加密过程中使用的 key 是客户端传递给服务端的？ 还是服务端传递给客户端的？ 

都不是，显然没法传递，否则怎么保证传递过程的安全。

其实这个key 是两端协商并最终生成的。

```shell
enc_key=Fuc(random_C, random_S, Pre-Master)
```

该算法是两端协商确定的，参数也是分别两端提供的。

### https 加密过程参考链接

<https://www.wosign.com/faq/faq2016-0309-04.htm> 

这个链接是沃通公司提供的，这是国内目前最有影响力的 CA 机构。业务覆盖 360、各大高校，英文名 WoSign

### 这三个参数分别来自哪里：

#### random_C

client_hello 阶段客户端携带随机数 random_C 参数给服务器

#### random_S

server_hello 阶段服务器携带随机数 random_S 参数给客户端

#### Pre-master

Pre-master 是 client_key_exchange 阶段客户端生成的, 用服务端证书上的公钥加密后发送给服务器，服务器可以用对应的私钥解密得到  Pre-master 。

最终服务器和客户端都是通过  enc_key=Fuc(random_C, random_S, Pre-Master) 得到的 对称加密的 key 。

## 加密效率

非对称加密的计算耗时是大于对称加密几个数量级的，所以https中的数据传输不能一直使用非对称加密，否则效率非常低下，因此是 非对称与对称结合来做。

# 三大特性



## 身份认证



## 数据加密



## 数据一致性

sender 把message和用message生成的hash值都发给receiver,reciver使用同样的hash算法对收到的message进行hash计算，如果两个hash值一致就说吗消息没有被篡改。

# 网络劫持

## 客户端劫持

木马，浏览器插件

##链路劫持

最广泛，路由器，网关，防火墙

## 服务器劫持

入侵服务器

## 使用https可以杜绝网络劫持

# QUIC

http3，第三代http,基于 udp 实现

无TCP队头阻塞：由于使用UDP而非TCP，所以就不存在TCP队头阻塞的问题











