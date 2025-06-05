# 使用自签发CA证书运行docker\_registry

> 来自 docker 官方文档 《Test an insecure registry》

## Test an insecure registry

While it’s highly recommended to secure your registry using a TLS certificate issued **by a known CA**, you can choose to use self-signed certificates, or use your registry over an unencrypted HTTP connection. Either of these choices involves security trade-offs and additional configuration steps. 你可以使用自签发证书，或是改用不安全的 HTTP 通信。

> 以下的生成证书命令是基于 openssl 3.0. 而大部分 linux 发行版自带的都是 openssl 1.0. 建议先升级。升级教程参见我的博客： [将 openssl 升级到 3.0\_openssl3.0\_Erato Rabbit的博客-CSDN博客](https://blog.csdn.net/Enchanter06/article/details/131418665?spm=1001.2014.3001.5501)

下面假设我们要给 myregistry.domain.com 来签发一个证书 certificate。

1.  Generate your own certificate:

    ```shell
    $ mkdir -p certs

    $ openssl req \
      -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key \
      -addext "subjectAltName = DNS:myregistry.domain.com" \
      -x509 -days 365 -out certs/domain.crt
    ```

    Be sure to use the name `myregistry.domain.com` as a CN.
2. Use the result to [start your registry with TLS enabled](https://docs.docker.com/registry/deploying/#get-a-certificate).
3. Instruct every Docker daemon to trust that certificate. The way to do this depends on your OS. 让所有(客户端上的) docker daemon 都信任这个证书。
   * **Linux**: Copy the `domain.crt` file to `/etc/docker/certs.d/myregistrydomain.com:5000/ca.crt` on every Docker host. You do not need to restart Docker.
   * **Windows Server**:
     1. Open Windows Explorer, right-click the `domain.crt` file, and choose Install certificate.
     2. Click **Browser** and select **Trusted Root Certificate Authorities**.
     3. Click **Finish**. Restart Docker.
