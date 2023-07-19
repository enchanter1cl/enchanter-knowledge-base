**MIRRORING DOCKER HUB**

/*注意了，虽然翻译都是“镜像”，但这个功能是 Mirror 不是 image* /

想必我们都用过阿里云镜像加速器，它是 Docker Hub 的缓存。实际上是 *a registry as pull through cache*.



[Mirroring Docker Hub - Docker (gdevillele.github.io)](https://gdevillele.github.io/registry/recipes/mirror/)

Gotcha

It’s currently not possible to mirror another private registry. Only the central Hub can be mirrored.

***How does it work?***

The first time you request an image from your local registry mirror, it pulls the image from the public Docker registry and stores it locally before handing it back to you. On subsequent requests, the local registry mirror is able to serve the image from its own storage. 首次从本地注册表镜像请求映像时，它会从公共 Docker 注册表中提取映像并将其存储在本地，然后再将其交还给您。在后续请求中，本地注册表镜像能够从其自己的存储中提供映像。

***如果 Docker Hub 中心上的内容发生更改怎么办?***
当尝试使用 tag 拉取时，这个 registry 注册表将检查远程以确保自己具有所请求内容的最新版本。如果没有，它将获取最新内容并缓存它。

> 比如，目前为止，阿里云的缓存 registry 没缓存着 redis:7.0, 也没缓存着 calico:v3.20.6. 阿里云 registry 的设置和本文不太一样。假设你配了阿里云镜像，但拉取 redis:7.0 时，阿里云 registry 并不会替你检查远程和更新它自己。会直接让你请求 Docker Hub，所以能不能 pull 成功就看你运气了。

***How about my disk? 我的磁盘呢？***[¶](https://gdevillele.github.io/registry/recipes/mirror/#what-about-my-disk)

在流失率较高的环境中，过时的数据可能会在缓存中累积。作为拉取缓存运行时，注册表将定期删除旧内容以节省磁盘空间。对已删除内容的后续请求将导致远程提取和本地重新缓存。

***Configuring the cache***

/*假设你想在私网搞一个 cache registry. 这一小节是要在 registry 服务器上配的。这一小节配置阿里云镜像加速器会自己操心*/

To configure a Registry to run as a ***pull through cache\***, the addition of a `proxy` section is required to the config file.

In order to access private images on the Docker Hub, a username and password can be supplied.

```
proxy:
  remoteurl: https://registry-1.docker.io
  username: [username]
  password: [password]
```

***Configuring the Docker daemon***

/*这一小节是在 client 上改 docker-daemon 组件*/

You will need to pass the `--registry-mirror` option to your Docker daemon on startup:

```
docker --registry-mirror=https://<my-docker-mirror-host> daemon
```

For example, if your mirror is serving on `http://10.0.0.2:5000`, you would run:

```
docker --registry-mirror=https://10.0.0.2:5000 daemon
```

NOTE: Depending on your local host setup, you may be able to add the `--registry-mirror` option to the `DOCKER_OPTS` variable in `/etc/default/docker`. /*目前一般在`/etc/docker/daemon.json`*/

阿里云官网：

![image.png](Docker_阿里云镜像与pull_through_cache.assets/20230719224207.png)