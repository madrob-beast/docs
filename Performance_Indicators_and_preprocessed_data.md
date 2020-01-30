# MADROB Benchmark procedure

## Terminology
The **starting side** is the side of the door where the robot is at the start of the benchmark. The **destination side** is the side of the door that the robot must reach by going through the door.

The terms **clockwise/CW** and **counterclockwise/CCW** identify the two possible directions of rotation of the door panel when observed from above, with respect to its central (closed) position.

The same terms are also used to identify the two sides of the testbed area that are separated by the wall to which the door is affixed. Precisely, CW is the side entered by the door panel when it rotates in the CW direction; CCW the side it enters when it rotates in CCW direction.
In the following, the values of `starting side` and `destination side` can be either `CW` or `CCW`.

In the execution of the benchmark movement of the door can be constrained so that it only opens when rotated either CW or CCW. The **opening side** of the testbed area is the side towards which the door is allowed to move. The value of `opening side` can be either `CW` or `CCW`. Such value is independently from the value of `starting side`.

The **threshold** is the ideal line on the ground corresponding to the projection of the door panel when locked. The threshold separates the CW and CCW sides of the testbed. A robot *crosses the threshold* when it goes from a pose where its projection on the ground is entirely on one side of the threshold to a pose where its projection is entirely on the other side.

## Procedure
In order to execute the benchmark, the robot must execute, in order, the following steps:

1. Touch the door handle.

2. Unlock the door.

3. Open the door.

4. Cross the threshold.

5. Close the door, without crossing the threshold again.

6. Lock the door, without crossing the threshold again.

*NOTE: events "door unlocked" (relevant to point 2) and "door locked" (relevant to step 6) are currently (January 2020) not detectable. They will be once an additional sensor (under development) is installed.*

For the benchmark to start, the robot must first be set in a **start pose** specified by the referees. Once this precondition is satisfied, the benchmark starts in a time instant specified by the referees.

The benchmark stops as soon as any of the following events occurs:

* the last step of the benchmark is completed  by the robot;

* the timeout has elapsed.

The benchmark can also be manually interrupted by a referee (via the GUI). In this case, the execution is invalid.


## MADROB and the European Robotics League
The benchmark procedure of MADROB intentionally resembles very closely the one used by the ERL benchmark "Through the Door". Notwithstanding this parallelism, performance evaluation metrics (**Performance Indicators** or PIs in the terminology of EUROBENCH) used by MADROB do not correspond to those used by the ERL.

The differences between MADROB's PIs and "Through the Door"'s metrics are such that, according to the concepts underpinning the ERL, MADROB is closer to a *Functionality Benchmark* than a *Task Benchmark*, while the opposite is true for "Through the Door".


# MADROB pre-processed data

Pre-processed data are the inputs to the Performance Indicators.

## Pre-processed data: events sequence
Time series containing detectable events as values.

Columns: timestamp [float] (absolute ros time: seconds since epoc), event [string/enum].

Events:
 - benchmark start: identifies the start of benchmark execution. This event is always present. The timing of this event is affected by a delay due to the people operating the robot having to manually start the robot.
 - door opens: `|door angular position| > threshold for closed door`, given door was previously closed. The threshold needs manual calibration.
 - door closes: `|door angular position| < threshold for closed door`, given door was previously open. The threshold needs manual calibration.
 - handle is touched: event occurs when force signal exceeds a predefined threshold. The threshold needs manual calibration. Only one occurrence of this event can be present. (Note: Currently force signal is not zero when door is closed. This will change after hardware upgrade).
 - humanoid moves to cw side, humanoid moves to ccw side: last detection of humanoid moving from one side to the other by passage sensors. Only one occurrence of these events can be present.
 - humanoid approaches the door on cw side, humanoid approaches the door on ccw side: first detection of a humanoid under the passage sensors. Only one occurrence of these events can be present.
 - door is locked, door is unlocked: currently not implemented, requires addition of latch sensor to the testbed.
 - benchmark stop: identifies the end of benchmark execution. This event is always present.

Note: The process to generate passage signals, used to compute events `humanoid approaches the door on [cw|ccw] side` and `humanoid moves to [cw|ccw] side`, requires fine-tuning regarding noise filtering and door geometry (data from sensors which - given the angular position of the door - may be perceiving the door panel must be discarded)


## Pre-processed data: force on handle
Copy of raw data from topic /madrob/handle/force.

Columns: timestamp [float] (absolute ros time: seconds since epoc), force [grams].


## Pre-processed data: door angular position
Copy of raw data from topic /madrob/door/angle.

Columns: timestamp [float] (absolute ros time: seconds since epoc), door angular position [rad].


## Pre-processed data: door angular velocity
Computed by applying a moving average on the angular position from topic /madrob/door/angle to reduce noise (see https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.core.window.Rolling.mean.html), and computing the first order derivative of the position using the Savitzky-Golay filter (see https://docs.scipy.org/doc/scipy-0.16.1/reference/generated/scipy.signal.savgol_filter.html).

These two filtering operations depend from the following parameters: `moving average window size`, `Savitzky-Golay window size` and `Savitzky-Golay polynomial order`.

`moving average window size` and `Savitzky-Golay window size` are computed by rounding up to the closest odd number of samples falling in a time window of 0.2 seconds. This value was chosen as a compromise between noise reduction and preserving the maximum velocity and acceleration values.

`Savitzky-Golay polynomial order` is set to 2, the lowest possible value enabling the computation of angular velocity and acceleration. Higher values would introduce artefacts and require a larger window size.

Columns: timestamp [float] (absolute ros time: seconds since epoc), door angular velocity [rad/s].


## Pre-processed data: door angular acceleration
Computed by applying a moving average on the angular position from topic /madrob/door/angle to reduce noise (see https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.core.window.Rolling.mean.html), and computing the second order derivative of the position using the Savitzky-Golay filter (see https://docs.scipy.org/doc/scipy-0.16.1/reference/generated/scipy.signal.savgol_filter.html).

These two filtering operations depend from the following parameters: `moving average window size`, `Savitzky-Golay window size` and `Savitzky-Golay polynomial order`.

`moving average window size` and `Savitzky-Golay window size` are computed by rounding up to the closest odd number of samples falling in a time window of 0.2 seconds. This value was chosen as a compromise between noise reduction and preserving the maximum velocity and acceleration values.

`Savitzky-Golay polynomial order` is set to 2, the lowest possible value enabling the computation of angular velocity and acceleration. Higher values would introduce artefacts and require a larger window size.

Columns: timestamp [float] (absolute ros time: seconds since epoc), door angular acceleration [rad/(s^2)].



# MADROB Performance Indicators

Each Performance Indicator (PI) is described by:

**Input**
List of pre-processed data timeseries and testbed configuration values used to compute the PI.

**Output**
Value of the PI. Data type depends on the PI.

**Computation method**
Algorithm or formula used to generate the Output from the Input.

**Parameters**
Values influencing the result of the PI.



## Performance Indicator: Overall execution time
### Input

#### pre-processed data:
 - events sequence

#### Testbed configuration values:
 - opening side
 - destination side

### Output
Overall duration of benchmark execution.

### Computation method
The output is the difference between the times of events `benchmark stop` and `benchmark start`.

### Parameters
None.

### Notes
Execution Time cannot exceed the timeout.
This measure uses the `start benchmark` event to take into account the time employed by the humanoid to percieve the door and plan its actions.



## Performance Indicator: Time to handle
### Input

#### Pre-processed data:
 - events sequence

#### Testbed configuration values:
None

### Output
Time elapsed from the start of the benchmark to the first time the handle is touched by the robot/humanoid. This PI accounts for the time the robot takes to perceive the door, plan its actions and start opening the door.

### Computation method
Time elapsed from event `benchmark start` to first event `handle is touched`.

Note: What if the robot never reaches the final step? should the result be the value of timeout or `infinite`? 

### Parameters
None

### Notes
Time To Handle cannot exceed the timeout.



## Performance Indicator: Threshold occupation time
### Input

#### Pre-processed data:
 - events sequence
 
#### Testbed configuration:
 - opening side
 - destination side

### Output
Time elapsed between the humanoid approaching the threshold from the starting side and the humanoid leaving the proximity of the threshold on the destination side (measured by sensors detecting when the humanoid is present in the proximity of the door on each side).

### Computation method
Time elapsed from event `humanoid approaches the door on [C]CW side` to event `humanoid moves to [c]cw side`.

Note: What if the robot never reaches the final step? should the result be the value of timeout or `infinite`? 

### Parameters
None

### Notes
Door Occupation Time cannot exceed the timeout.
A door threshold is a bottleneck in people mobility between environments.
Door Occupation Time evaluates how good the robot is in limiting its use of this limited resource.
This measure may be affected by the robot shape since it relies on the passage sensors, so it may not be a good indicator.



## Performance Indicator: Passage time
### Input

#### Pre-processed data:
 - events sequence

#### Testbed configuration values:
 - opening side
 - destination side

### Output
Time elapsed between the humanoid starts opening the door (touching the handle) and the humanoid closes the door after reaching the destination side.

### Computation method
The `final event` is considered as the last `door closes` event, occurring after an event `humanoid moves to cw side` or `humanoid moves to ccw side`, depending on `destination side` for the run.
If the `final event` is not present in the etvents sequence, the result is considered a timeout.
The output is the difference between time of event `handle is touched` and time of `benchmark stop`.

### Parameters
None.

### Notes
Execution Time cannot exceed the timeout.
This measure is similar to the PIs Door occupation time and Overall execution time, but avoids using the timing of the events `benchmark start` and passage events, that rely on manual timing (starting benchmark and the robot at the same time), and may give results skewed by the robot shape (laser sensors may provide different timings based on the height/shape of the robot).
The difference with the PI Overall execution time is that the time took by the robot to perceive the door and plan its actions is excluded.



## Performance Indicator: Door operation safety
### Input

#### Pre-processed data:
 - door angular acceleration

#### Testbed configuration values:
None

### Output
This PI is a measurement of the safety of door operation by the robot based on the maximum angular acceleration of the door panel.

### Computation method
`max(abs(a)), for each value of angular acceleration a`

### Parameters
Parameters of filter applied to door angular acceleration signal (see Notes)

### Notes
This PI is a measurement of the unsafety of door operation by the robot.
High angular accelerations correspond to sudden movements and high forces exerted on any object or person touching the door panel.
The lower door acceleration, the lower the risk of serious incidents.
High angular acceleration can indicate tremors in the robot's hand effectors. However, these are not as significant for safety because of the limited range of motion they impose to the door panel, therefore low-pass filtering is applied to the door angular acceleration before computing this PI (as opposed to the PI Door operation smoothness).



## Performance Indicator: Door operation smoothness
### Input

#### Pre-processed data:
 - door angular acceleration

#### Testbed configuration values:
None

### Output
This PI is a measurement of the smoothness of the actuation of the door panel based on its  angular acceleration.

### Computation method
`1/sqrt(sum(a^2)), for each value of angular acceleration a`

### Parameters
None

### Notes
To operate the door smoothly the humanoid should minimise the acceleration of the door.
Angular accelerations and decelerations can indicate bumps against the handle/panel, tremors and unnecessary corrections by the humanoid in the actuation of the door panel.
The acceleration values are squared to bias the indicator against higher values of acceleration.



## Performance Indicator: Door handling care
### Input

#### Pre-processed data:
 - force on handle

#### Testbed configuration values:
None

### Output
This PI measures how delicately the robot operates the door based on the maximum force applied to the handle.

### Computation method
`max(abs(f)), for each value of force f`

### Parameters
None

### Notes
High force applied to the handle is a symptom of bad handling of the door and a risk for the integrity of the door.
This PI measures how rough (or undelicate) the robot is in operating the door.



## Performance Indicator: Capability Level
### Input

#### Pre-processed data:
 - events sequence

#### Testbed configuration values:
None

### Output
Number of steps of the benchmark procedure actually completed by the robot.
Each step is considered "completed" only after all the steps preceding it have been completed as well.

### Computation method
TBD

### Parameters
None

### Notes
Capability Level is an integer ranging from 0 to the number of steps composing the benchmark procedure.