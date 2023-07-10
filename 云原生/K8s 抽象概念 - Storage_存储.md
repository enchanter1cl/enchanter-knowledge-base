> 本文内容翻译自官网

# 0. Volume
卷

当多个容器在一个 Pod 中运行并且需要共享文件时，会出现另一个问题。 跨所有容器设置和访问共享文件系统具有一定的挑战性。

[临时卷](https://kubernetes.io/zh-cn/docs/concepts/storage/ephemeral-volumes/)类型的生命周期与 Pod 相同（container 销毁则卷销毁， container 重启不影响）， 但[持久卷](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/)可以比 Pod 的存活期长。
 
使用卷时, 在 `.spec.volumes` 字段中设置为 Pod 提供的卷，并在 `.spec.containers[*].volumeMounts` 字段中声明卷在容器中的挂载位置。

configMap 也是一种卷哦。

### configMap[](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#configmap)

[`configMap`](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-pod-configmap/) 卷提供了向 Pod 注入配置数据的方法。 ConfigMap 对象中存储 store 的 data 可以被 `configMap` 类型的 volume 引用，然后被 Pod 中运行 container applications 使用。

引用 configMap 对象时，你可以在 volume 中通过它的 name 来引用。 你可以自定义 ConfigMap 中特定条目所要使用的路径 path 。 下面的 yaml 显示了如何将名为 `log-config` 的 ConfigMap 挂载 mount 到名为 `configmap-pod` 的 Pod 中：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod # pod的名字
spec:
  containers:
    - name: test
      image: busybox:1.28
      volumeMounts:
        - name: config-vol # configMap的名字
          mountPath: /etc/config
  volumes:
    - name: config-vol
      items:
        - key: log_level
          path: log_level
```

（这个写法就是赤裸裸的 docker-compose.yaml 那些一级分类啊）

`log-config` ConfigMap 以 volume 的形式 mount，并且存储在 `log_level` 条目 item 中的所有 content 都会被 mount 到 Pod 的 `/etc/config/log_level` path下。 请注意，这个 path 来源于 containers.volumeMounts.mountPath  + volumes.items.`log_level` key 对应的 `path`。 

**说明：**

- 在使用 [ConfigMap](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-pod-configmap/) 之前你首先要创建它。
- ConfigMap 总是以 `readOnly` 的模式 mount。
- 容器以 [`subPath`](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#using-subpath) 卷挂载方式使用 ConfigMap 时，将无法接收 ConfigMap 的更新。
- 文本数据挂载成文件时采用 UTF-8 字符编码。如果使用其他字符编码形式，可。。（略）

# 1. StorageClass
存储类

## 1.0 Introduction[](https://kubernetes.io/docs/concepts/storage/storage-classes/#introduction)

A StorageClass provides a way for administrators to describe the "classes" of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators. This concept is sometimes called "profiles" in other storage systems.

StorageClass 为管理员提供了描述存储 "类" 的方法。 不同的类型可能会映射到不同的服务质量等级或备份策略，或是由集群管理员制定的任意策略。 这个类的概念在其他存储系统中有时被称为 "配置文件“。

## 1.1 The StorageClass Resource[](https://kubernetes.io/docs/concepts/storage/storage-classes/#the-storageclass-resource)

Each StorageClass contains the fields `provisioner`, `parameters`, and `reclaimPolicy`, which are used when a Persist 这些字段会在 StorageClass 需要动态制备 PersistentVolume 时使用到。

StorageClass 对象, 一旦创建了对象就不能再对其更新。

管理员可以为没有申请绑定到特定 StorageClass 的 PVC 指定一个默认的storageClass： 更多详情请参阅 [PersistentVolumeClaim 章节](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)。

storageClass resource 的 yaml:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```

## 1.2 Parameters

### NFS

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: example-nfs
provisioner: example.com/external-nfs
parameters:
  server: nfs-server.example.com
  path: /share
  readOnly: "false"
```

- `server`: Server is the hostname or IP address of the NFS server.
- `path`: Path that is exported by the NFS server.
- `readOnly`: A flag indicating whether the storage will be mounted as read only (default false).

Kubernetes doesn't include an internal NFS provisioner. You need to use an external provisioner to create a StorageClass for NFS. Here are some examples:

- [NFS Ganesha server and external provisioner](https://github.com/kubernetes-sigs/nfs-ganesha-server-and-external-provisioner)
- [NFS subdir external provision](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)

# 2. Dynamic Volume Provisioning
动态卷制备

## 2.0 背景[](https://kubernetes.io/zh-cn/docs/concepts/storage/dynamic-provisioning/#background)

动态卷制备的实现基于 `storage.k8s.io` API 组中的 `StorageClass` API 对象。 集群管理员可以根据需要定义多个 `StorageClass` 对象，每个对象指定一个**卷插件**（又名 **provisioner**）， 卷插件向卷制备商提供在创建卷时需要的数据卷信息及相关参数。

集群管理员可以在 cluster 中定义和公开多种存储（来自相同或不同的存储系统），每种都具有自定义参数集。

点击[这里](https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/)查阅有关 storageClass 的更多信息。

## 2.1 Enable Dynamic Volume Provision[](https://kubernetes.io/zh-cn/docs/concepts/storage/dynamic-provisioning/#enabling-dynamic-provisioning)
启用动态卷制备

要启用动态制备功能，集群管理员需要为用户预先创建一个或多个 `StorageClass` 对象。 `StorageClass` 对象定义当动态制备被调用时，哪一个驱动将被使用(/*哪块场地将被使用？*/)和哪些参数将被传递给驱动。 StorageClass 对象的名字必须是一个合法的 [DNS 子域名](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names#dns-subdomain-names)。 以下清单创建了一个 `StorageClass`  "slow"，它提供类似标准磁盘的永久磁盘。

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
```

以下清单创建了一个 "fast" StorageClass，它提供类似 SSD 的永久磁盘。

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

## 2.2 Using Dynamic Provision[](https://kubernetes.io/zh-cn/docs/concepts/storage/dynamic-provisioning/#using-dynamic-provisioning)

用户通过在 `PersistentVolumeClaim` 中包含存储类来请求动态制备的存储。 

例如，要选择 “fast”  StorageClass，用户将创建 PVC 申请书：

a user would create the following PersistentVolumeClaim:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 30Gi
```

When the **claim(PVC)** is deleted, the volume is destroyed.  PVC 被删除时，卷也会被销毁。