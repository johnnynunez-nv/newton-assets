# About Structured USD Robots

This asset was created with the [mujoco-usd-converter](https://github.com/newton-physics/mujoco-usd-converter), having converted the source MJCF from the MuJoCo Menagerie as of [this commit](https://github.com/google-deepmind/mujoco_menagerie/blob/08ea356c1bf6ad38d8569ce5891e905b0dc55b8a/seeed_rebot_devarm/seeed_rebot_devarm.xml). The conversion results are unchanged here.

That commit is the head of [mujoco_menagerie#300](https://github.com/google-deepmind/mujoco_menagerie/pull/300), which is still open, so the source is pinned to an immutable SHA rather than to a branch. Once #300 merges this should be repinned to the resulting Menagerie commit, and the asset reconverted if review changed the model.

[This layer](./seeed_rebot_devarm.usda) is the main entrypoint for the asset, called the Asset Interface. Consuming users/code/applications should load this file to access the fully composed USD Robot. The interface is a lightweight plain text layer & the bulk of the robot is behind a Payload, enabling delayed-load access to the asset.

The pertinent simulation data is contained in the [Physics Layer](./Payload/Physics.usda) within the Payload. This is a plain text layer, for legibility & ease of editing. It includes all `UsdPhysics`, `NewtonPhysics`, and `MjcPhysics` schemas & attributes, so tuning simulation values can be entirely accomplished by editing this layer alone.

Larger structural changes (e.g. adding new bodies or colliders) would require editing several layers & is best done with a USD aware application rather than by manual edits.

The body hierarchy in this asset is nested, with child bodies specified relative to their parent body, just like the original MJCF. This is a newer feature in OpenUSD & requires at least USD v25.11 for full support, though it is possible to parse nested bodies in older runtimes as well.

The source MJCF leaves `solreflimit` at MuJoCo's implicit default, so no `mjc:solreflimit` is authored here and `SolverMuJoCo` applies that default.

## Self-Collision

The source MJCF disables all robot self-collision: every collision geom is authored with `contype="0" conaffinity="1"`, so no two robot geoms can generate contacts in MuJoCo while contacts against default world geoms are preserved. This is deliberate: the single-convex-hull collision meshes interpenetrate in nominal poses (the finger hulls overlap by ~35 mm at the closed home keyframe and separate as the gripper opens; base_link/link1 by ~10 mm), so enabling self-collision would report false contacts at the home pose.

The converter does not carry this filtering into USD: no `mjc:contype`/`mjc:conaffinity` opinions and no `PhysicsFilteredPairsAPI` are authored, so the applied `MjcCollisionAPI` falls back to MuJoCo's defaults (`contype=1 conaffinity=1`). Consumers therefore see 10 enabled collision shapes, with self-collision limited only by whatever articulation adjacency filtering their runtime applies. To match the source model's behavior, disable self-collision for the articulation in the consuming runtime (for example `enabledSelfCollisions = False` on the articulation root).

## Gripper Coupling

The two fingers are not independent degrees of freedom on the physical arm: a single motor drives two opposed racks through one pinion, so their travel is rigidly 1:1. The source MJCF expresses this with an `<equality joint>` constraint, which the converter carries into USD as `NewtonMimicAPI` (`newton:mimicJoint`) alongside `MjcEqualityJointAPI` on `joint_left`.

Consuming runtimes must honor that coupling. Without it the two prismatic joints are free to drift apart whenever the arm accelerates, and the jaws no longer hold their commanded opening.

The finger drive gains follow from the same transmission rather than being tuned by hand: the pinion radius is 7.353 mm/rad, so the actuator's rotary stiffness and its 14 Nm torque limit appear at the finger as ~925 kN/m and 1904 N respectively. `mjc:gainPrm` is capped at the stiffest value that remains stable at the solver step, with the bias term set for a damping ratio of 1 against the 0.0752 kg finger.

## Warnings

The following warnings were emitted during conversion, due to unsupported features in the converter:

```
[Warning] [mujoco_usd_converter._impl.convert.warn] keys are not supported
```
