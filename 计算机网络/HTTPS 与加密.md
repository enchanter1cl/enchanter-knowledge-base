# HTTPS 与加密

## 0. HTTPS

HTTPS protocol 通过 SSL/TLS 为 data 加密。

SSL, Secure Socket Layer. TLS Transport Layer Security.

### 0.0 验证与加密 authentication\&secret Overview

HTTP 明文传输三大风险，被窃听，被篡改，被冒充。（e.g. 截胡，打开页面跟工行一模一样）HTTPS 协议通过**身份验证**与**传输加密**避免三大风险。身份验证采用非对称加密，传输采用对称加密。

什么是对称加密？比如用户提交的的用户名密码和服务器存的一样才行。加密解密用的相同 secret key.

什么是非对称加密？这就涉及 public key, private key 的概念。private key 是只&#x6709;_&#x52A0;密者_ 自己保存的 secret key. public key 是发出去的 secret key。

下面我们以一个故事场景来理解 HTTPS——假设总部李四和特工张三需要进行加密通信。

如图，明文通信：

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230623011114.png)

如上图，比如指令发给张三这步就容易被截胡和冒充。

下面三张图是加密通信。

第一步，建立连接

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230623011415.png)

第二部 交流 (注意什么是 “Digital Signature”)

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230623012227.png)

下面，我们可以借助第三方权威机构（CA），改造这个过程使通信更安全。

### 0.1 数字证书 Digital Certificates

使用数字证书做，非对称加密的 authentication 和 非对称加密的 communication.

第一步，前置工作

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230623015924.png) /_张三本地需要存的文件： 自己的 private key, public key. CA public key+certificate (value) (这里张三有点像李四)_/

第二步，建立连接的前置工作

李四 保存张三的个人信息和 Public key，以及 CA public key.

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230623020332.png)

/\*总部李四的机器本地需要存： CA public key + certificate(value) -> 解出来的张三的 public key + info(value); \*/

第三步，建立连接【握手】。 先问一嘴，是张三吗？Is there zhangsan?

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230623020715.png)

/\*张三靠 digital certificate 来证明自己的身份 (authentication). \*/

第四步，交流 ![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230623021049.png)

### 0.1 对称加密通信

但是，上面方案的问题是，每次都传 digital certificates 和 解析 digital certifates.太耗时间和总部李四的资源。所以把 communication 换成对称加密。

在 “建立连接(握手)的前置工作”中，总部李四生成一个随机数作为 secret key R，并用张三的 public key 加密 R. 发送给张三。这个 R 就是‘用户名密码’。后面对 内容content 都是使用 R 来加密解密。形成~~digital signature~~ 密文。

张三是 server/ 平台。 李四是 client. secret key R 是 client 生成的，减轻了 server 的压力。

### 0.2 数字证书 VS CA 根证书

数字证书和根证书是非常容易混淆的概念。

digital certificate 是身份证(明) ，是被 Certificate Authority 的 private key 加密过的。

CA 根证书 是 Certificate Authority 的 public key， 有叫根证书。

为什么我们感知不到 “握手前置工作” 呢？浏览器本身存了大平台们的数字证书和根证书。没有的才需要我们手动安。

### 0.3 数字摘要 Digital Digest

李四不可能主动地安装王五的 digital certificate and CA root certificate. 数字摘要是将任意长度的消息变成固定长度的短消息。数字摘要就是利用了H中sh函数的单向性，将需要加密的明文“摘要”成一串128粒长度数字串。这个数字串又称为数字指 这个字符串有叫数字指纹。

### 0.4 数字签名

相当于手写签名，只有信息sender 才能产生。用 Private key 对明文**的数字摘要digital digest** 进行加密，形成数字签名 digital signature。

## 1 htppasswd

```shell
mkdir auth
ca suth 
htppassword -Bbn zhaoliu 123 > aaa.log
```

## 2. OpenSSL 生成 SSL 证书

如果在公网用，最好找 CA (或代理商如阿里云) 申请。 只是内网用，可以自己生成。

```shell
# "ca" 的 private key
# 使用 rsa 生成 ca private key,  长度为4060,  写入文件 ca.key
openssl genrsa -out ca.key 4090

# 
openssl req -x509 -new -nodes -sha512 -days 3650 
-subj "C=CN/ST=Beijing/L"
-key ca.key
-out ca.out

# server zhangsan 的 private key
openssl genrsa -out xxx.com.key 4090
```

xxx.com.key 文件生成了。

一个 server 在生成正式的数字证书前, 需要先有一个请求文件 csr ，里面写着自己信息

```shell
openssl req -sha512 -new \ 
-subj "/C=CN/ST=Beijing/O=.................."
-key yourdomain.com.key
-out yourdomain.com.key
```

yourdomain.com.csr 申请者请求文件生成了

v3.ext 文件

```shell
openssl req -sha512
```

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230623130440.png)

yourdomain.com.crt， 真正的证书生成了。

yourdomian.com.crt -> yourdomain.com.cert docker 认为张三生成的证书是 cert， 否则是他以为是 CA 证书。.

docker 默认会查看 /etc/pki/tis/certs/ca-bundle.crt 的证书。所以用我们的证书替换这个文件。

***

要配置 HTTPS，您必须创建 SSL 证书。您可以使用由受信任的第三方 CA 签名的证书，也可以使用自签名证书。本节介绍如何使用 [OpenSSL](https://www.openssl.org/) 创建 CA，以及如何使用 CA 对服务器证书和客户端证书进行签名。

### 2.0 生成证书颁发机构证书

在生产环境中，应从 CA 获取证书。在测试或开发环境中，您可以生成自己的 CA。若要自己生成 CA 证书，请运行以下命令。

1.  Generate a CA certificate private key. 生成 CA 证书私钥(和公钥)。

    ```sh
    openssl genrsa -out ca.key 4096
    ```
2.  Generrate the CA certificate. 生成 CA 证书。

    调整选项中的值以反映您的组织。`-subj``CN`

    ```sh
    openssl req -x509 -new -nodes -sha512 -days 3650 \
     -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=yourdomain.com" \
     -key ca.key \
     -out ca.crt
    ```

### 2.1 生成服务器证书 Generate a Server Certificate

The certificate usually contains a `.crt` file and a `.key` file, for example, `yourdomain.com.crt` and `yourdomain.com.key`.

1.  Generate a private key. 生成服务器私钥(和公钥)

    ```sh
    openssl genrsa -out yourdomain.com.key 4096
    ```
2.  Generate a certificate **s**igning **r**equest (CSR). 请求文件

    ```sh
    ```

openssl req -sha512 -new\
-subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=yourdomain.com"\
-key yourdomain.com.key\
-out yourdomain.com.csr

````

3. v3.ext
4. v3.ext + 请求文件 ->  crt     服务器证书       generate a certificate for your Harbor host.  Replace the `yourdomain.com` in the CRS and CRT file names with the Harbor host name.

```shell

openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in yourdomain.com.csr \
    -out yourdomain.com.crt

````

### 2.2 Verify the HTTPS Connection

验证一下。

*   Open a browser and enter [https://yourdomain.com](https://yourdomain.com/). It should display the Harbor interface.

    Some browsers might show a warning stating that the Certificate Authority (CA) is unknown. This happens when using a self-signed CA t某些浏览器可能会显示一条警告，指出证书颁发机构 （CA Certificate Autohority） 未知。当使用不是来自受信任的第三方 CA 的自签名 CA 时，会发生这种情况。您可以将 CA 导入浏览器以删除警告。
* 。
* 在运行 Docker 守护程序的计算机上，检查该文件以确保未为 [https://yourdomain.com](https://yourdomain.com/) 设置该选项。`/etc/docker/daemon.json``-insecure-registry` (你如果设过这个，docker 就走 http 了)
*   从 Docker 客户端登录 Harbor。

    ```sh
    docker login yourdomain.com
    ```

    如果已将 443 端口映射改了- If you've mapped `nginx` 443 port to a different port, add the port in the `login` command.

    ```sh
    docker login yourdomain.com:port
    ```

### 3. Summary

_**Summary**_

CA 机构生成私钥，和CA根证书（公钥）。

服务器张三生成公私钥，个人info centent 写入请求文件。结合CA根证书(v3.ext里有好像)拿到(生成) digital cert。

客户端李四 docker login yourdomain.com，问 “是张三吗？”， 张三说我是，给李四 digital cert 和 pub key 做证明。李四的 docker 默默存好。
