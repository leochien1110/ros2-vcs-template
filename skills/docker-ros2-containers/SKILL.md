---
name: docker-ros2-containers
description: Use when building, testing, modifying, or troubleshooting this repository's ROS 2 Docker Compose services, CUDA containers, Jetson containers, Dockerfiles, or Docker image validation workflow.
---

# Docker ROS 2 Containers

Read `docker/README.md` before changing files under `docker/`.

## Files

- `docker/compose.yaml`: generic CPU and CUDA services for `amd64` and `arm64`.
- `docker/Dockerfile.ros-desktop`: optional custom desktop-full image build.
- `docker/Dockerfile.ros-cuda`: ROS 2 on official `nvidia/cuda` bases.
- `docker/compose.jetson.yaml`: JetPack 6 Jetson service.
- `docker/Dockerfile.ros-jetson`: ROS 2 on official Jetson L4T/JetPack base.

## Guardrails

- Ask before pulling large images, building CUDA/Jetson images, or starting long-running containers.
- Ask what the user wants to build before choosing a service: base for SSH edge devices; desktop-full with X11 for local GUI, AnyDesk, RustDesk, VNC, or other remote desktop use.
- Keep headless CPU services based on Docker Official `ros:*ros-base*` images.
- Keep desktop-full GUI services based on official OSRF `osrf/ros:*desktop-full` images.
- Do not invent `osrf/ros:*ros-base*` tags; use the Docker Official `ros` namespace for ros-base.
- Use `docker/Dockerfile.ros-desktop` only for explicit custom desktop-full builds.
- Keep CUDA images based on official `nvidia/cuda:*` images.
- Keep Jetson images based on L4T/JetPack images matching the Jetson host BSP.
- Do not treat generic `linux/arm64` CUDA images as Jetson runtime images.
- When this skill is triggered on a Jetson device, check whether an official public NVIDIA JetPack 7 L4T container tag is available before deciding whether to add or update JetPack 7 services.

## Jetson Detection

Treat the host as Jetson when one of these checks succeeds:

```bash
test -f /etc/nv_tegra_release && cat /etc/nv_tegra_release
test -d /proc/device-tree && tr -d '\0' < /proc/device-tree/model
```

On Jetson, verify the L4T/JetPack family with `/etc/nv_tegra_release` and choose
an L4T/JetPack container tag that matches the host BSP.

## JetPack 7 Availability Check

When on Jetson and JetPack 7 is relevant, check current official NVIDIA tags
instead of relying on stale memory. Use lightweight manifest probes first:

```bash
docker manifest inspect nvcr.io/nvidia/l4t-jetpack:r38.0.0
docker manifest inspect nvcr.io/nvidia/l4t-jetpack:r38.1.0
docker manifest inspect nvcr.io/nvidia/l4t-jetpack:r38.2.0
```

If a public official JP7/L4T tag resolves, add or update a JetPack 7 service
using that tag and validate `docker compose config`. If no tag resolves, leave
JetPack 7 out and document that it was checked but unavailable.

## Build Checks

Use lightweight checks before expensive builds:

```bash
docker compose -f docker/compose.yaml --profile humble-amd64 config humble-amd64
docker compose -f docker/compose.yaml --profile humble-desktop-full-amd64 config humble-desktop-full-amd64
docker compose -f docker/compose.yaml --profile jazzy-cuda13-amd64 config jazzy-cuda13-amd64
docker compose -f docker/compose.jetson.yaml --profile jetpack6-humble config jetpack6-humble
docker buildx build --check --platform linux/amd64 -f docker/Dockerfile.ros-cuda --build-arg CUDA_IMAGE=nvidia/cuda:13.3.0-devel-ubuntu24.04 --build-arg ROS_DISTRO=jazzy docker
docker buildx build --check --platform linux/arm64 -f docker/Dockerfile.ros-jetson --build-arg JETSON_IMAGE=nvcr.io/nvidia/l4t-jetpack:r36.4.0 --build-arg ROS_DISTRO=humble docker
```

## Runtime Validation

CPU service smoke test:

```bash
docker compose -f docker/compose.yaml run --rm --no-deps humble-amd64 bash -lc 'echo ROS_DISTRO=$ROS_DISTRO; source /opt/ros/$ROS_DISTRO/setup.bash; ros2 -h >/dev/null && echo ros2_ok; command -v vcs; command -v colcon; test -f /workspace/docker/compose.yaml && echo workspace_ok'
```

Desktop-full X11 setup:

```bash
xhost +
docker compose -f docker/compose.yaml run --rm --no-deps humble-desktop-full-amd64 bash -lc 'echo ROS_DISTRO=$ROS_DISTRO; source /opt/ros/$ROS_DISTRO/setup.bash; ros2 -h >/dev/null && echo ros2_ok; test -S /tmp/.X11-unix/X${DISPLAY#:} && echo x11_socket_ok'
xhost -
```

Custom desktop-full build, only when official OSRF image customization is needed:

```bash
docker build -f docker/Dockerfile.ros-desktop --build-arg BASE_IMAGE=ros:humble-ros-base-jammy --build-arg ROS_DISTRO=humble -t my-project-ros:humble-desktop-full-custom docker
```

CUDA service smoke test:

```bash
docker compose -f docker/compose.yaml build humble-cuda12-amd64
docker compose -f docker/compose.yaml run --rm --no-deps humble-cuda12-amd64 bash -lc 'echo ROS_DISTRO=$ROS_DISTRO; source /opt/ros/$ROS_DISTRO/setup.bash; ros2 -h >/dev/null && echo ros2_ok; command -v vcs; command -v colcon; nvidia-smi'
```

Separate image build/ROS validation from GPU runtime validation. If host
`nvidia-smi` fails, report that as a host driver/runtime issue, not a Dockerfile
failure.
