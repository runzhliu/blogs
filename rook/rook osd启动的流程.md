# Rook Operator 源码分析(1) - osd 启动的流程

> 阅读本文可能需要有手动部署 Ceph 集群的经验。

Rook 本身很复杂，包含很多 Controller，而 Rook 的复杂不仅体现在这里，并且 Ceph 也非常复杂，在部署和运维上有很多需要注意的地方。本文主要剖析 Rook 启动 osd 的流程，如果有部署过 Ceph 的经验，应该知道加入 osd 大概有两个步骤，1是先 prepare，也就是检查节点上的一些设备是否符合安装 osd，2是激活，也就是 activate。这个过程在 Rook 里也同样需要。

下面是部署完毕后的 Operator 和 osd 相关的 pod 的情况，测试条件下，部署了 3 个 osd，每个节点有 11 块盘。

```bash
# kubectl get pod -n rook-ceph
NAME                                                      READY   STATUS             RESTARTS   AGE
rook-ceph-operator-7658565d97-ptw5n                       1/1     Running            0          20h
rook-ceph-osd-0-78b77dc8d7-xbxhz                          1/1     Running            0          20h
rook-ceph-osd-1-b59bcd696-pz9ml                           1/1     Running            0          20h
rook-ceph-osd-10-5df9b68f95-lfpnd                         1/1     Running            0          20h
rook-ceph-osd-11-5cf6b4bcc5-9cllt                         1/1     Running            0          20h
rook-ceph-osd-12-f9f8cb486-4sdts                          1/1     Running            0          20h
rook-ceph-osd-13-778bdd8db8-qn6t8                         1/1     Running            0          20h
rook-ceph-osd-14-86bc4cd79f-svv92                         1/1     Running            0          20h
rook-ceph-osd-15-bd699f59d-txmbx                          1/1     Running            0          20h
rook-ceph-osd-16-c68c76f8f-bgl7l                          1/1     Running            0          20h
rook-ceph-osd-17-544f7bcf47-xq7dn                         1/1     Running            0          20h
rook-ceph-osd-18-6f75c97cdd-lw8xl                         1/1     Running            0          20h
rook-ceph-osd-19-99f8f8c6f-7drdz                          1/1     Running            0          20h
rook-ceph-osd-2-864c77cc6f-zctcr                          1/1     Running            0          20h
rook-ceph-osd-20-7b76984897-g29zj                         1/1     Running            0          20h
rook-ceph-osd-21-545fd69888-sbfhl                         1/1     Running            0          20h
rook-ceph-osd-22-579897959d-5bff6                         1/1     Running            0          20h
rook-ceph-osd-23-57555ffd8d-2jhwt                         1/1     Running            0          20h
rook-ceph-osd-24-74457df8fd-vrbnt                         1/1     Running            0          20h
rook-ceph-osd-25-799f5bc7d5-sdktc                         1/1     Running            0          20h
rook-ceph-osd-26-5c7fb9dc6c-8vrg9                         1/1     Running            0          20h
rook-ceph-osd-27-74ff56f958-ndjlx                         1/1     Running            0          20h
rook-ceph-osd-28-564dc76999-fzf28                         1/1     Running            0          20h
rook-ceph-osd-29-56dd458d4c-76hm6                         1/1     Running            0          20h
rook-ceph-osd-3-74d7b78fdf-jxll6                          1/1     Running            0          20h
rook-ceph-osd-30-86bf9b4b86-9bxlg                         1/1     Running            0          20h
rook-ceph-osd-31-cb6f96589-29pkb                          1/1     Running            0          20h
rook-ceph-osd-32-6687876667-6zt6g                         1/1     Running            0          20h
rook-ceph-osd-4-c7766fb54-lfcbp                           1/1     Running            0          20h
rook-ceph-osd-5-79f8c8f87c-pk85b                          1/1     Running            0          20h
rook-ceph-osd-6-88cb5ffd7-h59cl                           1/1     Running            0          20h
rook-ceph-osd-7-65697c96-xcgtt                            1/1     Running            0          20h
rook-ceph-osd-8-868475d68b-vk5sl                          1/1     Running            0          20h
rook-ceph-osd-9-5bf449b4c9-75qwc                          1/1     Running            0          20h
rook-ceph-osd-prepare-9.51.1.105-xp2cn                    0/1     Completed          0          152m
rook-ceph-osd-prepare-9.51.1.107-4lgqx                    0/1     Completed          0          152m
rook-ceph-osd-prepare-9.51.2.164-hqj2f                    0/1     Completed          0          152m
rook-discover-7m26q                                       1/1     Running            0          20h
rook-discover-cxcbw                                       1/1     Running            0          20h
rook-discover-slncz                                       1/1     Running            0          20h
```

Rook 代码结构是比较清晰的，关于 osd 的代码可以从 `pkg/operator/ceph/cluster/osd` 找到。

![](rook osd启动的流程_images/def15469.png)

![](rook osd启动的流程_images/386f7c25.png)

![](rook osd启动的流程_images/4b67dfbc.png)

![](rook osd启动的流程_images/bf1598d7.png)

现在都用ceph-volume来安装ceph集群了，ceph-deploy已经不维护了。不用ceph-volume就用ceph本来的命令。

![](rook osd启动的流程_images/f30c1a2a.png)

由于有一个 osd prepare 的过程，在 Rook 里是通过 init-container 来运行一个脚本来实现的。

![](rook osd启动的流程_images/40b15be8.png)

![](rook osd启动的流程_images/365fc9d3.png)

![](rook osd启动的流程_images/47627e69.png)

![](rook osd启动的流程_images/4308866c.png)

关于手动部署，一定要看看 ceph 的[官方文档](https://docs.ceph.com/en/latest/install/manual-deployment/) 。

![](rook osd启动的流程_images/79497a3d.png)

![](rook osd启动的流程_images/ec565aa1.png)

只有当状态为 completed 的时候才会真正的去启动 osd。

![](rook osd启动的流程_images/769c1a29.png)

![](rook osd启动的流程_images/51c97148.png)

![](rook osd启动的流程_images/8a08faf3.png)

![](rook osd启动的流程_images/981ccd28.png)

![](rook osd启动的流程_images/2e56a3d1.png)

![](rook osd启动的流程_images/7893a997.png)

![](rook osd启动的流程_images/deddd3d7.png)

osd 的部署就是通过 `lsblk`、`udevadm` 这些命令来做检查的，下面的命令即使 Rook 里用到的命令，在节点上执行一次看看结果，Rook 会根据一些条件来筛选合适的设备来启动 osd。

```bash
lsblk --all --noheadings --list --output KNAME
```

![](rook osd启动的流程_images/ae6eab6a.png)

```bash
lsblk /dev/sdc --bytes --nodeps --pairs --paths --output SIZE,ROTA,TYPE,PKNAME,NAME,KNAME
```

![](rook osd启动的流程_images/d5cb6577.png)

![](rook osd启动的流程_images/46f44406.png)

![](rook osd启动的流程_images/9d8c5ea9.png)

```bash
udevadm info --query=property /dev/sdc
```

![](rook osd启动的流程_images/11bf44b5.png)

![](rook osd启动的流程_images/e4612555.png)

通过下面的命令，我们在 osd 的节点上运行一下，`ceph-volume` 是用 python 写的，基本上就是通过一些磁盘、设备等命令来检查节点上的设备。如果 osd prepare 之后没有启动真正的 osd pod 的话，就需要查一下 osd prepare pod 的日志了，或者看一下 local-device 开头的 ConfigMap，里面会有执行 `ceph-volume` 的结果，下图的结果明显就是设备因为某些原因被拒绝了，具体尅看 `rejected_resons` 的结果。

```bash
ceph-volume inventory --format json
```

![](rook osd启动的流程_images/666a7c6c.png)

上面的代码走读比较麻烦，跳转来跳转去，下面是我画的一个图，可以参考这个图来理解 Rook 是如何启动 osd 的。



