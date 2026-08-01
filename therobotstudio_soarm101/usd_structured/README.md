# About Structured USD Robots

This asset was created with the [mujoco-usd-converter](https://github.com/newton-physics/mujoco-usd-converter) v0.4.1, having converted the source MJCF from the MuJoCo Menagerie [`robotstudio_so101`](https://github.com/google-deepmind/mujoco_menagerie/tree/main/robotstudio_so101) model. The conversion results are unchanged here — 0.4.1 authors `physics:staticFriction` on both physics materials itself, so the manual edit that earlier revisions of this asset carried is no longer needed.

[This layer](./so101.usda) is the main entrypoint for the asset, called the Asset Interface. Consuming users/code/applications should load this file to access the fully composed USD Robot. The interface is a lightweight plain text layer & the bulk of the robot is behind a Payload, enabling delayed-load access to the asset.

The pertinent simulation data is contained in the [Physics Layer](./Payload/Physics.usda) within the Payload. This is a plain text layer, for legibility & ease of editing. It includes all `UsdPhysics`, `NewtonPhysics`, and `MjcPhysics` schemas & attributes, so tuning simulation values can be entirely accomplished by editing this layer alone.

The body hierarchy in this asset is nested, with child bodies specified relative to their parent body, just like the original MJCF. This is a newer feature in OpenUSD & requires at least USD v25.11 for full support, though it is possible to parse nested bodies in older runtimes as well.

## Validation

Loaded into Newton via `ModelBuilder.add_usd`: 6 DoF, total mass 0.644 kg. Body masses and inertias are identical to the source MJCF (converter is lossless for mass properties); the colours (orange printed parts, black STS3215 servos) are preserved as UsdPreviewSurface materials. Generalized gravity torque g(q) matches the MuJoCo MJCF source to < 1e-6 N·m across poses (body masses and inertias are authored at body level from the compiled model).

## Known Issue: joint limit gains

Converter 0.4.x authors `newton:limitStiffness` / `newton:limitDamping` derived from
MuJoCo's normalized (acceleration-space) solref, while `NewtonJointAPI` documents those
attributes in effort space. Newton therefore rescales them by each joint's effective
inertia and recovers a different `jnt_solref` than the source MJCF. This is tracked in
[newton#3762](https://github.com/newton-physics/newton/issues/3762) and affects every
robot converted with 0.4.x, including the eight in
[newton-assets#48](https://github.com/newton-physics/newton-assets/pull/48).

Measured on this asset, comparing Newton's resolved `jnt_solref` against the same robot
loaded natively from its source MJCF, `max |jnt_solref − native| = 0.336`. The source
MJCF never authors `solreflimit`, so all six joints compile natively to MuJoCo's implicit
default `[0.02, 1]`; the error grows monotonically along the chain, consistent with the
effective-inertia rescale. This asset will be regenerated once newton#3762 is resolved.

## Warnings

The following warnings were emitted during conversion, due to unsupported features in the converter:

```text
[Warning] [mujoco_usd_converter._impl.convert.warn] cameras are not supported
```
