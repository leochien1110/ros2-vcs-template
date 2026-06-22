# ROS 2 VCS Template Workspace

This repository is a template for [ROS 2](https://docs.ros.org/) workspaces that
are populated with [`vcstool`](https://github.com/dirk-thomas/vcstool)
repository manifests. A project can keep one or more `.repos` files at the
workspace root, then import the needed source repositories into `src/`.

The included manifests import the official `ros2/examples` repository, which
contains C++ `rclcpp` and Python `rclpy` talker/listener examples.

## Create a Repository from This Template

Start by creating your own repository from this template. See the official
[GitHub documentation for creating a repository from a template](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template)
for the full GitHub workflow.

From the GitHub web UI:

1. Open this template repository on GitHub.
2. Click **Use this template** above the file list.
3. Select **Create a new repository**.
4. Choose the owner account or organization.
5. Enter a repository name, such as `my_ros2_ws`.
6. Choose the repository visibility.
7. Leave **Include all branches** unchecked unless this template intentionally
   stores useful template branches.
8. Click **Create repository from template**.

After GitHub creates the new repository, continue with the machine setup below,
then clone your new repository as the root of your ROS 2 workspace.

## Prerequisites

Use an Ubuntu version supported by the ROS 2 distribution you want to install.
The ROS 2 distribution changes by Ubuntu release, so do not hard-code a distro
name in reusable instructions. Pick a supported distro from the official
[ROS 2 installation docs](https://docs.ros.org/en/jazzy/Installation.html) or
[REP 2000](https://www.ros.org/reps/rep-2000.html), then set it once:

```bash
export ROS_DISTRO=<your-ros2-distro>
```

This template intentionally uses stable ROS 2 distributions:

- Ubuntu 22.04 Jammy: use [ROS 2 Humble Hawksbill](https://docs.ros.org/en/humble/).
- Ubuntu 24.04 Noble: use [ROS 2 Jazzy Jalisco](https://docs.ros.org/en/jazzy/).

For Ubuntu 22.04:

```bash
export ROS_DISTRO=humble
```

For Ubuntu 24.04:

```bash
export ROS_DISTRO=jazzy
```

## Install ROS 2 and Tools

These commands follow the ROS 2 Ubuntu Debian package flow and keep the ROS 2
distro configurable through `ROS_DISTRO`. See the official install pages for
[Humble](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html)
or [Jazzy](https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html)
when you need distro-specific detail.

```bash
locale
sudo apt update
sudo apt install -y locales software-properties-common curl
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

sudo add-apt-repository universe
sudo apt update

export ROS_APT_SOURCE_VERSION=$(
  curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest |
  grep -F "tag_name" |
  awk -F'"' '{print $4}'
)
curl -L -o /tmp/ros2-apt-source.deb \
  "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb

sudo apt update
sudo apt upgrade
sudo apt install -y "ros-${ROS_DISTRO}-desktop" ros-dev-tools python3-vcstool git
sudo rosdep init
rosdep update
```

For a smaller headless install, replace `ros-${ROS_DISTRO}-desktop` with
`ros-${ROS_DISTRO}-ros-base`.

Source ROS 2 in every terminal that builds or runs the workspace:

```bash
source "/opt/ros/${ROS_DISTRO}/setup.bash"
```

Optional shell startup:

```bash
echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> ~/.bashrc
```

## Clone and Build the Workspace

After ROS 2, `git`, `vcstool`, `rosdep`, and `colcon` are installed, clone your
new repository as the root of your ROS 2 workspace:

```bash
git clone <your-new-repository-url> my_ros2_ws
cd my_ros2_ws
mkdir -p src
source "/opt/ros/${ROS_DISTRO}/setup.bash"
```

Import one project manifest into `src/`.

For Ubuntu 22.04 / ROS 2 Humble:

```bash
vcs validate < examples-humble.repos
vcs import src < examples-humble.repos
```

For Ubuntu 24.04 / ROS 2 Jazzy:

```bash
vcs validate < examples-jazzy.repos
vcs import src < examples-jazzy.repos
```

If imported repositories contain Git submodules, initialize them after import:

```bash
cd src
vcs custom --git --args submodule update --init --recursive
cd ..
```

Install package dependencies and build from the workspace root:

```bash
rosdep install -i --from-path src --rosdistro "${ROS_DISTRO}" -y
colcon build --symlink-install
source install/local_setup.bash
```

## Docker Containers

This template includes Docker Compose profiles under `docker/` for Docker
Official ROS `ros-base` images, official OSRF
[`osrf/ros`](https://hub.docker.com/r/osrf/ros/tags) `desktop-full` images with
X11 support, CUDA-enabled images built from official NVIDIA CUDA bases, and
Jetson-specific services for JetPack 6. See [docker/README.md](docker/README.md)
for usage details:

```bash
docker compose -f docker/compose.yaml --profile humble-amd64 run --rm humble-amd64
docker compose -f docker/compose.yaml --profile jazzy-desktop-full-amd64 run --rm jazzy-desktop-full-amd64
docker compose -f docker/compose.yaml --profile jazzy-cuda13-amd64 run --rm jazzy-cuda13-amd64
```

Manual customization is still supported. For example, build a project-specific
desktop-full image from the included Dockerfile:

```bash
docker build -f docker/Dockerfile.ros-desktop \
  --build-arg BASE_IMAGE=ros:humble-ros-base-jammy \
  --build-arg ROS_DISTRO=humble \
  -t my-project-ros:humble-desktop-full-custom \
  docker
```

## Run Example Nodes

The example manifests are only for verifying that the template works. Run these
talker/listener nodes manually in separate terminals so you can see the ROS 2
pub/sub behavior directly.

For the C++ `rclcpp` example, start the listener first:

```bash
source "/opt/ros/${ROS_DISTRO}/setup.bash"
source install/local_setup.bash
ros2 run examples_rclcpp_minimal_subscriber subscriber_member_function
```

Then start the talker in another terminal:

```bash
source "/opt/ros/${ROS_DISTRO}/setup.bash"
source install/local_setup.bash
ros2 run examples_rclcpp_minimal_publisher publisher_member_function
```

For the Python `rclpy` example, start the listener first:

```bash
source "/opt/ros/${ROS_DISTRO}/setup.bash"
source install/local_setup.bash
ros2 run examples_rclpy_minimal_subscriber subscriber_member_function
```

Then start the talker in another terminal:

```bash
source "/opt/ros/${ROS_DISTRO}/setup.bash"
source install/local_setup.bash
ros2 run examples_rclpy_minimal_publisher publisher_member_function
```

You should see the talker publish messages and the listener print matching
messages. Press `Ctrl-C` in each terminal to stop the nodes.

After you replace the examples with your own project manifests, remove
`examples-humble.repos` and `examples-jazzy.repos` so this repository becomes an
official project manager repo instead of an example workspace.

## Multiple Projects

For multiple projects, keep one manifest per project at the workspace root:

```text
my_ros2_ws/
  examples-humble.repos
  examples-jazzy.repos
  robot_base.repos
  simulation.repos
  src/
```

Import only the project you need:

```bash
vcs import src < robot_base.repos
```

Use clear manifest names and keep each `.repos` file focused on one deployable
or buildable project. If several projects share common repositories, either add
a `common.repos` manifest or document the intended import order.

## Expected Workspace Structure

A ROS 2 workspace should keep source packages under `src/`. Build products are
generated by `colcon` and should not be committed.

```text
my_ros2_ws/
  README.md
  *.repos
  src/
    upstream_repo_a/
      package_a/
        package.xml
        CMakeLists.txt
        include/package_a/
        src/
      package_b/
        package.xml
        setup.py
        setup.cfg
        resource/package_b
        package_b/
          __init__.py
    upstream_repo_b/
      package_c/
        package.xml
  build/
  install/
  log/
```

ROS 2 package basics:

- A CMake package usually contains `package.xml`, `CMakeLists.txt`, `include/<package_name>/`, and `src/`.
- A Python package usually contains `package.xml`, `setup.py`, `setup.cfg`, `resource/<package_name>`, and a Python module directory with the same package name.
- Do not nest ROS packages inside other ROS packages. A workspace can contain many packages, but each package should live in its own directory.

## Maintaining `.repos` Files

Update already-imported repositories:

```bash
vcs pull src
```

After adding or updating repositories in `src/`, export the current state:

```bash
vcs export src > my_project.repos
```

For reproducible checkouts pinned to exact revisions:

```bash
vcs export --exact src > my_project.repos
```

Validate manifests before committing them:

```bash
vcs validate < my_project.repos
```

## Agent Skills

This template keeps reusable agent guidance in repo-local skills to reduce the
size of `AGENT.md` and `CLAUDE.md`:

- `skills/ros2-workspace/SKILL.md` covers ROS 2 distro selection, workspace layout, `rosdep`, and `colcon`.
- `skills/vcstool-workspace/SKILL.md` covers `.repos` files, `vcs validate`, `vcs import`, `vcs pull`, submodules, and exports.
