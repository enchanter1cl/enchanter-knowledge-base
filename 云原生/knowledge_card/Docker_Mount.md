
# Storage Overview
请查阅官方文档 Docker Engine / Storage / Overview

DISK: Docker has two options for containers to store files on the host machine, so that the files **are persisted** even after the container stops: _volumes_, and _bind mounts_.

MEMORY: Docker also supports containers storing files in-memory on the host machine. Such files **are not persisted**. If you’re running Docker on Linux, _tmpfs mount_ is used to store files in the host’s system memory

Choose your type:

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230628205433.png)

- **Volumes** are stored in a part of the host filesystem which is _managed by Docker_ (`/var/lib/docker/volumes/` on Linux). **Non-Docker processes should not** modify this part of the filesystem. 
    
- **Bind mounts** may be stored _anywhere_ on the host system. Non-Docker processes can modify them at any time.
    
- **`tmpfs` mounts** are stored in the host system’s memory only, and are never written to the host system’s filesystem.