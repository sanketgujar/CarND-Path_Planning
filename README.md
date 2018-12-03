# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program

### Simulator.
You can download the Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2).

### Goals
In this project your goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. You will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

#### The map of the highway is in data/highway_map.txt
Each waypoint in the list contains  [x,y,s,dx,dy] values. x and y are the waypoint's map coordinate position, the s value is the distance along the road to get to that waypoint in meters, the dx and dy values define the unit normal vector pointing outward of the highway loop.

The highway's waypoints loop around so the frenet s value, distance along the road, goes from 0 to 6945.554.

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.

Here is the data provided from the Simulator to the C++ Program

#### Main car's localization Data (No Noise)

["x"] The car's x position in map coordinates

["y"] The car's y position in map coordinates

["s"] The car's s position in frenet coordinates

["d"] The car's d position in frenet coordinates

["yaw"] The car's yaw angle in the map

["speed"] The car's speed in MPH

#### Previous path data given to the Planner

//Note: Return the previous list but with processed points removed, can be a nice tool to show how far along
the path has processed since last time.

["previous_path_x"] The previous list of x points previously given to the simulator

["previous_path_y"] The previous list of y points previously given to the simulator

#### Previous path's end s and d values

["end_path_s"] The previous list's last point's frenet s value

["end_path_d"] The previous list's last point's frenet d value

#### Sensor Fusion Data, a list of all other car's attributes on the same side of the road. (No Noise)

["sensor_fusion"] A 2d vector of cars and then that car's [car's unique ID, car's x position in map coordinates, car's y position in map coordinates, car's x velocity in m/s, car's y velocity in m/s, car's s position in frenet coordinates, car's d position in frenet coordinates.

## Details

1. The car uses a perfect controller and will visit every (x,y) point it recieves in the list every .02 seconds. The units for the (x,y) points are in meters and the spacing of the points determines the speed of the car. The vector going from a point to the next point in the list dictates the angle of the car. Acceleration both in the tangential and normal directions is measured along with the jerk, the rate of change of total Acceleration. The (x,y) point paths that the planner recieves should not have a total acceleration that goes over 10 m/s^2, also the jerk should not go over 50 m/s^3. (NOTE: As this is BETA, these requirements might change. Also currently jerk is over a .02 second interval, it would probably be better to average total acceleration over 1 second and measure jerk from that.

2. There will be some latency between the simulator running and the path planner returning a path, with optimized code usually its not very long maybe just 1-3 time steps. During this delay the simulator will continue using points that it was last given, because of this its a good idea to store the last points you have used so you can have a smooth transition. previous_path_x, and previous_path_y can be helpful for this transition since they show the last points given to the simulator controller with the processed points already removed. You would either return a path that extends this previous path or make sure to create a new path that has a smooth transition with this last path.


## Spline
A really helpful resource for doing this project and creating smooth trajectories was using http://kluge.in-chemnitz.de/opensource/spline/, the spline function is in a single hearder file is really easy to use.

I added th file [src/spline.h](./scr/spline.h). Spline is the Cubic Spline interpolation implementation: a single .h file and you can use splines instead of polynomials.


## Safe Valid trajectories

### The car is able to drive at least 4.36 miles and an average of 15-20 miles without incident.

### The car drives according to the speed limit and no warning was seen.

### Car does not have collide with any other cars.

<!-- ### The car stays in its lane, except for the time between changing lanes. -->

### The car is able to change lanes


### Jerk and max acceleration are not Exceeded and no warning signal is seen.


## Reflection

The seed code was provided in the start of the project. I implemented everything in a single file just to avoid jumping in files due to lack of time. My implementation consist of three parts:

### Prediction [line 247-278](./src/main.cpp#L247)
It deals with sensor fusion data and  telemetry. It reasons about the surrounding and is curious about main three aspects:

- If there is a car to the right making a lane change not safe.
- If a car is in front of us blocking the traffic.
- Is there is a car to the left of us making a lane change not safe.

It is solved by calculating the lane each other car is and the position it will be at the end of the last plan trajectory.

A car will be considered "Dangerous" when its distance to our car is less than 30 meters in front or behind us.


### Behavior [line 279-299](./src/main.cpp#L279)
The main functions are:
1. decide to change lanes or not
2. decide to speed up or slow down

Based on the prediction of the situation we are in, this part is responsible for making all the actions until its safe to do so.

Instead of increasing the speed at this part of the code, a `speed_diff` is created to be used for speed changes when generating the trajectory. It makes the car more responsive towards fast changing situations when a car in front of it trying to apply breaks to cause a collision.

### Trajectory [line 301-349](./src/main.cpp#L301)
The objective of this part is to calculate the trajectory based on the speed and the lane layout, the surrounding car coordinates and path points.


1. The last two points of the previous trajectory are are used in conjunction three points at a far distance to initialize the spline calculation.
2. To make the work less complicated to the spline calculation based on those points, the coordinates are transformed (i.e. rotation and shift) to local car coordinates.
3. To ensure more continuity in the trajectory, the pass trajectory points are copied   to the new trajectory.
4. The rest of the points are calculated by evaluating the spline and transforming the output coordinates to not local coordinates.
5. The speed change is decided by the behavior module, but it is used in that part to increase/decrease speed on every trajectory points instead of doing it for the complete trajectory.


## Amendment

1. [Spline](http://kluge.in-chemnitz.de/opensource/spline/)
I added th file [src/spline.h](./scr/spline.h). Spline is the Cubic Spline interpolation implementation: a single .h file and you can use splines instead of polynomials. The spline implementation in mentioned in Trjectory module in Reflection
---
