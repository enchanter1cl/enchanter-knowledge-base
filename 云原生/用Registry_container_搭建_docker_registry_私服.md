# 用Registry\_container\_搭建\_docker\_registry\_私服

> 带读文档版

registry 是 dockerhub 上的一个 Image. 要求 docker 1.6+

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230730021730.png)

该 image 的 Overview 非常简(fu)洁(yan)，就不看了。用法建议看 docker 官方文档。

## Docker 的官方文档

> Manuals / Open-source projects / Docker Registry / [Registry overview](https://docs.docker.com/registry/)

## Docker Registry

> This page contains information about hosting your own registry using the [open source Docker Registry](https://github.com/distribution/distribution). For information about Docker Hub, which offers a hosted registry with additional features, see [Docker Hub](https://docs.docker.com/docker-hub/).

### What it is

The Registry is a stateless, highly scalable server side application that stores and lets you distribute Docker images. The Registry is open-source.

### Basic commands

Start your registry

```shell
docker run -d -p 5000:5000 --name registry registry:2
```

Pull (or build) some image from the hub

```shell
docker pull ubuntu
```

Tag the image so that it **points to** your registry

```shell
docker image tag ubuntu localhost:5000/myfirstimage
```

Push it

```shell
docker push localhost:5000/myfirstimage
```

Pull it back

```shell
docker pull localhost:5000/myfirstimage
```

Now stop your registry and remove all data

```shell
docker container stop registry && docker container rm -v registry
```

## About Registry

### Understanding image naming

Image names as used in typical docker commands reflect their origin 典型的 docker 命令中使用的镜像名称反映了它们的来源：

* `docker pull ubuntu` instructs docker to pull an image named `ubuntu` from the official Docker Hub. This is simply a shortcut for the longer `docker pull docker.io/library/ubuntu` command
* `docker pull myregistrydomain:port/foo/bar` instructs docker to contact the registry located at `myregistrydomain:port` to find the image `foo/bar`

You can find out more about the various Docker commands dealing with images in the [official Docker engine documentation](https://docs.docker.com/engine/reference/commandline/cli/).

### Use cases

It’s the best way to distribute 发行发布 images inside an isolated network.

## Deploy a registry server

#### Customize the storage location

By default, your registry data is persisted as a [docker volume](https://docs.docker.com/storage/volumes/) on the host filesystem. If you want to store your registry contents at a specific location on your host filesystem, you might decide to use a **bind mount** instead. 默认使用普通 docker volume，即，数据存在哪由 docker 自己决定。你也可以改成 bind mount 来指定存储路径。

### Run an externally-accessible registry

Running a registry only accessible on `localhost` has limited usefulness. In order to make your registry accessible to external hosts, you must first secure it using TLS (即HTTPS). （你也可以改成不安全的 HTTP 访问，本教程暂且不表）

#### Get a certificate

These examples assume the following:

* Your registry URL is `https://myregistry.domain.com/`.
* Your DNS, routing, and firewall settings allow access to the registry’s host on port 443.
* You have already obtained a certificate from a certificate authority (CA).

> 由于没钱，本教程中我们使用自签发的 CA 证书。生成教程可参考这篇《使用自签发 CA 证书运行docker\_registry》
>
> [使用自签发CA证书运行docker\_registry.md](使用自签发CA证书运行docker\_registry.md "mention")

1.  Create a `certs` directory.

<<<<<<< HEAD
1. Create a `certs` directory.
   
    ```shell
    $ mkdir -p certs
    ```
    
    Copy the `.crt` and `.key` files from the CA into the `certs` directory. The following steps assume that the files are named `domain.crt` and `domain.key`.
    
2. Stop the registry if it is currently running.
   
    ```shell
    $ docker container stop registry
    ```
    
3. Restart the registry, directing it to use the TLS certificate. **This command bind-mounts the `certs/` directory into the container at `/certs/`**, <span style="color:orange">and sets environment variables that tell the container where to find</span> the `domain.crt` and `domain.key` file. The registry runs on port 443, the default HTTPS port.
   
=======
    ```shell
    $ mkdir -p certs
    ```

    Copy the `.crt` and `.key` files from the CA into the `certs` directory. The following steps assume that the files are named `domain.crt` and `domain.key`.
2.  Stop the registry if it is currently running.

    ```shell
    $ docker container stop registry
    ```
3.  Restart the registry, directing it to use the TLS certificate. **This command bind-mounts the `certs/` directory into the container at `/certs/`**, and sets environment variables that tell the container where to find the `domain.crt` and `domain.key` file. The registry runs on port 443, the default HTTPS port.

>>>>>>> d373ebdd29fc2d3c913c2ba5af852dae0677debf
    ```shell
    $ docker run -d \
      --restart=always \
      --name registry \
      -v "$(pwd)"/certs:/certs \
      -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
      -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
      -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
      -p 443:443 \
      registry:2
    ```
<<<<<<< HEAD
    
4. Docker clients （即你的其他主机上的 docker） can now pull from and push to your registry using its external address. The following commands demonstrate this:
   
=======
4.  Docker clients （即你的其他主机上的 docker） can now pull from and push to your registry using its external address. The following commands demonstrate this:

>>>>>>> d373ebdd29fc2d3c913c2ba5af852dae0677debf
    ```shell
    $ docker pull ubuntu:16.04
    $ docker tag ubuntu:16.04 myregistry.domain.com/my-ubuntu
    $ docker push myregistry.domain.com/my-ubuntu
    $ docker pull myregistry.domain.com/my-ubuntu
    ```

### Restricting access

Except for registries running on secure local networks, registries should always implement access restrictions. 除在安全的本地网络上运行的 registry 外，registry 应始终执行访问限制。

#### Native basic auth

The simplest way to achieve access restriction is through basic authentication (this is very similar to other web servers’ basic authentication mechanism). This example uses native basic authentication using `htpasswd` to store the **secrets**.

> **Warning**: You **cannot** use authentication with authentication schemes that send credentials as clear text. You must [configure TLS first](https://docs.docker.com/registry/deploying/#run-an-externally-accessible-registry) for authentication to work. 您**不能**使用以明文发送凭证的身份验证方案进行身份验证。必须先配置 TLS，身份验证才能正常工作。

1.  Create a password file with one entry for the user `testuser`, with password `testpassword`:

<<<<<<< HEAD
1. Create a password file with one entry for the user `testuser`, with password `testpassword`:
   
=======
>>>>>>> d373ebdd29fc2d3c913c2ba5af852dae0677debf
    ```shell
    $ mkdir auth
    $ docker run \
      --entrypoint htpasswd \
      httpd:2 -Bbn testuser testpassword > auth/htpasswd
    ```
<<<<<<< HEAD
    
2. Stop the registry.
   
    ```shell
    $ docker container stop registry
    ```
    
3. Start the registry with basic authentication.
   
=======
2.  Stop the registry.

    ```shell
    $ docker container stop registry
    ```
3.  Start the registry with basic authentication.

>>>>>>> d373ebdd29fc2d3c913c2ba5af852dae0677debf
    ```shell
    $ docker run -d \
      -p 5000:5000 \
      --restart=always \
      --name registry \
      -v "$(pwd)"/auth:/auth \
      -e "REGISTRY_AUTH=htpasswd" \
      -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
      -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
      -v "$(pwd)"/certs:/certs \
      -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
      -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
      registry:2
    ```
4. Try to pull an image from the registry, or push an image to the registry. These commands fail.
<<<<<<< HEAD
   
5. Log in to the registry.
   
=======
5.  Log in to the registry.

>>>>>>> d373ebdd29fc2d3c913c2ba5af852dae0677debf
    ```
    $ docker login myregistrydomain.com:5000
    ```

Provide the username and password from the first step. Test that you can now pull an image from the registry or push an image to the registry.

### Deploy your registry using a Compose file

If your registry invocation is advanced, it may be easier to use a Docker compose file to deploy it, rather than relying on a specific `docker run` invocation. 有时使用 docker compose 进行部署可能比依赖特定的 "docker run " 调用更方便. Use the following example `docker-compose.yml` as a template.

```yaml
registry:
  restart: always
  image: registry:2
  ports:
    - 5000:5000
  environment:
    REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
    REGISTRY_HTTP_TLS_KEY: /certs/domain.key
    REGISTRY_AUTH: htpasswd
    REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
  volumes:
    - /path/data:/var/lib/registry
    - /path/certs:/certs
    - /path/auth:/auth
```

Replace `/path` with the directory which contains the `certs/` and `auth/` directories.

Start your registry by issuing the following command in the directory containing the `docker-compose.yml` file:

```bash
docker-compose up -d
```
