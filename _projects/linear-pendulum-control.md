---
title: "Linear Pendulum Control"
collection: projects
permalink: /projects/linear-pendulum-control/
date: 2026-02-06
layout: single
author_profile: true
toc: true
toc_sticky: true
classes: wide
excerpt: "Modeling, linearization, and control of a pendulum system using state-space methods and feedback control design."
---

<p>
<a href="/files/linear-pendulum-control.pdf" class="btn btn--primary" target="_blank">Download Full Document (PDF)</a>
</p>

## Project Overview

This project was developed as part of the **Mechatronics Laboratory course** and carried out as a **group project**.  
Its objective was the **modeling, simulation, and control of a linear inverted pendulum system**, a classical benchmark problem in control engineering due to its nonlinear dynamics and instability in the upright position.
The system consists of a **pendulum mounted on a motorized cart moving along a horizontal rail**. The project included the derivation of the system model, **experimental parameter identification**, development of a **Simulink simulation model**, and the design of different control strategies such as **PID and LQR controllers**.  
Additional work included **signal filtering** to handle measurement noise and the implementation of an **energy-based swing-up control** to bring the pendulum from the downward to the upright equilibrium.

<p align="center"> <img src="/images/pendulum_system_setup.png" width="65%"> </p>


## System Modeling

The analytical model of the system was derived using the **Lagrangian approach**.  
The system has two degrees of freedom: the **horizontal displacement of the cart** \(x\) and the **pendulum angle** \(\theta\).

<p align="center"> <img src="/images/pendulum_system_model.png" width="65%"> </p>

By applying the Lagrange equations, the following **two coupled nonlinear equations of motion** are obtained:

$$
\begin{cases}
\left( M + m \right) \ddot{x} + m h \cos \theta \, \ddot{\theta} + b_x \dot{x} - m h \sin \theta \, \dot{\theta}^2 = F \\[6pt]
m h \cos \theta \, \ddot{x} + \left( J + m h^2 \right) \ddot{\theta} + b_\theta \dot{\theta} + m g h \sin \theta = 0
\end{cases}
$$

The system can then be rewritten in **state-space form** by defining the state vector and input as:

$$
\mathbf{x} =
\begin{bmatrix}
x_1 \\ x_2 \\ x_3 \\ x_4
\end{bmatrix}
=
\begin{bmatrix}
x \\
\dot{x} \\
\theta \\
\dot{\theta}
\end{bmatrix},
\qquad
u = F
$$

which leads to the nonlinear state-space representation:

$$
\begin{cases}
\dot{x}_1 = x_2 \\[8pt]
\dot{x}_2 =
\frac{\left( u - b_x x_2 + mh \sin x_3 \, x_4^2 \right)\left( J + mh^2 \right) + \left( b_\theta x_4 + mgh \sin x_3 \right) mh \cos x_3}
{\left( M+m \right)\left( J+mh^2 \right) - \left( mh \cos x_3 \right)^2} \\[12pt]
\dot{x}_3 = x_4 \\[8pt]
\dot{x}_4 =
-
\frac{\left( b_\theta x_4 + mgh \sin x_3 \right)\left( M+m \right) + \left( u - b_x x_2 + mh \sin x_3 \, x_4^2 \right) mh \cos x_3}
{\left( M+m \right)\left( J+mh^2 \right) - \left( mh \cos x_3 \right)^2}
\end{cases}
$$

The output vector is defined as:

$$
\mathbf{y} =
\begin{bmatrix}
y_1 \\ y_2
\end{bmatrix}
=
\begin{bmatrix}
x_1 \\ x_3
\end{bmatrix}
$$

so that the complete system can be written as:

$$
\begin{cases}
\dot{\mathbf{x}} = \mathbf{f}(\mathbf{x},u) \\
\mathbf{y} = \mathbf{h}(\mathbf{x},u)
\end{cases}
$$

To simplify the controller design, the model was then **linearized around the equilibrium points**.  
The system admits two equilibrium configurations, corresponding to the pendulum in the downward and upward vertical positions:

$$
\begin{cases}
x_0 = x \\
\theta_0 = 0
\end{cases}
,\qquad
\begin{cases}
x_0 = x \\
\theta_0 = \pi
\end{cases}
\qquad \forall x \in \mathbb{R}
$$

Using the perturbation variables

$$
\tilde{x} = x - x_0,
\qquad
\tilde{\theta} = \theta - \theta_0
$$

the linearized equations can be expressed in compact form as:

$$
\begin{cases}
\left( M + m \right) \ddot{\tilde{x}} + z m h \ddot{\tilde{\theta}} + b_x \dot{\tilde{x}} = F \\[6pt]
z m h \ddot{\tilde{x}} + \left( J + m h^2 \right) \ddot{\tilde{\theta}} + b_\theta \dot{\tilde{\theta}} + z m g h \tilde{\theta} = 0
\end{cases}
$$

where

$$
z =
\begin{cases}
1 & \text{if } \theta_0 = 0 \\
-1 & \text{if } \theta_0 = \pi
\end{cases}
$$

The corresponding **linear state-space model** is:

$$
\begin{cases}
\dot{\mathbf{x}} = A \mathbf{x} + B \mathbf{u} \\
\mathbf{y} = C \mathbf{x} + D \mathbf{u}
\end{cases}
$$

with

$$
A =
\begin{bmatrix}
0 & 1 & 0 & 0 \\
0 & -\frac{b_x \left( J+mh^2 \right)}{p} & \frac{m^2gh^2}{p} & z\frac{mhb_\theta}{p} \\
0 & 0 & 0 & 1 \\
0 & z\frac{mhb_x}{p} & -z\frac{\left( M+m \right)gmh}{p} & -\frac{\left( M+m \right)b_\theta}{p}
\end{bmatrix}
$$

$$
B =
\begin{bmatrix}
0 \\
\frac{J+mh^2}{p} \\
0 \\
-z\frac{mh}{p}
\end{bmatrix},
\qquad
C =
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 0 & 1 & 0
\end{bmatrix},
\qquad
D =
\begin{bmatrix}
0 \\
0
\end{bmatrix}
$$

where

$$
p = J(M+m) + Mmh^2
$$


## Parameter Identification

Several physical parameters required by the model were not directly known and therefore had to be **experimentally identified**.  
These included the **pendulum mass**, the **position of its center of mass**, the **moment of inertia**, the **hinge rotational friction coefficient**, as well as the **actuation gain \(k_m\)** and **friction bias \(\beta\)** describing the relationship between the motor torque and the PWM control index.  
Additional parameters such as the **cart friction coefficient** and the **cart mass** were also estimated through dedicated experiments.

To identify these quantities, a series of **experimental tests and measurements** were carried out on the laboratory setup. These experiments included static equilibrium measurements, free oscillation analysis, interpolation of motor force data, and dynamic tests on the cart motion.

A detailed description of the identification procedures, experimental setups, and obtained results is provided in the **full project report**.


## Simulation Model

After deriving the analytical model and identifying the system parameters, a **simulation model of the cart–pendulum system was developed in Simulink**.  
This virtual representation of the system allows the dynamics to be analyzed and different control strategies to be tested in a controlled environment before implementing them on the real laboratory setup.

### Creation of the Virtual Model in Simulink

The nonlinear equations of motion derived from the analytical model were implemented in **Simulink**, reproducing the dynamic interaction between the cart and the pendulum.  
The resulting model accurately replicates the system behavior and provides a reliable environment for controller design and testing.

<p align="center"> <img src="/images/simulink_model.jpg" width="85%"> </p>

### Model Validation

To validate the accuracy of the simulation model, its response was compared with measurements obtained from the **real laboratory system**.  
Two experiments were performed: the **free decay of the pendulum** and the **cart response to a constant motor input**.  
The comparison shows a strong agreement between simulation and real measurements, confirming the reliability of the model and the identified parameters.

<div style="display: flex; gap: 20px; justify-content: center; align-items: flex-start;">

<div style="text-align: center;">
<img src="/images/free_decay_validation.png" width="450">
<figcaption>Free decay dynamics comparison.</figcaption>
</div>

<div style="text-align: center;">
<img src="/images/free_decay_validation_zoom.png" width="450">
<figcaption>Zoom of the free decay dynamics comparison.</figcaption>
</div>

</div>


<div style="display: flex; gap: 20px; justify-content: center; align-items: flex-start;">

<div style="text-align: center;">
<img src="/images/free_decay_validation_cart.png" width="450">
<figcaption>Cart response to a constant motor input comparison.</figcaption>
</div>

<div style="text-align: center;">
<img src="/images/constant_force_validation_pendulum.png" width="450">
<figcaption>Pendulum response to a constant motor input comparison.</figcaption>
</div>

</div>


## System Control Design

Once the simulation model was validated, different **control strategies** were designed to regulate the pendulum dynamics.  
Two approaches were explored: a **PID controller**, commonly used in industrial control systems, and a **Linear Quadratic Regulator (LQR)**, which allows a multivariable state-feedback control of the system.

### PID Control Implementation

The first control strategy implemented was a **Proportional–Integral–Derivative (PID) controller**.  
The controller was designed to stabilize the pendulum around the **upright unstable equilibrium** using the pendulum angle as the controlled variable.

The control architecture was implemented in **Simulink** as a feedback loop where the measured pendulum angle is used to compute the control force applied to the cart.

<p align="center"> <img src="/images/pid_system.png" width="85%"> </p>

The PID gains were tuned using MATLAB’s **`pidtune`** function, which automatically selects the parameters to balance settling time, overshoot, and robustness.  
After tuning the controller, the following behavior was obtained when stabilizing the pendulum in the upright position.

<p align="center"> <img src="/images/pid_upward_result.png" width="65%"> </p>

A comparison between simulation and laboratory results revealed an interesting discrepancy.  
While the simulation predicts a **linear motion of the cart**, the real system exhibits a **quadratic trajectory**, meaning the cart tends to accelerate during stabilization.  
This difference highlights a modeling limitation that becomes even more evident when implementing the LQR controller and is discussed further in the conclusions.

---

### Linear Quadratic Regulator (LQR) Control Implementation

To improve the control performance, a **Linear Quadratic Regulator (LQR)** was implemented using the linearized state-space model of the system.

The LQR controller computes an optimal feedback gain matrix by minimizing the quadratic cost function:

$$
J = \int_{0}^{\infty} \left( \mathbf{x}^T Q \mathbf{x} + u^T R u \right) dt
$$

where:

- \(Q\) penalizes deviations of the system states from the reference  
- \(R\) penalizes excessive control effort  

The control law is given by:

$$
u = -Kx
$$

where \(K\) is the optimal feedback gain matrix obtained by solving the Riccati equation.

The controller was implemented in **Simulink** using the state-space model of the system.

<p align="center"> <img src="/images/lqr_system.png" width="85%"> </p>

The weighting matrices \(Q\) and \(R\) were tuned through a **grid search procedure**, evaluating multiple combinations and selecting the configuration that provided the best compromise between **settling time and overshoot**.

The following plots show the resulting system behavior in simulation and in laboratory experiments.

<div style="display: flex; gap: 20px; justify-content: center; align-items: flex-start;">

<div style="text-align: center;">
<img src="/images/lqr_simulation_results.jpg" width="450">
<figcaption>LQR simulation results.</figcaption>
</div>

<div style="text-align: center;">
<img src="/images/lqr_lab_results.jpg" width="450">
<figcaption>LQR laboratory results.</figcaption>
</div>

</div>


## Filtering

As observed during the control experiments, the **sensor measuring the pendulum angle** is significantly affected by noise.  
Although the simulation model showed that the controllers could stabilize the system, real-world measurements introduced disturbances that degraded the control performance.

To address this issue, **signal filtering techniques** were implemented to improve the quality of the state measurements. Two approaches were investigated:
- a **Low-Pass Filter (LPF)** to attenuate high-frequency noise  
- a **Kalman Filter (KF)** for model-based state estimation

### Low Pass Filter

The Low-Pass Filter was introduced to remove high-frequency noise from the pendulum angle measurement.  
The cutoff frequency of the filter was determined by analyzing the **Power Spectral Density (PSD)** of the measured pendulum angle signal.

The PSD analysis allowed the identification of the frequency range associated with the system dynamics, separating it from the noise-dominated region.  
Based on this analysis, an appropriate cutoff frequency was selected to preserve the relevant dynamics while attenuating noise.

As expected, applying the filter introduces a **small delay** in the signal, which was estimated through cross-correlation analysis.

The following figure shows the effect of the filtering process on the pendulum angle measurement.

<p align="center"> <img src="/images/lpf_filtering_result.jpg" width="65%"> </p>

### Kalman Filter

To further improve the state estimation, a **Kalman Filter** was implemented.  
Unlike the Low-Pass Filter, the Kalman approach combines the **system model** with the **sensor measurements** to estimate the system states in an optimal way.

In this project an **Extended Kalman Filter (EKF)** formulation was adopted to account for the nonlinear dynamics of the system.  
The observer estimates the system states using two main steps:

- **Prediction**, where the system dynamics are propagated using the model  
- **Update**, where the estimate is corrected using the measurements

The estimated states were then used within the **LQR feedback control law**.

Simulation results showed that the Kalman filter significantly improves the estimation of the pendulum angle, reducing the effect of measurement noise and improving the stability of the controller.

Due to time constraints during the laboratory sessions, the Kalman filter implementation was **not tested on the real hardware**, and therefore the results presented are limited to simulation experiments.


## Swing-Up Control

To move the pendulum from the stable downward equilibrium to the unstable upright position, a **swing-up control strategy** was implemented.  
The approach adopted in this project is based on an **energy control method**, which gradually increases the mechanical energy of the pendulum until it reaches the level required for the upright configuration.

The total energy of the pendulum is expressed as:

$$
E_p = \frac{1}{2}(J + mh^2)\dot{\theta}^2 - mgh\cos(\theta)
$$

The control objective is to regulate the energy difference between the current pendulum energy and the target energy corresponding to the upright equilibrium.  
By appropriately commanding the cart acceleration, the controller injects energy into the system until the pendulum reaches the vicinity of the upright position.

Once the pendulum angle enters a small region around the upright configuration (approximately ±10°), the control is switched to the **stabilization controller (LQR)** to maintain the pendulum balanced.

The following figures show the swing-up behavior obtained in simulation and on the real laboratory system.

<div style="display: flex; gap: 20px; justify-content: center; align-items: flex-start;">

<div style="text-align: center;">
<img src="/images/swingup_simulation.jpg" width="450">
<figcaption>Swing up simulation.</figcaption>
</div>

<div style="text-align: center;">
<img src="/images/swingup_lab.jpg" width="450">
<figcaption>Swing up laboratory exeuction.</figcaption>
</div>

</div>



## Conclusions

This project covered the complete development pipeline for controlling a **linear inverted pendulum system**, from analytical modeling to experimental validation on a real laboratory setup.

The derived dynamic model and the experimentally identified parameters allowed the creation of a **reliable simulation environment**, which proved essential for the design and tuning of the control strategies. Both **PID and LQR controllers** were successfully implemented and tested, demonstrating effective stabilization of the pendulum in the downward configuration and partial success in the upright case.

During the experimental phase, several **practical challenges** emerged that were not fully captured by the theoretical model. In particular, measurement noise and a bias in the pendulum angle sensor significantly affected the controller performance. Filtering techniques such as a **Low-Pass Filter** and a **Kalman-based state estimator** were therefore investigated to improve state estimation and robustness.

Despite these limitations, the project successfully demonstrated the **swing-up maneuver using an energy-based control approach**, allowing the pendulum to transition from the downward equilibrium to the upright position before switching to a stabilization controller.

Overall, the work highlights the importance of combining **analytical modeling, simulation, and experimental validation** when designing control systems for nonlinear mechanical systems, as well as the challenges that arise when transferring theoretical solutions to real-world implementations.

