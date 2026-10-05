# Robotiq description dependency for ROX

Description package from
https://github.com/PickNikRobotics/ros2_robotiq_gripper
at commit `a74d007d8f2f06dc6a503ad21038ba869d4999a6`.
The upstream BSD-3-Clause license is included in LICENSE.

This workspace copy supplies both 2F-85 and 2F-140 models and the
Gazebo Sim `sim_gazebo` interface required by rox_description.
The installed Jazzy description only supplies the 2F-85 and an older
`sim_ignition` interface. Build this package and source the workspace
before launching ROX with either simulated gripper.

Only the description package is included; the upstream driver and
hardware controllers are not needed for Gazebo simulation.

## Local simulation patch

For the 2F-140 only, `right_outer_knuckle_joint` is continuous when
`sim_gazebo` is true. Its URDF mimic relation to the bounded `finger_joint`
still limits the commanded motion. Hardware/mock mode retains the original
revolute joint and its limits. This avoids a Gazebo velocity-motor joint-stop
problem observed in the open/close simulation test:
https://github.com/gazebosim/gz-sim/issues/1684

The flag is passed from `robotiq_2f_140_macro.urdf.xacro` to its joint macro.
No meshes or inertial properties were changed.

For the 2F-85, all six moving joints use a velocity limit of 10 rad/s only
when `sim_gazebo` is true. With the original 0.5 rad/s limits, the Gazebo
position/mimic velocity motors diverged at a 0.1 rad goal: the primary
settled around 0.27 rad and the fingertip followers lost their coupling.
Increasing the simulation limits accommodates corrective motor commands.
This is simulation tracking headroom, not a model of the hardware speed.
All position stops remain unchanged; hardware/mock mode retains 0.5 rad/s.

Validated with UR5e in `neo_indoor_test.sdf` at its 0.003 s physics step:
0 -> 0.1 -> 0.3 -> 0.55 -> 0 rad. All six measured joint positions tracked
the primary angle with their respective mimic signs, and every action
succeeded. The earlier check covered only the outer follower and missed
the fingertip failure.
