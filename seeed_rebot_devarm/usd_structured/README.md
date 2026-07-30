# About Structured USD Robots

This asset was created with the [mujoco-usd-converter](https://github.com/newton-physics/mujoco-usd-converter) v0.4.0rc1, having converted the source MJCF from the MuJoCo Menagerie (`seeed_rebot_devarm/seeed_rebot_devarm.xml`, [PR #300](https://github.com/google-deepmind/mujoco_menagerie/pull/300), revision [`0cd5e4c`](https://github.com/johnnynunez/mujoco_menagerie/blob/0cd5e4cd5ea6b4fd684458c5dd918c93517821d8/seeed_rebot_devarm/seeed_rebot_devarm.xml) carrying the full link inertia tensors and explicit actuator `ctrlrange`). The conversion results are unchanged here.

[This layer](./seeed_rebot_devarm.usda) is the main entrypoint for the asset, called the Asset Interface. Consuming users/code/applications should load this file to access the fully composed USD Robot. The interface is a lightweight plain text layer & the bulk of the robot is behind a Payload, enabling delayed-load access to the asset.

The pertinent simulation data is contained in the [Physics Layer](./Payload/Physics.usda) within the Payload. This is a plain text layer, for legibility & ease of editing. It includes all `UsdPhysics`, `NewtonPhysics`, and `MjcPhysics` schemas & attributes, so tuning simulation values can be entirely accomplished by editing this layer alone.

The body hierarchy in this asset is nested, with child bodies specified relative to their parent body, just like the original MJCF. This is a newer feature in OpenUSD & requires at least USD v25.11 for full support, though it is possible to parse nested bodies in older runtimes as well.

## Self-Collision

The source MJCF disables all robot self-collision: every collision geom is authored with `contype="0" conaffinity="1"`, so no two robot geoms can generate contacts in MuJoCo while contacts against default world geoms are preserved. This is deliberate — the single-convex-hull collision meshes interpenetrate in nominal poses (the finger hulls overlap by ~35 mm at the closed home keyframe and separate as the gripper opens; base_link/link1 by ~10 mm), so enabling self-collision would report false contacts at the home pose.

The converter does not carry this filtering into USD: no `mjc:contype`/`mjc:conaffinity` opinions and no `PhysicsFilteredPairsAPI` are authored, so the applied `MjcCollisionAPI` falls back to MuJoCo's defaults (`contype=1 conaffinity=1`). Consumers therefore see 10 enabled collision shapes, with self-collision limited only by whatever articulation adjacency filtering their runtime applies. To match the source model's behavior, disable self-collision for the articulation in the consuming runtime (e.g. `enabledSelfCollisions = False` on the articulation root).

## Warnings

The following warnings were emitted during conversion, due to unsupported features in the converter:

```text
[Warning] [mujoco_usd_converter._impl.convert.warn] keys are not supported
```
