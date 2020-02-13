# BEAST Benchmark procedure

<p align="center">
  <img src="/img/beast_arena_checkpoints.png?raw=true" width="689" alt="Arena with checkpoints">
</p>
<p align="center">
 Arena with checkpoints
</p>


## Terminology
TODO

## Procedure
In order to execute the benchmark, the robot must execute, in order, the following steps:

1. Grasp the walker/trolley's handle. **Optional: Robot can start with end effectors already positioned on handle**

2. Navigate with the walker/trolley to checkpoint 1.

3. Navigate with the walker/trolley to checkpoint 2 with minimum deviation from a straight trajectory.

4. Navigate with the walker/trolley to checkpoints 3, 4 and 5.

For the benchmark to start, the robot must first be set in a **start pose** specified by the referees. Once this precondition is satisfied, the benchmark starts in a time instant specified by the referees.

The benchmark stops when manually interrupted by the referee via the GUI.


# BEAST Testbed configuration values

The following options are the testbed and benchmark configuration:

 - `start_already_gripping`: `False` if the humanoid must autonomously grasp the handle after the start of the benchmark [bool] 
 - `disturbance_type`: Type of disturbance introduced through the walker/trolley actuators. **To be defined** [enum/string]
 - `load`: Amount of additional weight added to the walker/trolley [grams]


# BEAST pre-processed data

Pre-processed data are computed from the topics provided by the testbed and the inputs of the Performance Indicators.

## Pre-processed data: events sequence
Time series containing detectable events as values.

Columns: timestamp [float] (absolute ros time: seconds since epoch), event [string/enum].

Events:
 - benchmark start: identifies the start of benchmark execution. This event is always present. The timing of this event is affected by a delay due to the people operating the robot having to manually start the robot.
 - handle is touched: event occurs when handle force signal exceeds a predefined threshold. The threshold needs manual calibration. Only one occurrence of this event can be present.
 - checkpoint_1: walker/trolley reaches checkpoint 1
 - checkpoint_2: walker/trolley reaches checkpoint 2
 - checkpoint_3: walker/trolley reaches checkpoint 3 (to ensure the robot starts the slalom by keeping the first cone on the right)
 - checkpoint_4: walker/trolley reaches checkpoint 4
 - checkpoint_5: walker/trolley reaches checkpoint 5
 - checkpoint_6: walker/trolley reaches checkpoint 6
 - checkpoint_7: walker/trolley reaches checkpoint 7 (corresponding to checkpoint 1)
 - benchmark stop: identifies the end of benchmark execution. This event is always present.

Note:
Checkpoints are identified by two points and a direction.
Reaching a checkpoint means the odometry center of the walker/trolley (base_link) passes between the two points of the checkpoint.
Direction of movement must also be considered (otherwise a humanoid may reach a checkpoint from the wrong side to minimise the time and distance of its trajectory).


## Pre-processed data: force on handle
Copy of raw data from topic `/beast/handle/force`.

Columns: timestamp [float] (absolute ros time: seconds since epoch), force [grams].


## Pre-processed data: velocity
Copy of raw data from topic `/odom`.

Columns: timestamp [float] (absolute ros time: seconds since epoch), x [m], y [m], theta [rad].

Linear and angular velocity of the walker/trolley with respect to the testbed origin reference frame.
The x, y, theta values follow the amcl ROS package convention.


## Pre-processed data: pose
Copy of raw data from tf frame `base_link`.

Columns: timestamp [float] (absolute ros time: seconds since epoch), x [m], y [m], theta [rad].

Position and orientation of the walker/trolley with respect to the testbed origin reference frame.
The x, y, theta values follow the amcl ROS package convention.


## Pre-processed data: obstacle distance
Computed from topic `/scan`.
Minimum distance from the obstacles visible in the laser scan.

Computed as `min(r), for any range measurement r not intersecting the walker/trolley nor the humanoid`. 

Columns: timestamp [float] (absolute ros time: seconds since epoch), distance [m].



# BEAST Performance Indicators

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

#### Pre-processed data:
 - events_sequence

#### Testbed configuration values:
 - start_already_gripping

### Output
Overall duration of benchmark execution.

### Computation method
The output is the difference between the times of events `checkpoint_1` or `benchmark_start` (depending on `start_already_gripping`), and the last event in the `events_sequence`.

### Parameters
None.

### Notes
Execution Time cannot exceed the timeout.
This measure uses the `start benchmark` event to take into account the time employed by the humanoid to perceive the walker/trolley and plan its actions.



## Performance Indicator: Time to handle
### Input

#### Pre-processed data:
 - events sequence

#### Testbed configuration values:
 - start_already_gripping

### Output
Time elapsed from the start of the benchmark to the first time the handle is touched by the robot/humanoid. This PI accounts for the time the robot takes to perceive the walker/trolley's handle, plan its actions and grasp the handle.

### Computation method
Time elapsed from event `benchmark start` to first event `handle is touched`.

Note: What if the robot never reaches the final step? should the result be the value of timeout or `NaN`? 

### Parameters
None

### Notes
Time To Handle cannot exceed the timeout.



## Performance Indicator: Straight time
### Input

#### Pre-processed data:
 - events sequence
 
#### Testbed configuration:
None

### Output
Time elapsed on the straight segment between checkpoint 1 and checkpoint 2.

### Computation method
Time elapsed from event `checkpoint_1` to event `checkpoint_2`.

Note: What if the robot never reaches the final step? should the result be the value of timeout or `NaN`? 

### Parameters
None

### Notes
None



## Performance Indicator: Slalom time
### Input

#### Pre-processed data:
 - events sequence
 
#### Testbed configuration:
None

### Output
Time elapsed on the slalom segments between checkpoint 2 and checkpoint 7.

### Computation method
Time elapsed from event `checkpoint_2` to event `checkpoint_5`.

Note: What if the robot never reaches the final step? should the result be the value of timeout or `NaN`? 

### Parameters
None

### Notes
None


## Performance Indicator: Straight control accuracy
### Input

#### Pre-processed data:
 - pose

#### Testbed configuration values:
None

### Output
This PI is a measurement of the accuracy of controlling the walker/trolley by the humanoid based on the trajectory of the walker/trolley on the straight segment.

### Computation method
Area between the trajectory of the walker/trolley and the Least Squares Regression Line fitted to the trajectory points.

### Parameters
None

### Notes
None


## Performance Indicator: Safety of navigation 
### Input

#### Pre-processed data:
 - obstacle distance

#### Testbed configuration values:
None

### Output
Minimum distance in the `obstacle distance` timeseries

### Computation method
`min(r), for each value of obstacle distance r`

### Parameters
None

### Notes
The value is naturally limited by the distance between the closest cones between which the humanoid must pass with the walker/trolley.



## Performance Indicator: Capability Level
### Input

#### Pre-processed data:
 - events sequence

#### Testbed configuration values:
 - start_already_gripping

### Output
Number of steps of the benchmark procedure actually completed by the robot.
Each step is considered "completed" only after all the steps preceding it have been completed as well.
If `start_already_gripping` is true then the step about grasping the handle is ignored (the minimum score is still zero, the maximum score is one less).

### Computation method
TBD

### Parameters
None

### Notes
Capability Level is an integer ranging from 0 to the number of steps composing the benchmark procedure.
