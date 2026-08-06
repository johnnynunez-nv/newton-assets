# About Structured USD Robots

This asset was created with the [mujoco-usd-converter](https://github.com/newton-physics/mujoco-usd-converter), having converted the source MJCF from the MuJoCo Menagerie (`seeed_rebot_devarm/seeed_rebot_devarm.xml`, [PR #300](https://github.com/google-deepmind/mujoco_menagerie/pull/300), which carries the full link inertia tensors, the hardware-derived actuator gains, and the single-motor gripper coupling). The conversion results are unchanged here.

[This layer](./seeed_rebot_devarm.usda) is the main entrypoint for the asset, called the Asset Interface. Consuming users/code/applications should load this file to access the fully composed USD Robot. The interface is a lightweight plain text layer & the bulk of the robot is behind a Payload, enabling delayed-load access to the asset.

The pertinent simulation data is contained in the [Physics Layer](./Payload/Physics.usda) within the Payload. This is a plain text layer, for legibility & ease of editing. It includes all `UsdPhysics`, `NewtonPhysics`, and `MjcPhysics` schemas & attributes, so tuning simulation values can be entirely accomplished by editing this layer alone.

Larger structural changes (e.g. adding new bodies or colliders) would require editing several layers & is best done with a USD aware application rather than by manual edits.

The body hierarchy in this asset is nested, with child bodies specified relative to their parent body, just like the original MJCF. This is a newer feature in OpenUSD & requires at least USD v25.11 for full support, though it is possible to parse nested bodies in older runtimes as well.

## Validation

Loaded into Newton via `ModelBuilder.add_usd` and compared against the source MJCF loaded natively:

| check | result |
| --- | --- |
| degrees of freedom | 8 |
| total mass | 6.008505 kg, exact match |
| max per-body mass error | 0 |
| max per-body inertia error | 0 |
| max gravity torque error g(q), 6 poses | 4.8e-06 N m |
| 600 steps under `SolverMuJoCo` | stable, all states finite, 1 equality constraint active |

## Self-Collision

The source MJCF disables all robot self-collision: every collision geom is authored with `contype="0" conaffinity="1"`, so no two robot geoms can generate contacts in MuJoCo while contacts against default world geoms are preserved. This is deliberate: the single-convex-hull collision meshes interpenetrate in nominal poses (the finger hulls overlap by ~35 mm at the closed home keyframe and separate as the gripper opens; base_link/link1 by ~10 mm), so enabling self-collision would report false contacts at the home pose.

The converter does not carry this filtering into USD: no `mjc:contype`/`mjc:conaffinity` opinions and no `PhysicsFilteredPairsAPI` are authored, so the applied `MjcCollisionAPI` falls back to MuJoCo's defaults (`contype=1 conaffinity=1`). Consumers therefore see 10 enabled collision shapes, with self-collision limited only by whatever articulation adjacency filtering their runtime applies. To match the source model's behavior, disable self-collision for the articulation in the consuming runtime (for example `enabledSelfCollisions = False` on the articulation root).

## Gripper Coupling

The two fingers are not independent degrees of freedom on the physical arm: a single motor drives two opposed racks through one pinion, so their travel is rigidly 1:1. The source MJCF expresses this with an `<equality joint>` constraint, which the converter carries into USD as `NewtonMimicAPI` (`newton:mimicJoint`) alongside `MjcEqualityJointAPI` on `joint_left`.

Consuming runtimes must honor that coupling. Without it the two prismatic joints are free to drift apart whenever the arm accelerates, and the jaws no longer hold their commanded opening. Newton imports the constraint from `NewtonMimicAPI` and reports one mimic constraint on load; importing it from `MjcEqualityJointAPI` alone requires [newton#3761](https://github.com/newton-physics/newton/pull/3761) or newer.

The finger drive gains follow from the same transmission rather than being tuned by hand: the pinion radius is 7.353 mm/rad, so the actuator's rotary stiffness and its 14 Nm torque limit appear at the finger as ~925 kN/m and 1904 N respectively. `mjc:gainPrm` is capped at the stiffest value that remains stable at the solver step, with the bias term set for a damping ratio of 1 against the 0.0752 kg finger.

## Joint limit gains

The converter no longer derives `newton:limitStiffness` / `newton:limitDamping` from MuJoCo's normalized `solreflimit`, per the effort-space resolution of [newton#3762](https://github.com/newton-physics/newton/issues/3762); `mjc:solreflimit` carries the source values exactly.

The runtime half of that issue, items 4 and 5, is still open and visible here: `add_usd` leaves `joint_limit_ke` / `joint_limit_kd` at the `ModelBuilder` engine defaults (10000 / 10) rather than the MuJoCo-equivalent defaults (2500 / 100) the MJCF importer applies, so `SolverMuJoCo` compiles a different `jnt_solref` from this asset than from the source MJCF (`max |jnt_solref - native| = 0.96`). This is a property of the USD import path rather than of this asset: forcing those two model fields to 2500 / 100 after `add_usd` recovers the intended behaviour in the meantime.

## Warnings

The following warnings were emitted during conversion, due to unsupported features in the converter:

```
[Warning] [mujoco_usd_converter._impl.convert.warn] keys are not supported
```
