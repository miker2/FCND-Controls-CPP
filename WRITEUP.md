# Building a Controller
*Fly Car Nanodegree - Project 3*

## Motor Commands
In order to calculate the correct individual motor thrusts given a collective thrust and a 3-DOF
moment, we need to solve the following equation:

![thrust equation](/animations/thrust_eqn.gif)

Because the matrix is square (and non-singular) it is easily invertible, which will provide us
with a solution to the individual motor thrusts. The solution to this equation was implemented
in the `QuadControl::GenerateMotorCommands(...)` method.

## Implemented Controller
The following sections provide an explanation of the implementation of the various components
required to implement the quadrotor controller.

### Body Rate Control
Body rate control was implemented using the simple proportional control law:

![rate control](/animations/rate_control.gif)

Once implemented in `QuadControl::BodyRateController` the gains were tuned to achieve good
performance.

### Roll/Pitch Control
The roll & pitch controller is slightly more complicated. First, the vertical acceleration must be
calculated from the collective thrust command and the mass of the vehicle. From here the in-plane
acceleration commands can be used in conjunction with the vertical acceleration command to calculate
the commanded "pitch" and "roll" of the vehicle (these aren't actually pitch and roll angles). These
values (`b_x_c` and `b_y_c`) are constrained to be within `±maxTiltAngle`.
From here a proportional feedback law can be used to calculate an euler rate command:

![pitch roll fb](/animations/pitchroll_feedback.gif)

We must then take this euler rate command and convert it into a body rate command, which is
demonstrated in the `QuadControl::RollPitchControl` method.

### Altitude Control
Altitude controls is implemented as two cascaded proportional terms. The first proportional term
offsets the velocity command (`velZCmd`) by the error in the positon signal times the proportional
gain `kpPosZ`. We can then limit the velocity command using the maximum ascent and descent rate.
From here, we calculate the acceleration command by adding to the existing acceleration command the
integral term plus the proportional term on velocity feedback. The overall control structure looks
like the following:

![altitude control](/animations/altitude_ctl.png)

### Lateral Position Control
Lateral position control is implemented similarly to the altitude control, using two cascaded
proportional controllers. The first of these two controllers is a proportional gain on the position
error which feeds into the velocity command. This velocity command is limited to be between
`±maxSpeedXY`
The second proportional controller applies a gain on the velocity error which then feeds into the
acceleration command (which is initialized to the feedforward acceleraion command). This value is
then limited to be between `±maxAccelXY`.
Care is taken to zero the `z` component of this acceleration command before returning.

### Yaw Control
The yaw control is implemented as a single proportional gain on yaw error. Care was taking to
account for angles wrapping between 0 and 2π

## Flight Evaluation
The implemented controller allows the quadrotor to successfully perform each of the 5 provided
scenarios within the required performance envelope.