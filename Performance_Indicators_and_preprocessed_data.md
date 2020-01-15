# MADROB Benchmark procedure

Note: this procedure intentionally resembles very closely the one used in the ERL benchmark "Through the Door" benchmark. 
Please note that, notwithstanding this parallelism, performance evaluation metrics used by MADROB do not correspond to those used by the ERL.

1. Touch the door handle.
2. Unlock the door (currently not detectable, requires additional sensor).
3. Open the door.
4. Cross the threshold.
5. Close the door (without crossing the threshold again).
6. Lock the door (currently not detectable, requires additional sensor) (without crossing the threshold again).

When the benchmark starts, the robot is in a predefined start pose. The benchmark stops when the first of the following two events occurs:
a. the last operation of the benchmark is completed (i.e., the door is locked again after the robot passed through it);
b. the timeout has elapsed.

The STARTING SIDE is the side of the door where the robot is at the start of the benchmark. The DESTINATION SIDE is the side of the door that the robot must reach by going through the door.


# MADROB Pre-processed data

## Pre-processed data: event sequence
Time series containing detectable events as values.
Columns: timestamp (absolute ros time: seconds since epoc) [float], event [string/enum].
Events:
 - benchmark start: identifies the start of the benchmark. This event is always present. The timing of this event is affected by a delay due to the people operating the robot having to manually start the robot.
 - door opens: `|door angular position| > threshold for closed door`, given door was previously closed. The threshold needs manual calibration.
 - door closes: `|door angular position| < threshold for closed door`, given door was previously open. The threshold needs manual calibration.
 - handle is touched: event occurs when force signal exceeds a predefined threshold. The threshold needs manual calibration.
 - humanoid moves to cw side, humanoid moves to ccw side: last detection of humanoid moving from one side to the other by passage sensors. Only one occurrence of these events can be present.
 - humanoid approaches the door on cw side, humanoid approaches the door on ccw side: first detection of a humanoid under the passage sensors. Only one occurrence of these events can be present.
 - door is locked, door is unlocked: currently not implemented, requires addition of latch sensor to the testbed.

## Pre-processed data: force on handle
Copy of raw data from topic /madrob/handle/force. Noise reduction may be applied (if strictly necessary).
Columns: timestamp (absolute ros time: seconds since epoc) [float], force [grams].

## Pre-processed data: door angular position
Copy of raw data from topic /madrob/door/angle. Noise reduction may be applied (if strictly necessary).
Columns: timestamp (absolute ros time: seconds since epoc) [float], door angular position [rad].

## Pre-processed data: door angular acceleration
Copy of raw data from topic /madrob/door/velocity. Noise reduction may be applied (if strictly necessary).
Columns: timestamp (absolute ros time: seconds since epoc) [float], door angular acceleration [rad/(s^2)].
Note: It's probably better to compute this values by deriving twice door angular position. The velocity in /madrob/door/velocity may not be appropriate for this use.


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

pre-processed data:
 - event sequence

Testbed configuration values:
 - lock direction
 - destination side TODO

### Output
Overall duration of benchmark execution.

### Computation method
The final event is considered as the last 'door closes' event, occurring after an event 'humanoid moves to cw side' or 'humanoid moves to ccw side', depending on destination side for the run.
If the final event is not present in the timeseries, the result is considered a timeout.
The output is the difference between time of event 'benchmark start' and time of the final event.

### Parameters
None.

### Notes
Execution Time cannot exceed the timeout.
This measure uses the 'start benchmark' event to take into account the time employed by the humanoid to percieve the door and plan its actions.


## Performance Indicator: Time to handle
### Input

Pre-processed data:
 - event sequence

### Output
Time To Handle

### Computation method
Time elapsed from event 'benchmark start' to event 'handle is touched'.

### Parameters
Filtering (e.g., low-pass) and load cell signal threshold must be determined in order to reliably detect touch.

### Notes
Time To Handle cannot exceed the timeout.


## Performance Indicator: Door Occupation Time
### Input

Pre-processed data:
 - event sequence
 
Testbed configuration:
 - lock direction
 - destination side TODO

### Output
Door Occupation Time

### Computation method
Time elapsed from event 'humanoid approaches the door on [c]cw side' to event 'humanoid moves to [c]cw side'.

### Parameters
The process to generate proximity signals must be fine-tuned considering noise filtering and door geometry (data from sensors which -given the angular position of the door- may be perceiving the door panel must be discarded)

### Notes
Door Occupation Time cannot exceed the timeout.
A door threshold is a bottleneck in people mobility between environments. Door Occupation Time evaluates how good the robot is in limiting its use of this limited resource.
This measure may be affected by the robot shape since it relies on the passage sensors, so it may not be a good indicator.



## Performance Indicator: Passage time
### Input

pre-processed data:
 - event sequence

Testbed configuration values:
 - lock direction
 - destination side TODO

### Output
Time it takes the humanoid to operate the door, reach the destination side and close the door.

### Computation method
The final event is considered as the last 'door closes' event, occurring after an event 'humanoid moves to cw side' or 'humanoid moves to ccw side', depending on destination side for the run.
If the final event is not present in the timeseries, the result is considered a timeout.
The output is the difference between time of event 'handle is touched' and time of the final event.

### Parameters
None.

### Notes
Execution Time cannot exceed the timeout.
This measure is similar to the PIs Door occupation time and Overall execution time, but avoids using the timing of the events 'benchmark start' and passage events, that rely on manual timing (starting benchmark and the robot at the same time), and may give results skewed by the robot shape (laser sensors may provide different timings based on the height/shape of the robot).
The different with the PI Overall execution time is that the time took by the robot to perceive the door and plan its actions is excluded.

## Performance Indicator: Maximum door acceleration
### Input

Pre-processed data:
 - door angular acceleration (Filtered?)

### Output
Maximum door acceleration

### Computation method
`max(a), for each value of angular acceleration a`

### Parameters
Parameters of filter(s) applied to angular acceleration signal (see Notes)

### Notes
High angular accelerations correspond to high forces exerted on any object or person touching the door panel. The smoother the door panel movement, the lower the risk of serious incidents. Thus, this indicator is a measurement of the safety of door management by the robot. 
High angular acceleration can indicate tremors in the robot's hand effectors. However, these are not much significant for safety because of the limited range of motion they impose to the door panel. Therefore, it is necessary to introduce a degree of low-pass filtering of the acceleration data. Such filtering also helps avoid artifacts due to the derivation of acceleration from discrete-time velocity data.


## Performance Indicator: Smoothness of door actuation
### Input

Pre-processed data:
 - door angular acceleration

### Output
Smoothness of door actuation

### Computation method
`1/sqrt(sum(a^2)), for each value of angular acceleration a`

### Parameters
None

### Notes
To operate the door smoothly the humanoid should minimise the acceleration of the door. The acceleration values are squared to bias the indicator against higher values of acceleration.


## Performance Indicator: Maximum force on handle
### Input

Pre-processed data:
 - force on handle

### Output
Maximum value of force applied to the handle

### Computation method
`max(f), for each value of force f`

### Parameters
None

### Notes
High force applied to the handle is a symptom of bad handling of the door and a risk for the integrity of the door. Force On Handle measures how delicate the robot is in operating the door.


## Performance Indicator: Capability Level
### Input

Pre-processed data:
 - events sequence

### Output
Number of steps of the benchmark actually completed by the robot. Each step is considered "completed" only after all the steps preceding it have been completed as well.

### Computation method
TBD

### Parameters
The process to generate proximity signals must be fine-tuned considering noise filtering and door geometry (data from sensors which -given the angular position of the door- may be perceiving the door panel must be discarded)

### Notes
Capability Level is an integer ranging from 0 to the number of steps composing the benchmark procedure.
Reliable computation of this PI can require direct sensing of door latch presence within the door latch panel affixed to the door frame (feasibility to be verified with Nova Labs)
