[English](./README.md) | [中文](README_ZH.md)
## Building the conversion docker image
### Introduction
The conversion docker image's  `Dockerfile` corresponds to `docker-images/Dockerfile/convert/ubuntu1604/Dockerfile.*`.

There are currently two `Dockerfile` based on `ubuntu1604`, the differences of which are described later.

### Preparation
- Install `Docker`
  > The following `Dockerfile` can be built successfully on `Docker` version `20.10.7`
- High-quality network

### `Dockerfile` description
- `Dockerfile.all` contains `Python3.5 3.6 3.7 3.8 Python2.7`, where `Python3` is managed using `update-alternatives` and `Python3.6 3.7 3.8` requires installing `pip` manually .
  > Use `update-alternatives --config python3` to switch between versions \
  > After successful switch, use `curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && python3 get-pip.py` to install `pip`
- `Dockerfile.select` contains `Python2.7 Python3.7 Python3.5` by default, where `Python3.7` can be replaced by `Python3.6 Python3.8` at the build stage of the image, and the user **does not need** to install `pip`.

### Building images
- If you choose `Dockerfile.select` to build the image, you need to specify the `Python3` version with the `PYTHON_VERSION` parameter, otherwise `Python3.7` will be installed by default.
  > `docker build -t bmnnsdk2-bm1684/dev:ubuntu1604 -f Dockerfile.select --build-arg PYTHON_VERSION=3.x` \
  > `3.x` can be replaced with `3.6 3.7 3.8`
- If the user chooses `Dockerfile.all` for image building
  > `docker build -t bmnnsdk2-bm1684/dev:ubuntu1604 -f Dockerfile.all` \
  > Note that using this `Dockerfile` will **multiple times** compile the `Python` source code, so please be patient.
### Use
Please use it with the `SDK`, see the tutorial on the official website for details.

## Building the deployment image
### Introduction
The deployment image's `Dockerfile` corresponds to `docker-images/Dockerfile/deploy/x86/ubuntu1604/Dockerfile/*`.
### Preparation
- Install `Docker`
  > The following `Dockerfile` can be built successfully on `Docker` version `20.10.7`
- High-quality network

### `Dockerfile` description

- If you choose `Dockerfile.select` to build the image, you need to specify the `Python3` version with the `PYTHON_VERSION` parameter, otherwise `Python3.7` will be installed by default.
  > `docker build -t bmnnsdk2-bm1684/product:ubuntu1604 -f Dockerfile.select --build-arg PYTHON_VERSION=3.x` \
  > `3.x` can be replaced with `3.6 3.7 3.8`
- If the user chooses `Dockerfile.all` for image building
  > `docker build -t bmnnsdk2-bm1684/product:ubuntu1604 -f Dockerfile.all` \
  > Note that using this `Dockerfile` will **multiple times** compile the `Python` source code, so please be patient.

### Use
#### with the full `SDK`
1. Go to the `SDK` root directory. 
2. Create the script file `start_docker.sh`, the content of which is referenced below and can be adapted to suit your needs.
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
3. Run the script to start `Docker`

4. `cd scripts && ./install_lib.sh nntc && source ./envsetup_pcie.sh`