[English](./README.md) | [中文](README_ZH.md)
## 构建转换容器镜像
### 简介
转换的容器镜像对应 `Dockerfile` 路径为 `./docker-images/Dockerfile/convert` 。

目前提供基于 `ubuntu1604` 的两种 `Dockerfile` ，具体区别见后文介绍。
### 准备工作
- 安装 `Docker`
  > 以下 `Dockerfile` 在 `Docker` 版本为 `20.10.7` 可 `build` 成功
- 一个较好的网络

### `Dockerfile` 说明
- `Dockerfile.all` 包含 `Python3.5 3.6 3.7 3.8 Python2.7` 五个版本，其中 `Python3` 使用 `update-alternatives` 统一管理，另外 `Python3.6 3.7 3.8` 需要自行安装 `pip` 。
  > 使用 `update-alternatives --config python3` 即可切换版本 \
  > 切换成功之后 使用 `curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && python3 get-pip.py` 安装 `pip`
- `Dockerfile.select` 默认包含 `Python2.7 Python3.7 Python3.5` 三个版本 `Python` ，其中 `Python3.7` 用户可在镜像编译阶段选择替换成 `Python3.6 Python3.8` ，用户**无需**自行安装 `pip` 。

### 构建镜像
- 用户如选择 `Dockerfile.select` 进行镜像编译，需通过 `PYTHON_VERSION` 参数指定 `Python3` 的版本，不指定参数则默认使用 `Python3.7` 。
  > `docker build -t bmnnsdk2-bm1684/dev:ubuntu1604 -f Dockerfile.select --build-arg PYTHON_VERSION=3.x` \
  > 其中 `3.x` 可替换为 `3.6 3.7 3.8`
- 用户如选择 `Dockerfile.all` 进行镜像编译，则使用下列命令
  > `docker build -t bmnnsdk2-bm1684/dev:ubuntu1604 -f Dockerfile.all` \
  > 注意，使用此 `Dockerfile` 会**多次**编译 `Python` 源码，请务必耐心等待。
### 使用
请搭配 `SDK` 使用，具体使用见官网教程。

## 构建部署镜像
### 简介
转换的容器镜像对应 `Dockerfile` 路径为 `./docker-images/Dockerfile/deploy` 。
### 准备工作
- 安装 `Docker`
  > 以下 `Dockerfile` 在 `Docker` 版本为 `20.10.7` 可 `build` 成功
- 一个较好的网络

### `Dockerfile` 说明

- 用户如选择 `Dockerfile.select` 进行镜像编译，需通过 `PYTHON_VERSION` 参数指定 `Python3` 的版本，不指定参数则默认使用 `Python3.7` 。
  > `docker build -t bmnnsdk2-bm1684/product:ubuntu1604 -f Dockerfile.select --build-arg PYTHON_VERSION=3.x` \
  > 其中 `3.x` 可替换为 `3.6 3.7 3.8`
- 用户如选择 `Dockerfile.all` 进行镜像编译，则使用下列命令
  > `docker build -t bmnnsdk2-bm1684/product:ubuntu1604 -f Dockerfile.all` \
  > 注意，使用此 `Dockerfile` 会**多次**编译 `Python` 源码，请务必耐心等待。

### 使用
#### 搭配全量 `SDK` 使用
1. 进入 `SDK` 根目录。
2. 创建脚本 `start_docker.sh`，内容参考如下，可根据具体需要作调整。
```
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
WORKSPACE=$PWD
if [ -c "/dev/bm-sophon0" ]; then
  for dev in $(ls /dev/bm-sophon*);
  do
    mount_options+="--device="$dev:$dev" "
  done
fi
docker run \
      --workdir=/workspace \
      --privileged=true \
      ${mount_options} \
      --device=/dev/bmdev-ctl:/dev/bmdev-ctl \
      -v /dev/shm --tmpfs /dev/shm:exec \
      -v $WORKSPACE:/workspace \
      -v /dev:/dev \
      -v /etc/localtime:/etc/localtime \
      -e LOCAL_USER_ID=`id -u` \
      -it bmnnsdk2-bm1684/product:ubuntu1604 \
      bash
```
3. 运行脚本以启动 `Docker`

4. `cd scripts && ./install_lib.sh nntc && source ./envsetup_pcie.sh`