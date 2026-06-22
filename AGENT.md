# Codex Agent Instructions

This is a ROS 2 `vcstool` template workspace. Before editing manifests or build
docs, read these repo-local skills:

- `skills/ros2-workspace/SKILL.md`
- `skills/vcstool-workspace/SKILL.md`
- `skills/docker-ros2-containers/SKILL.md` before Docker build/test work
- `docker/README.md` before modifying anything under `docker/`

Preserve the template shape: `.repos` manifests live at the workspace root,
imported repositories live under `src/`, and generated `colcon` output stays
uncommitted.

Ask the user whether there is one project or multiple projects when that is
ambiguous. For Ubuntu 22.04, use ROS 2 Humble unless the user selects another
supported distro.

When validating example nodes, recommend that the user run the talker/listener
commands manually in separate terminals. Do not keep long-running ROS nodes
alive for the user unless they explicitly ask for automated smoke tests.

Once the user creates real project `.repos` manifests, ask whether to remove
`examples-humble.repos` and `examples-jazzy.repos` so this becomes an official
project manager repository.

For Docker work, keep headless CPU services based on Docker Official
`ros:*ros-base*` images, desktop-full GUI services based on official OSRF
`osrf/ros:*desktop-full` images, and GPU services based on official NVIDIA CUDA
images. Use `docker/Dockerfile.ros-desktop` only for explicit custom
desktop-full builds. Ask for user confirmation before `docker compose build`,
pulling large images, running CUDA containers, or starting long-running
container sessions.
Ask what to build before choosing a Docker profile: use base/ros-base for edge
devices accessed over SSH; use desktop-full with X11 for local GUI or remote
desktop sessions such as AnyDesk, RustDesk, or VNC.

When testing CUDA containers, separate image build/ROS validation from GPU
runtime validation. If host `nvidia-smi` fails, report that as a host driver
issue rather than a Dockerfile failure.

For Jetson work, use `docker/compose.jetson.yaml` and L4T/JetPack images that
match the Jetson host BSP. Do not treat generic `linux/arm64` CUDA images as
Jetson runtime images. When Docker work is triggered on a Jetson device, check
whether an official public NVIDIA JetPack 7 L4T container tag is available
before deciding whether to add JetPack 7 services.
