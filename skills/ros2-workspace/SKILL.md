---
name: ros2-workspace
description: Use when creating, documenting, building, or testing ROS 2 workspaces, ROS 2 packages, colcon overlays, rosdep dependency installation, distro selection, or Ubuntu-to-ROS-distro guidance.
---

# ROS 2 Workspace

Keep ROS 2 workspace guidance short and version-aware.

## Distro Selection

- Do not hard-code a ROS 2 distro unless the user chooses one or a project manifest requires one.
- For Ubuntu 22.04 Jammy, use `ROS_DISTRO=humble` when the user asks for the ROS 2 Humble LTS path.
- For Ubuntu 24.04 Noble, use `ROS_DISTRO=jazzy` when the user asks for the ROS 2 Jazzy LTS path.
- Prefer Humble and Jazzy examples in this template. Do not use unstable or development branches unless the user explicitly asks for them.
- For reusable commands, use `ROS_DISTRO`:

```bash
export ROS_DISTRO=<supported-distro>
source "/opt/ros/${ROS_DISTRO}/setup.bash"
```

## Workspace Pipeline

Run from the workspace root unless noted:

```bash
mkdir -p src
source "/opt/ros/${ROS_DISTRO}/setup.bash"
rosdep update
rosdep install -i --from-path src --rosdistro "${ROS_DISTRO}" -y
colcon build --symlink-install
source install/local_setup.bash
```

Use a new terminal, source the ROS underlay, then source `install/local_setup.bash` before running overlay packages.

## Workspace Shape

```text
workspace/
  *.repos
  src/
    repo/
      package/
        package.xml
  build/
  install/
  log/
```

- Commit manifests and docs.
- Do not commit `build/`, `install/`, `log/`, or imported `src/` repos unless vendoring is explicitly requested.
- CMake packages usually contain `package.xml`, `CMakeLists.txt`, `include/<package>/`, and `src/`.
- Python packages usually contain `package.xml`, `setup.py`, `setup.cfg`, `resource/<package>`, and a module directory matching the package name.
- Do not nest ROS packages inside other ROS packages.
