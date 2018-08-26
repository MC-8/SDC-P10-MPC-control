# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

In this project I implemented a model predictive controller (MPC) for steering and throttle of a car to follow known waypoints in a race track.
The MPC utilizes optimization methods to calculate an optimal control sequence to reach our control objective (path following) considering the limitations of our car, that is takes the model into account to predict the outcome of the control sequence and use that to further produce control signals. 
An additional challenge in this project is the presence of a 100ms delay between the calculation of a control input and the execution of it (which simulates real-world scenario where actuation is not instantaneous).

[//]: # (Image References)
[model]:   ./model.png

## The Model
The model considered for the car is based on the bycicle model, where the car is approximated to a bycicle (that is one front wheel and one back wheel), no skidding, and limited turning rate.
The model implemented in the project is the following:

![alt text][success]

Where our state vector is ```x, y, psi, v, cte, epsi``` that are the vehicle position (x, y), heading (psi), longitudinal velocity (v), the cross track error (difference between the position of our vehicle and the path to be followed), and the heading error (epsi).

## Timestep Length and Elapsed Duration (N & dt)

For simplicity, the duration between timestep was selected to be 100 milliseconds, matching the actuator delay. This may be a bit too slow in real situations where control algorithm may run with a timestamp of 20 milliseconds, but it proved to produce good results and was not tweaked further.
The timestep length N was selected to be 10, that is our prediction horizon would be of 1 second.

## Polynomial Fitting and MPC Preprocessing

A third order polynomial is fitted to waypoints, 3rd order polynomials can represent many common trajectories and it was used to fit the desired trajectory as well.
Since all the coordinates in the system are represented in map coordinates (global), I've decided to convert all the cooridnates (waypoints and vehicle), to the car's reference frame. This can be seen for example in lines ```120``` to ```125``` in ```main.cpp```.
This conversion makes the following steps simpler to understand, as the velocity is longitudinal the car heading on the x axis and a 0 desired heading means that the heading is following the path correctly.

## Model Predictive Control with Latency

In order to cope with the 100ms latency, I've decided to pass to the MPC controller not the current state, but a predicted state based on the current state. 
In essence, knowing the current state and controls of our car, it is possible to predict where it would be 100ms later, so that the MPC optimizator will return control values that are useful 100ms in the future, so that by the time these values are used, they are exactly (or, within our best guess) what the car needs to follow the path. This can be observed in lines  ```137``` to ```150``` in ```main.cpp```.
The weights selected in the MPC optimization reflect our control desires.
In fact the minimization of cross-track error and heading error have a very high weight, also, we don't want the car to sway much suddenly from left to right so I selected a modest weight for sequential steering controls. Changes in acceleration and magnitude of steering have a low weight so the controller has more freedom on those, and limits on actuators help in not providing unreasonable controls to the car.

The controller was tested in the provided simulator with very good results even at high speed (60mph) with the car successfully follow the path without steering off track, and braking whenever turns are too sharp for the car to follow them at high speed.

---

## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1(mac, linux), 3.81(Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets
    cd uWebSockets
    git checkout e94b6e1
    ```
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.

* **Ipopt and CppAD:** Please refer to [this document](https://github.com/udacity/CarND-MPC-Project/blob/master/install_Ipopt_CppAD.md) for installation instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.)
4.  Tips for setting up your environment are available [here](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/23d376c7-0195-4276-bdf0-e02f1f3c665d)
5. **VM Latency:** Some students have reported differences in behavior using VM's ostensibly a result of latency.  Please let us know if issues arise as a result of a VM environment.

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!

More information is only accessible by people who are already enrolled in Term 2
of CarND. If you are enrolled, see [the project page](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/f1820894-8322-4bb3-81aa-b26b3c6dcbaf/lessons/b1ff3be0-c904-438e-aad3-2b5379f0e0c3/concepts/1a2255a0-e23c-44cf-8d41-39b8a3c8264a)
for instructions and the project rubric.

## Hints!

* You don't have to follow this directory structure, but if you do, your work
  will span all of the .cpp files here. Keep an eye out for TODOs.

## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to we ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./

## How to write a README
A well written README file can enhance your project and portfolio.  Develop your abilities to create professional README files by completing [this free course](https://www.udacity.com/course/writing-readmes--ud777).
