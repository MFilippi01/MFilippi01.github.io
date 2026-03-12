---
title: "Bayesian Optimization for PID Tuning of a 7-DoF Robotic Manipulator"
collection: projects
permalink: /projects/bayes-opt-pid-tuning/
date: 2025-07-11
layout: single
author_profile: true
toc: true
toc_sticky: true
classes: wide
excerpt: "Bayesian Optimization framework for tuning PID gains and estimating dynamic parameters of a 7-DoF Franka Panda robot, enabling efficient exploration of high-dimensional control parameters.."
---

## Project Overview

This project investigates the use of **Bayesian Optimization** to automatically tune the **PID gains of a 7-DoF Franka Panda robotic manipulator** for accurate trajectory tracking.  
The optimization problem involves **21 controller parameters**, making manual tuning impractical.
Two optimization strategies were explored: a **one-shot approach**, where all gains are optimized simultaneously, and a **cascade approach**, where joints are tuned sequentially to reduce the optimization dimensionality.  
Two cost functions were also considered: one defined in **joint space** and one in **workspace**.
The reference trajectory and velcoity profiles used during the optimization are shown below.

<div style="display: flex; gap: 20px; justify-content: center; align-items: flex-start;">

<div style="text-align: center;">
<img src="/images/reference_trajectories.jpg" width="450">
<figcaption>Reference trajectory profiles.</figcaption>
</div>

<div style="text-align: center;">
<img src="/images/reference_trajectories_velocity.jpg" width="450">
<figcaption>Reference velocity profile.</figcaption>
</div>

</div>

The robot dynamics used in the simulations follow the standard rigid-body formulation:

$$
B(q)\ddot{q} + C(q,\dot{q})\dot{q} + g(q) = \tau
$$


## Bayesian Optimization Setup

The controller tuning was performed using **MATLAB’s `bayesopt`** function.  
The optimization problem involves **21 parameters**, corresponding to the proportional, integral, and derivative gains of the seven robot joints.
Bayesian Optimization was used to efficiently explore this **high-dimensional parameter space** through a **Gaussian Process (GP) surrogate model**, which approximates the objective function and guides the search toward promising regions.
A **two-phase strategy** was adopted to balance exploration and exploitation.

### Exploration Phase

The optimization initially performs a broad search using the **Lower Confidence Bound (LCB)** acquisition function.  
A fixed number of **seed points** is first evaluated to initialize the Gaussian Process model and provide an initial approximation of the objective landscape.

### Exploitation Phase

Once sufficient samples have been collected, the optimization switches to the **Expected Improvement (EI)** acquisition function.  
The Gaussian Process is initialized using the **XTrace samples and objective values obtained during the exploration phase**, allowing the optimizer to focus the search on regions likely to improve the current best solution.


## One-Shot Optimization

In the **one-shot approach**, all **21 PID gains** are optimized simultaneously.  
This allows the optimizer to directly account for the **coupling effects between joints**, potentially leading to a better global optimum.

### Cost Function in Joint Space

The joint-space cost function used to guide the optimization is defined as:

$$
J = \mathbf{w}^{T}\sum_{i=1}^{7}\boldsymbol{\epsilon}^{i} + p_{max} + p_{prop}
$$

where:

$$
\mathbf{w} =
\begin{bmatrix}
w_{e_{max}} \\
w_{\dot e_{max}} \\
w_{e_{avg}} \\
w_{\dot e_{avg}} \\
w_{\ddot e_{avg}} \\
w_{\tau_{max}}
\end{bmatrix},
\qquad
\boldsymbol{\epsilon}^{i} =
\begin{bmatrix}
e^{i}_{max} \\
\dot e^{i}_{max} \\
e^{i}_{avg} \\
\dot e^{i}_{avg} \\
\ddot e^{i}_{avg} \\
\tau^{i}_{max}
\end{bmatrix}
$$

The term \(e^i\) denotes the angular tracking error of the \(i^{th}\) joint.
The resulting controller provides very accurate **joint-space tracking**, as shown in the plots below.

<div style="display: flex; gap: 20px; justify-content: center; align-items: flex-start;">

<div style="text-align: center;">
<img src="/images/joint_tracking_initial.jpg" width="450">
</div>

<div style="text-align: center;">
<img src="/images/joint_tracking_initial_velocity.jpg" width="450">
</div>

</div>

However, the optimized controller produced **very large torque peaks**, particularly in the first joints, reaching values **above 500 Nm**, which are not physically feasible for the robot.

---

### Motor Saturation and Anti-Windup

To ensure realistic behavior, **motor torque limits** consistent with the Franka Panda specifications were introduced in the simulation.

<p align="center">
<table style="border-collapse: collapse; text-align: center;">
  <thead style="background-color:#1f3a5f; color:white;">
    <tr>
      <th style="padding:8px;">Motor</th>
      <th style="padding:8px;">Joint 1</th>
      <th style="padding:8px;">Joint 2</th>
      <th style="padding:8px;">Joint 3</th>
      <th style="padding:8px;">Joint 4</th>
      <th style="padding:8px;">Joint 5</th>
      <th style="padding:8px;">Joint 6</th>
      <th style="padding:8px;">Joint 7</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding:8px; text-align:left;"><b>Max torque</b></td>
      <td>87 Nm</td>
      <td>87 Nm</td>
      <td>87 Nm</td>
      <td>87 Nm</td>
      <td>12 Nm</td>
      <td>12 Nm</td>
      <td>12 Nm</td>
    </tr>
  </tbody>
</table>
</p>

Additionally, an **anti-windup mechanism** was implemented to prevent the integral term of the PID controller from accumulating excessive error when the actuator saturation occurs.

With these modifications, the controller behavior becomes physically consistent while maintaining good tracking performance.

<div style="display: flex; gap: 20px; justify-content: center; align-items: flex-start;">

<div style="text-align: center;">
<img src="/images/joint_tracking_initial_saturated.jpg" width="450">
</div>

<div style="text-align: center;">
<img src="/images/joint_tracking_initial_velocity_saturated.jpg" width="450">
</div>

</div>

---

### Reference Trajectory Refinement

An additional issue was observed at the beginning of the motion.  
The original reference trajectory started with a **non-zero initial velocity**, which caused an unavoidable tracking error during the first instants of motion.

To eliminate this mismatch, the initial part of the sinusoidal trajectory was replaced with a **6th-order polynomial**, ensuring **zero initial velocity**.

<div style="display: flex; gap: 20px; justify-content: center; align-items: flex-start;">

<div style="text-align: center;">
<img src="/images/trayectory_new_reference.jpg" width="450">
</div>

<div style="text-align: center;">
<img src="/images/velocity_new_reference.jpg" width="450">
</div>

</div>

The controller was then re-optimized using the updated reference trajectory, resulting in improved initial tracking behavior.

<div style="display: flex; gap: 20px; justify-content: center; align-items: flex-start;">

<div style="text-align: center;">
<img src="/images/joint_tracking_initial_saturated_corrected.jpg" width="450">
</div>

<div style="text-align: center;">
<img src="/images/joint_tracking_initial_velocity_saturated_corrected.jpg" width="450">
</div>

</div>

---

### Cost Function in Workspace


A second objective function was defined in **workspace**, minimizing the tracking error of the **end-effector position**.

$$
J = \mathbf{w}^{T}\sum_{i=1}^{3}\boldsymbol{\epsilon}^{i} + p_{max} + p_{prop}
$$

where:

$$
\mathbf{w} =
\begin{bmatrix}
w_{e_{max}} \\
w_{\dot e_{max}} \\
w_{e_{avg}} \\
w_{\dot e_{avg}} \\
w_{\ddot e_{avg}} \\
w_{\tau_{max}}
\end{bmatrix},
\qquad
\boldsymbol{\epsilon}^{i} =
\begin{bmatrix}
e^{i}_{max} \\
\dot e^{i}_{max} \\
e^{i}_{avg} \\
\dot e^{i}_{avg} \\
\ddot e^{i}_{avg} \\
\tau^{i}_{max}
\end{bmatrix}
$$

The term $e^i$ denotes the **positional tracking error of the end-effector** along the $i$-th Cartesian axis.
This formulation prioritizes accurate tracking of the end-effector trajectory.

<div style="display: flex; gap: 20px; justify-content: center; align-items: flex-start;">

<div style="text-align: center;">
<img src="/images/cartesian_tracking_initial_saturated_corrected.jpg" width="450">
</div>

<div style="text-align: center;">
<img src="/images/cartesian_tracking_initial_velocity_saturated_corrected.jpg" width="450">
</div>

</div>

As expected, this cost function produces **better tracking in workspace**, but slightly degrades the **tracking accuracy in joint space**, highlighting the trade-off between joint-level and task-space optimization.


## Cascade Optimization

To reduce the complexity of the optimization problem, a **cascade optimization strategy** was also investigated.  
Instead of optimizing all **21 PID gains simultaneously**, the controller parameters are tuned **sequentially joint by joint**, reducing the dimensionality of each optimization step.
The procedure starts from the **outermost joint** and progressively moves toward the **base of the manipulator**.  
At each step, only the PID gains of the current joint are optimized, while the gains of the previously tuned joints remain fixed. This approach decomposes the original high-dimensional problem into a series of smaller and more tractable optimization tasks.
The cascade approach relies on the assumption that the **coupling effects between joints are limited**, particularly when moving from the end-effector toward the base. Under this assumption, the behavior of the already tuned joints does not significantly affect the optimization of the remaining ones.
The resulting controller achieves stable and accurate tracking, as shown in the plots below.

<div style="display: flex; gap: 20px; justify-content: center; align-items: flex-start;">

<div style="text-align: center;">
<img src="/images/cascade_joint_tracking.jpg" width="450">
</div>

<div style="text-align: center;">
<img src="/images/cascade_joint_tracking_velocities.jpg" width="450">
</div>

</div>

Compared with the **one-shot optimization**, the cascade approach simplifies the optimization process and improves convergence robustness. However, because it does not fully account for the coupling between joints, it may lead to a slightly higher overall tracking error than the global one-shot solution.


## Mass Estimation and PID Tuning

In many practical scenarios, the **link masses** required by the dynamic model are not precisely known.  
Since these parameters directly affect gravity and Coriolis compensation, inaccurate mass values can degrade the tracking performance of the controller.
To address this issue, **Bayesian Optimization** was also used to estimate the unknown masses.  
The estimation procedure is based on the robot dynamic model and exploits simulations in which **known joint torques** are applied. The masses are then tuned so that the **accelerations predicted by the model** match the reference accelerations as closely as possible.
The control input used for the estimation is defined as:

$$
\tau = B(q,m)\ddot{q}_d + C(q,\dot{q},m)\dot{q} + g(q,m)
$$

If the masses are correctly estimated, the simulated dynamics satisfy:

$$
B(q)\ddot{q} + C(q,\dot{q})\dot{q} + g(q)
=
B(q,m)\ddot{q}_d + C(q,\dot{q},m)\dot{q} + g(q,m)
\qquad \Rightarrow \qquad
\ddot{q} = \ddot{q}_d
$$

A cost function is therefore defined to penalize the difference between the **simulated accelerations** and the **reference accelerations**, and the unknown mass parameters are optimized accordingly.
It is important to note that **links 2, 4, and 6 were not estimated**, since their mass contribution is set to zero in the adopted dynamic model.

The estimated masses are reported below.

<p align="center">
<table style="border-collapse: collapse; text-align: center;">
  <thead style="background-color:#1f3a5f; color:white;">
    <tr>
      <th style="padding:8px;">Link</th>
      <th style="padding:8px;">Link 1</th>
      <th style="padding:8px;">Link 2</th>
      <th style="padding:8px;">Link 3</th>
      <th style="padding:8px;">Link 4</th>
      <th style="padding:8px;">Link 5</th>
      <th style="padding:8px;">Link 6</th>
      <th style="padding:8px;">Link 7</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding:8px; text-align:left;"><b>Real mass [kg]</b></td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
      <td>2.5</td>
    </tr>
    <tr>
      <td style="padding:8px; text-align:left;"><b>Estimated mass [kg]</b></td>
      <td>1.0031</td>
      <td>/</td>
      <td>2.9972</td>
      <td>/</td>
      <td>5.0056</td>
      <td>/</td>
      <td>2.4991</td>
    </tr>
  </tbody>
</table>
</p>

The results of the mass estimation process are illustrated below.

<div style="display: flex; gap: 20px; justify-content: center; align-items: flex-start;">

<div style="text-align: center;">
<img src="/images/mass_estimation_bayesopt.jpg" width="450">
<figcaption>Mass estimates across Bayesian optimization iterations.</figcaption>
</div>

<div style="text-align: center;">
<img src="/images/acceleration_tracking.jpg" width="450">
<figcaption>Acceleration tracking with estimated masses.</figcaption>
</div>

</div>

Once the mass values were identified, the **PID tuning procedure was repeated** using the updated dynamic model.  
This second stage made it possible to improve the controller tuning by relying on a more accurate model of the manipulator dynamics.

<div style="display: flex; gap: 20px; justify-content: center; align-items: flex-start;">

<div style="text-align: center;">
<img src="/images/pid_tracking_estimated_masses.jpg" width="450">
<figcaption>Joint trajectory tracking.</figcaption>
</div>

<div style="text-align: center;">
<img src="/images/pid_velocity_tracking_estimated_masses.jpg" width="450">
<figcaption>Joint velocity tracking.</figcaption>
</div>

</div>



## Results Comparison

This section summarizes the performance obtained with the different cost functions and optimization strategies.

### Cost Functions Comparison

The performance obtained with the two cost functions is summarized in the tables below.

<p align="center">
<table style="border-collapse: collapse; text-align: center;">
  <thead style="background-color:#1f3a5f; color:white;">
    <tr>
      <th style="padding:8px;">Metric</th>
      <th style="padding:8px;">e<sup>θ</sup><sub>max</sub> [rad]</th>
      <th style="padding:8px;">e<sup>θ</sup><sub>avg</sub> [rad]</th>
      <th style="padding:8px;">ė<sup>θ</sup><sub>max</sub> [rad/s]</th>
      <th style="padding:8px;">ė<sup>θ</sup><sub>avg</sub> [rad/s]</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding:8px; text-align:left;"><b>Joint-space cost</b></td>
      <td style="color:#6DBE45;"><b>0.0659</b></td>
      <td style="color:#6DBE45;"><b>0.0317</b></td>
      <td style="color:#6DBE45;"><b>62.8319</b></td>
      <td style="color:#6DBE45;"><b>0.2372</b></td>
    </tr>
    <tr>
      <td style="padding:8px; text-align:left;"><b>Workspace cost</b></td>
      <td style="color:red;"><b>0.2136</b></td>
      <td style="color:red;"><b>0.0426</b></td>
      <td style="color:red;"><b>62.8320</b></td>
      <td style="color:red;"><b>0.2831</b></td>
    </tr>
  </tbody>
</table>
</p>

<p align="center">
<table style="border-collapse: collapse; text-align: center;">
  <thead style="background-color:#1f3a5f; color:white;">
    <tr>
      <th style="padding:8px;">Metric</th>
      <th style="padding:8px;">e<sup>x</sup><sub>max</sub> [m]</th>
      <th style="padding:8px;">e<sup>x</sup><sub>avg</sub> [m]</th>
      <th style="padding:8px;">ė<sup>x</sup><sub>max</sub> [m/s]</th>
      <th style="padding:8px;">ė<sup>x</sup><sub>avg</sub> [m/s]</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding:8px; text-align:left;"><b>Joint-space cost</b></td>
      <td style="color:red;"><b>7.22 · 10<sup>-4</sup></b></td>
      <td style="color:red;"><b>1.33 · 10<sup>-4</sup></b></td>
      <td style="color:red;"><b>0.1423</b></td>
      <td style="color:red;"><b>0.0012</b></td>
    </tr>
    <tr>
      <td style="padding:8px; text-align:left;"><b>Workspace cost</b></td>
      <td style="color:#6DBE45;"><b>2.23 · 10<sup>-4</sup></b></td>
      <td style="color:#6DBE45;"><b>1.22 · 10<sup>-4</sup></b></td>
      <td style="color:#6DBE45;"><b>0.1412</b></td>
      <td style="color:#6DBE45;"><b>9.15 · 10<sup>-4</sup></b></td>
    </tr>
  </tbody>
</table>
</p>

The results clearly highlight the trade-off between the two objective formulations.  
The **joint-space cost function** produces the lowest joint tracking errors, as expected, while the **workspace cost function** provides better accuracy in the end-effector motion.



### Optimization Approaches Comparison

The performance of the two optimization strategies, **one-shot** and **cascade optimization**, is compared in the figure below.

<p align="center">
<img src="/images/one_shot_vs_cascade.JPG" width="70%">
<figcaption style="text-align:center;">Comparison between one-shot and cascade optimization performance.</figcaption>
</p>

The results show that **one-shot optimization** achieves a better global optimum.  
By optimizing all controller parameters simultaneously, this approach is able to account for the **coupling effects between the robot joints**, leading to lower overall tracking errors.
The **cascade approach**, on the other hand, decomposes the tuning problem into smaller sequential optimizations.  
This makes the procedure **more stable and easier to manage**, and reduces the dimensionality of each optimization step.  
However, since the coupling between joints is only partially captured, the final solution is generally **less optimal** than the one obtained with the one-shot strategy.



## Conclusions

This project demonstrated the effectiveness of **Bayesian Optimization** for tuning the PID gains of a high-dimensional robotic control problem.  
The approach allows efficient exploration of the **21-dimensional parameter space**, enabling automatic controller tuning without requiring manual parameter adjustments.
The comparison between the optimization strategies showed that **one-shot optimization** achieves the best overall performance, as it captures the **coupling effects between the robot joints** and allows the optimizer to reach a better global optimum.
The **cascade approach**, although slightly less optimal, reduces the dimensionality of the optimization problem and results in a **more stable and computationally manageable tuning process**.
Future work could extend this framework toward **online controller tuning** or **adaptive control strategies**, allowing the robot to continuously update its controller parameters in response to changes in the system dynamics.

