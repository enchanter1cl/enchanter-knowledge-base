> 手把手截图版

0， 1， 2 步骤都是在服务端完成。

# 0. 生成自签名证书

openssl 版本 3.0.8：

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230730001724.png)

我打算使用域名 eratoregistry.com 生成证书：

```shell
mkdir -p certs

openssl req \
      -newkey rsa:4096 -nodes -sha256 -keyout certs/erato.key \
      -addext "subjectAltName = DNS:eratoregistry.com" \
      -x509 -days 365 -out certs/erato.crt
```

填写基本信息，可以瞎填

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230730002148.png)

可以看到已经生成好了：

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230730002822.png)

# 1. 设置用户名密码

使用 htppass

```shell
$ mkdir auth
$ docker run \
--entrypoint htpasswd \
httpd:2 -Bbn erato 123456 > auth/htpasswd
```

# 2. 基于证书和用户名跑 registry

我当前所在的路径是 /erato,  docker-compose.yml 如下

```yaml
registry:
  restart: always
  image: registry:2
  ports:
    - 5000:5000
  environment:
    REGISTRY_HTTP_TLS_CERTIFICATE: /certs/erato.crt
    REGISTRY_HTTP_TLS_KEY: /certs/erato.key
    REGISTRY_AUTH: htpasswd
    REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
  volumes:
    - /erato/data:/var/lib/registry
    - /erato/certs:/certs
    - /erato/auth:/auth
```

```shell
docker compose up -d
```

好了，一个 registry server 已经跑起来了~

# 3. 客户端访问

找同一 vpc 下的另一台安了 docker 的服务器，作为客户端。

从服务端 copy 给客户端一份 `erato.crt` 文件：

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230730014503.png)

在客户端上，复制进下面的目录：

```shell
mkdir -p /etc/docker/certs.d/eratoregistry.com/
cp erato.crt /etc/docker/certs.d/eratoregistry.com/ca.crt
```

登录

```shell
docke login eratoregistry.com
```

用户名为 erato,  密码 123456

可以 pull/push 了：

```shell
docker pull alpine:3.14
docker tag alpine:3.14 eratoregistry.com/alpine:3.14
docker push eratoregistry.com/alpine:3.14
```

```shell
docker pull eratoregistry.com/alpine:3.14
```

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230730015344.png)

OK 完成！