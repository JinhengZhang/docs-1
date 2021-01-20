# Docker方式安装MindSpore GPU版本

<!-- TOC -->

- [Docker方式安装MindSpore GPU版本](#docker方式安装mindspore-gpu版本)
    - [确认系统环境信息](#确认系统环境信息)
    - [获取MindSpore镜像](#获取mindspore镜像)
    - [nvidia-container-toolkit安装](#nvidia-container-toolkit安装)
    - [运行MindSpore镜像](#运行mindspore镜像)
    - [验证是否安装成功](#验证是否安装成功)

<!-- /TOC -->

<a href="https://gitee.com/mindspore/docs/blob/master/install/mindspore_gpu_install_docker.md" target="_blank"><img src="https://gitee.com/mindspore/docs/raw/master/resource/_static/logo_source.png"></a>

[Docker](https://docs.docker.com/get-docker/)是一个开源的应用容器引擎，让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中。通过使用Docker，可以实现MindSpore的快速部署，并与系统环境隔离。

本文档介绍如何在GPU环境的Linux系统上，使用Docker方式快速安装MindSpore。

MindSpore的Docker镜像托管在[Huawei SWR](https://support.huaweicloud.com/swr/index.html)上。
目前容器化构建选项支持情况如下：

| 硬件平台   | Docker镜像仓库                | 标签                       | 说明                                       |
| :----- | :------------------------ | :----------------------- | :--------------------------------------- |
| GPU    | `mindspore/mindspore-gpu` | `x.y.z`                  | 已经预安装MindSpore `x.y.z` GPU版本的生产环境。       |
|        |                           | `devel`                  | 提供开发环境从源头构建MindSpore（`GPU CUDA10.1`后端）。安装详情请参考<https://www.mindspore.cn/install> 。 |
|        |                           | `runtime`                | 提供运行时环境，未安装MindSpore二进制包（`GPU CUDA10.1`后端）。 |

> **注意：** 不建议从源头构建GPU `devel` Docker镜像后直接安装whl包。我们强烈建议您在GPU `runtime` Docker镜像中传输并安装whl包。
> x.y.z对应MindSpore版本号，例如安装1.1.0版本MindSpore时，`x.y.z`应写为1.1.0。

## 确认系统环境信息

- 确认安装Ubuntu 18.04是64位操作系统。
- 确认安装[Docker 18.03或者更高版本](https://docs.docker.com/get-docker/)。

## 获取MindSpore镜像

对于`GPU`后端，可以直接使用以下命令获取最新的稳定镜像：

```bash
docker pull swr.cn-south-1.myhuaweicloud.com/mindspore/mindspore-gpu:{tag}
```

其中：

- `{tag}`对应上述表格中的标签。

## nvidia-container-toolkit安装

对于`GPU`后端，请确保`nvidia-container-toolkit`已经提前安装，以下是`Ubuntu`用户的`nvidia-container-toolkit`安装指南：

```bash
# Acquire version of operating system version
DISTRIBUTION=$(. /etc/os-release; echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$DISTRIBUTION/nvidia-docker.list | tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit nvidia-docker2
sudo systemctl restart docker
```

daemon.json是Docker的配置文件，编辑文件daemon.json配置容器运行时，让Docker可以使用nvidia-container-runtime:

```bash
$ vim /etc/docker/daemon.json
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```

再次重启Docker:

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 运行MindSpore镜像

使用以下命令获取并运行最新的稳定镜像：

```bash
docker run -it -v /dev/shm:/dev/shm --runtime=nvidia --privileged=true swr.cn-south-1.myhuaweicloud.com/mindspore/mindspore-gpu:{tag} /bin/bash
```

其中：

- `-v /dev/shm:/dev/shm` 将NCCL共享内存段所在目录挂载至容器内部；
- `--runtime=nvidia` 用于指定容器运行时为`nvidia-container-runtime`；
- `--privileged=true` 赋予容器扩展的能力;`{tag}`对应上述表格中的标签。

## 验证是否安装成功

按照上述步骤进入MindSpore容器后，测试Docker是否正常工作，请运行下面的Python代码并检查输出：

```python
import numpy as np
import mindspore.context as context
import mindspore.ops as ops
from mindspore import Tensor

context.set_context(mode=context.PYNATIVE_MODE, device_target="GPU")

x = Tensor(np.ones([1,3,3,4]).astype(np.float32))
y = Tensor(np.ones([1,3,3,4]).astype(np.float32))
print(ops.tensor_add(x, y))
```

容器成功运行时会输出：

```text
[[[ 2.  2.  2.  2.],
[ 2.  2.  2.  2.],
[ 2.  2.  2.  2.]],

[[ 2.  2.  2.  2.],
[ 2.  2.  2.  2.],
[ 2.  2.  2.  2.]],

[[ 2.  2.  2.  2.],
[ 2.  2.  2.  2.],
[ 2.  2.  2.  2.]]]
```

至此，你已经成功通过Docker方式安装了MindSpore GPU版本。
如果您想了解更多关于MindSpore Docker镜像的构建过程，请查看[docker repo](https://gitee.com/mindspore/mindspore/tree/master/docker/README.md) 了解详细信息。