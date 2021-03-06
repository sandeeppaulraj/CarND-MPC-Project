# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

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

## Reflection

I essentially followed the 3 main steps below to accomplish this

### Example Mind the Line Quiz

I leveraged a lot of the template code from this quiz.
Infact most of my project MPC.cpp is for all practical purposes the same as what was used in the quiz.

### Project Q&A in Google Hangout.

The Google hangout video gave a lot of hints/information on how to code the main.cpp.
After following the instructions and NOT incorporating latency into the model, i had issues with the car going of the track.
I realized that based on the project lesson and the project helper video that i had to modify the co-efficients of the cost function.
I had to experiment with several values to reach a stage where the car was able to lap the circuit without going off.
The co-efficients in the project Q7A video did not work for me.

### Add Latency

I consulted the forum to read up on how to add latency.
After going through various discussion threads, i settled on how to add latency to the model.

I chose a value of N = 10 and dt = 0.1

These values were based on suggestions given in the helper videos.
I will note however, that model is very sensitive to the co-efficients to minimize the value gap between sequential actuations.
For steering a value of 300 worked well for me; however a value of 100 and even 200 gave very poor results.


```sh
    for (unsigned int t = 0; t < N; t++) {
      fg[0] += 2000*CppAD::pow(vars[cte_start + t] - ref_cte, 2);
      fg[0] += 2000*CppAD::pow(vars[epsi_start + t] - ref_epsi, 2);
      fg[0] += CppAD::pow(vars[v_start + t] - ref_v, 2);
    }

    // Minimize the use of actuators.
    for (unsigned int t = 0; t < N - 1; t++) {
      fg[0] += 20*CppAD::pow(vars[delta_start + t], 2);
      fg[0] += 20*CppAD::pow(vars[a_start + t], 2);
    }

    // Minimize the value gap between sequential actuations.
    for (unsigned int t = 0; t < N - 2; t++) {
      fg[0] += 300*CppAD::pow(vars[delta_start + t + 1] - vars[delta_start + t], 2);
      fg[0] += 15*CppAD::pow(vars[a_start + t + 1] - vars[a_start + t], 2);
    }
```

For adding latency, i did the following simple update.
I decided to add code for latency only after my car able to go around the lap without going off the circuit.

The update i made for latency is as follows. I used the kinematic equations.

```sh
	px = px + v * cos(psi) * dt;
	py = py + v * sin(psi) * dt;
	psi = psi - v * delta / Lf * dt;
	v = v + a * dt;
```

I also made small modifications as suggested in the projects tips and tricks section to modify the psi update.

These are the snippets were the relevant updates were reflected.

```sh
	psi = psi - v * delta / Lf * dt;
```

```sh
	// VVIMP: From Lesson update the sign of the equation below.
	// Refer TIPS and Tricks section
	fg[1 + psi_start + t] = psi1 - (psi0 - v0 * delta0 / Lf * dt);

	// VVIMP: From Lesson update the sign of the equation below.
	// Refer TIPS and Tricks section
	fg[1 + epsi_start + t] = epsi1 - ((psi0 - psides0) - v0 * delta0 / Lf * dt);
```