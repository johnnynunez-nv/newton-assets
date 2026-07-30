# Seeed Studio reBot DevArm (RobStride) Simulation Assets

## Overview

This package contains robot assets for the **reBot DevArm**, a 6-DOF manipulator with a 2-finger parallel gripper (8 DoF total) developed by [Seeed Studio](https://www.seeedstudio.com/) around [RobStride](https://robstride.com/) quasi-direct-drive actuators. It is derived from the URDF in [Seeed-Projects/reBot-Isaacsim](https://github.com/Seeed-Projects/reBot-Isaacsim).

The subfolders contain:

- **usd_structured**: Structured USD layer files, converted from the MuJoCo Menagerie MJCF.

## Sources

### USD (Structured)

The structured USD model was created with the [mujoco-usd-converter](https://github.com/newton-physics/mujoco-usd-converter), converted from the MuJoCo Menagerie MJCF source. For details, see the [usd_structured README](usd_structured/README.md).

## License

This model is released under the [Apache-2.0 License](LICENSE).

- The **conversion work** — the structured USD layer files (`usd_structured/`)
  produced by the [mujoco-usd-converter](https://github.com/newton-physics/mujoco-usd-converter)
  — is Apache-2.0.
- The **source geometry (meshes) and physical parameters** are derived from the
  upstream [Seeed-Projects/reBot-Isaacsim](https://github.com/Seeed-Projects/reBot-Isaacsim)
  repository. Seeed Studio has confirmed these are released under **Apache-2.0**
  (cc @ZhuYaoHui1998), so the entire package is consistently Apache-2.0 licensed.
