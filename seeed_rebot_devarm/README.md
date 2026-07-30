# Seeed Studio reBot DevArm (RobStride) Simulation Assets

## Overview

This package contains robot assets for the **reBot DevArm**, a 6-DOF manipulator with a 2-finger parallel gripper (8 DoF total) developed by [Seeed Studio](https://www.seeedstudio.com/) around [RobStride](https://robstride.com/) quasi-direct-drive actuators. It is derived from the URDF in [Seeed-Projects/reBot-Isaacsim](https://github.com/Seeed-Projects/reBot-Isaacsim).

The subfolders contain:

- **usd_structured**: Structured USD layer files, converted from the MuJoCo Menagerie MJCF.

## Sources

### USD (Structured)

The structured USD model was created with the [mujoco-usd-converter](https://github.com/newton-physics/mujoco-usd-converter), converted from the MuJoCo Menagerie MJCF source. For details, see the [usd_structured README](usd_structured/README.md).

## License

This model is released under the [MIT License](LICENSE).

The **source geometry (meshes) and physical parameters** are derived from the
upstream [Seeed-Projects/reBot-Isaacsim](https://github.com/Seeed-Projects/reBot-Isaacsim)
repository, which Seeed Studio releases under the **MIT License**
(cc @ZhuYaoHui1998). The structured-USD **conversion work** in this package
(`usd_structured/`, produced by the [mujoco-usd-converter](https://github.com/newton-physics/mujoco-usd-converter))
is contributed under the same MIT terms, so the entire package is consistently
MIT licensed.
