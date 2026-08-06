# About Structured USD Robots

This asset was created with the [mujoco-usd-converter](https://github.com/newton-physics/mujoco-usd-converter), having converted the source MJCF from the MuJoCo Menagerie [`robotstudio_so101`](https://github.com/google-deepmind/mujoco_menagerie/tree/main/robotstudio_so101) model. The conversion results are unchanged here.

[This layer](./so101.usda) is the main entrypoint for the asset, called the Asset Interface. Consuming users/code/applications should load this file to access the fully composed USD Robot. The interface is a lightweight plain text layer & the bulk of the robot is behind a Payload, enabling delayed-load access to the asset.

The pertinent simulation data is contained in the [Physics Layer](./Payload/Physics.usda) within the Payload. This is a plain text layer, for legibility & ease of editing. It includes all `UsdPhysics`, `NewtonPhysics`, and `MjcPhysics` schemas & attributes, so tuning simulation values can be entirely accomplished by editing this layer alone.

Larger structural changes (e.g. adding new bodies or colliders) would require editing several layers & is best done with a USD aware application rather than by manual edits.

The body hierarchy in this asset is nested, with child bodies specified relative to their parent body, just like the original MJCF. This is a newer feature in OpenUSD & requires at least USD v25.11 for full support, though it is possible to parse nested bodies in older runtimes as well.

## Validation

Loaded into Newton via `ModelBuilder.add_usd` and compared against the source MJCF loaded natively:

| check | result |
| --- | --- |
| degrees of freedom | 6 |
| total mass | 0.644006 kg, exact match |
| max per-body mass error | 0 |
| max per-body inertia error | 1.1e-06 kg m^2 |
| max gravity torque error g(q), 6 poses | 3.0e-04 N m |
| 600 steps under `SolverMuJoCo` | stable, all states finite |

The residual inertia and gravity-torque terms are not authored by this asset. They come from Newton's MJCF importer recomputing the `camera_mount` body frame from geometry: its center of mass lands 2.6 mm from the value MuJoCo itself compiles. This USD reproduces MuJoCo's compiled value to 9e-10 m, so the asset is the more faithful of the two paths and the table above understates its accuracy.

## Joint limit gains

The converter no longer derives `newton:limitStiffness` / `newton:limitDamping` from MuJoCo's normalized `solreflimit`, per the effort-space resolution of [newton#3762](https://github.com/newton-physics/newton/issues/3762); `mjc:solreflimit` carries the source values exactly.

The runtime half of that issue, items 4 and 5, is still open and visible here: `add_usd` leaves `joint_limit_ke` / `joint_limit_kd` at the `ModelBuilder` engine defaults (10000 / 10) rather than the MuJoCo-equivalent defaults (2500 / 100) the MJCF importer applies, so `SolverMuJoCo` compiles a different `jnt_solref` from this asset than from the source MJCF (`max |jnt_solref - native| = 0.95`). This is a property of the USD import path rather than of this asset: forcing those two model fields to 2500 / 100 after `add_usd` recovers the intended behaviour in the meantime.

## Warnings

The following warnings were emitted during conversion, due to unsupported features in the converter:

```
[Warning] [mujoco_usd_converter._impl.convert.warn] cameras are not supported
```
