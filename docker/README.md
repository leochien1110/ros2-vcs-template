# Docker Workspace

This folder provides Docker Compose profiles for ROS 2 containers from official
images:

- Docker Official `ros:humble-ros-base-jammy` for Ubuntu 22.04 / ROS 2 Humble.
- Docker Official `ros:jazzy-ros-base-noble` for Ubuntu 24.04 / ROS 2 Jazzy.
- Official OSRF `osrf/ros:humble-desktop-full` for ROS 2 Humble GUI workflows.
- Official OSRF `osrf/ros:jazzy-desktop-full` for ROS 2 Jazzy GUI workflows.
- `nvidia/cuda:12.9.2-devel-*` and `nvidia/cuda:13.3.0-devel-*` bases for CUDA-enabled Humble and Jazzy images.
- `nvcr.io/nvidia/l4t-jetpack:r36.4.0` for JetPack 6 / ROS 2 Humble on Jetson.

The OSRF `osrf/ros` repository publishes `desktop-full` tags. The headless
`ros-base` services intentionally use the Docker Official `ros` namespace
because matching `osrf/ros:*ros-base*` tags are not published.

The Compose file mounts the repository root at `/workspace`.
It also mounts `../src` explicitly at `/workspace/src` so imported packages are
visible even when a container command starts directly from the ROS workspace.

## Choosing a Container

Ask what the target environment needs before building:

- Use `ros-base` services for edge devices accessed through SSH.
- Use `desktop-full` services when the user needs GUI tools through local X11,
  AnyDesk, RustDesk, VNC, or another remote desktop session.
- Use CUDA services only when GPU compute is needed and the host NVIDIA runtime
  is working.

## CPU Containers

Run an x86_64 Humble shell:

```bash
docker compose -f docker/compose.yaml --profile humble-amd64 run --rm humble-amd64
```

Run an arm64 Jazzy shell:

```bash
docker compose -f docker/compose.yaml --profile jazzy-arm64 run --rm jazzy-arm64
```

On x86_64 hosts, arm64 containers require binfmt/QEMU support.

## Desktop-Full GUI Containers

Desktop-full containers use official OSRF `osrf/ros:*desktop-full` images and
include GUI-capable ROS packages. On a host PC with X11, allow the container to
access the host display before running a desktop-full service:

```bash
xhost +
```

Run a Humble desktop-full shell:

```bash
docker compose -f docker/compose.yaml --profile humble-desktop-full-amd64 run --rm humble-desktop-full-amd64
```

Run a Jazzy desktop-full shell:

```bash
docker compose -f docker/compose.yaml --profile jazzy-desktop-full-amd64 run --rm jazzy-desktop-full-amd64
```

After the GUI session, disable broad X11 access again:

```bash
xhost -
```

## Manual Custom Builds

The default desktop-full services pull official OSRF images directly. If you
need to customize a desktop-full image, build from the included Dockerfile
manually and use a project-specific image tag:

```bash
docker build -f docker/Dockerfile.ros-desktop \
  --build-arg BASE_IMAGE=ros:humble-ros-base-jammy \
  --build-arg ROS_DISTRO=humble \
  -t my-project-ros:humble-desktop-full-custom \
  docker
```

For Jazzy, change the base image and distro:

```bash
docker build -f docker/Dockerfile.ros-desktop \
  --build-arg BASE_IMAGE=ros:jazzy-ros-base-noble \
  --build-arg ROS_DISTRO=jazzy \
  -t my-project-ros:jazzy-desktop-full-custom \
  docker
```

## CUDA Containers

CUDA containers require the NVIDIA Container Toolkit on the host.

Run a CUDA 12-enabled Humble shell:

```bash
docker compose -f docker/compose.yaml --profile humble-cuda12-amd64 run --rm humble-cuda12-amd64
```

Run a CUDA 12-enabled Jazzy shell:

```bash
docker compose -f docker/compose.yaml --profile jazzy-cuda12-amd64 run --rm jazzy-cuda12-amd64
```

Run a CUDA 13-enabled Humble shell:

```bash
docker compose -f docker/compose.yaml --profile humble-cuda13-amd64 run --rm humble-cuda13-amd64
```

Run a CUDA 13-enabled Jazzy shell:

```bash
docker compose -f docker/compose.yaml --profile jazzy-cuda13-amd64 run --rm jazzy-cuda13-amd64
```

## Jetson Containers

Jetson containers must run on a Jetson device with matching JetPack/L4T drivers.
Use `docker/compose.jetson.yaml` on the Jetson itself, not on a generic x86_64
workstation.

Run a JetPack 6 / ROS 2 Humble shell on Jetson:

```bash
docker compose -f docker/compose.jetson.yaml --profile jetpack6-humble run --rm jetpack6-humble
```

JetPack 7 is not included in this template. On a Jetson device, check current
official NVIDIA L4T/JetPack container tags before adding JetPack 7 services.

Do not reuse ordinary `nvidia/cuda:*` arm64 images for Jetson runtime work. Use
L4T/JetPack images that match the Jetson host BSP.

Inside any container, use the normal workspace workflow:

```bash
mkdir -p src
vcs import src < examples-humble.repos
rosdep update
rosdep install -i --from-path src --rosdistro "${ROS_DISTRO}" -y
colcon build --symlink-install
source install/local_setup.bash
```

## Validation Status

The amd64 services were tested on this workspace:

- `humble-amd64`: pulls and runs; `ros2`, `vcs`, `colcon`, and `/workspace` mount verified.
- `jazzy-amd64`: pulls and runs; `ros2`, `vcs`, `colcon`, and `/workspace` mount verified.
- `humble-desktop-full-amd64`: Compose config verified with official OSRF image.
- `jazzy-desktop-full-amd64`: Compose config verified with official OSRF image.
- `humble-cuda12-amd64`: builds and runs; `ros2`, `vcs`, `colcon`, and `nvidia-smi` verified.
- `jazzy-cuda12-amd64`: builds and runs; `ros2`, `vcs`, `colcon`, and `nvidia-smi` verified.
- `humble-cuda13-amd64`: builds and runs; `ros2`, `vcs`, `colcon`, and `nvidia-smi` verified.
- `jazzy-cuda13-amd64`: builds and runs; `ros2`, `vcs`, `colcon`, and `nvidia-smi` verified.

CUDA services require both a successful image build and a working NVIDIA host
driver/runtime. If `nvidia-smi` fails on the host, treat that as a host driver
issue first.
