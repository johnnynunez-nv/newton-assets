# About Structured USD Robots

This asset was created with the [mujoco-usd-converter](https://github.com/newton-physics/mujoco-usd-converter) v0.4.1, having converted the source MJCF from the MuJoCo Menagerie (`seeed_rebot_devarm/seeed_rebot_devarm.xml`, [PR #300](https://github.com/google-deepmind/mujoco_menagerie/pull/300), revision [`29f939e`](https://github.com/johnnynunez/mujoco_menagerie/blob/29f939e14dd6d77e54aa4b3fb78df5b61998bab1/seeed_rebot_devarm/seeed_rebot_devarm.xml) carrying the full link inertia tensors, hardware-derived actuator gains, and the single-motor gripper coupling). The conversion results are unchanged here.

[This layer](./seeed_rebot_devarm.usda) is the main entrypoint for the asset, called the Asset Interface. Consuming users/code/applications should load this file to access the fully composed USD Robot. The interface is a lightweight plain text layer & the bulk of the robot is behind a Payload, enabling delayed-load access to the asset.

The pertinent simulation data is contained in the [Physics Layer](./Payload/Physics.usda) within the Payload. This is a plain text layer, for legibility & ease of editing. It includes all `UsdPhysics`, `NewtonPhysics`, and `MjcPhysics` schemas & attributes, so tuning simulation values can be entirely accomplished by editing this layer alone.

The body hierarchy in this asset is nested, with child bodies specified relative to their parent body, just like the original MJCF. This is a newer feature in OpenUSD & requires at least USD v25.11 for full support, though it is possible to parse nested bodies in older runtimes as well.

## Self-Collision

The source MJCF disables all robot self-collision: every collision geom is authored with `contype="0" conaffinity="1"`, so no two robot geoms can generate contacts in MuJoCo while contacts against default world geoms are preserved. This is deliberate — the single-convex-hull collision meshes interpenetrate in nominal poses (the finger hulls overlap by ~35 mm at the closed home keyframe and separate as the gripper opens; base_link/link1 by ~10 mm), so enabling self-collision would report false contacts at the home pose.

The converter does not carry this filtering into USD: no `mjc:contype`/`mjc:conaffinity` opinions and no `PhysicsFilteredPairsAPI` are authored, so the applied `MjcCollisionAPI` falls back to MuJoCo's defaults (`contype=1 conaffinity=1`). Consumers therefore see 10 enabled collision shapes, with self-collision limited only by whatever articulation adjacency filtering their runtime applies. To match the source model's behavior, disable self-collision for the articulation in the consuming runtime (e.g. `enabledSelfCollisions = False` on the articulation root).

## Gripper Coupling

The two fingers are not independent degrees of freedom on the physical arm: a single
motor drives two opposed racks through one pinion, so their travel is rigidly 1:1. The
source MJCF expresses this with an `<equality joint>` constraint, which the converter
carries into USD as `NewtonMimicAPI` (`newton:mimicJoint`) alongside `MjcEqualityJointAPI`
on `joint_left`.

Consuming runtimes must honor that coupling. Without it the two prismatic joints are free
to drift apart whenever the arm accelerates, and the jaws no longer hold their commanded
opening. Newton imports the constraint from `NewtonMimicAPI`; note that importing it from
`MjcEqualityJointAPI` alone requires
[newton#3761](https://github.com/newton-physics/newton/pull/3761) or newer.

The finger drive gains follow from the same transmission rather than being tuned by hand:
the pinion radius is 7.353 mm/rad, so the actuator's rotary stiffness and its 14 Nm torque
limit appear at the finger as ~925 kN/m and 1904 N respectively. `mjc:gainPrm` is capped at
the stiffest value that remains stable at the solver step, with the bias term set for a
damping ratio of 1 against the 0.0752 kg finger.

## Known Issue: joint limit gains

Converter 0.4.x authors `newton:limitStiffness` / `newton:limitDamping` derived from
MuJoCo's normalized (acceleration-space) solref, while `NewtonJointAPI` documents those
attributes in effort space. Newton therefore rescales them by each joint's effective
inertia and recovers a different `jnt_solref` than the source MJCF. This is tracked in
[newton#3762](https://github.com/newton-physics/newton/issues/3762) and affects every
robot regenerated with 0.4.x, including the eight in
[newton-assets#48](https://github.com/newton-physics/newton-assets/pull/48).

Measured on this asset, comparing Newton's resolved `jnt_solref` against the same robot
loaded natively from its source MJCF:

| variant | `max \|jnt_solref − native\|` |
| --- | --- |
| as converted (0.4.1) | 1.189 |
| with the two attributes stripped | 0.965 |

The source MJCF never authors `solreflimit`, so all eight joints compile natively to
MuJoCo's implicit default `[0.02, 1]`. Note that unlike the Apollo case in
newton-assets#48, removing the two disputed attributes does not recover the native
solref here — see the [issue thread](https://github.com/newton-physics/newton/issues/3762)
for details. This asset will be regenerated once newton#3762 is resolved.

## Warnings

The following warnings were emitted during conversion, due to unsupported features in the converter:

```
[Warning] [mujoco_usd_converter._impl.convert.warn] keys are not supported
```
