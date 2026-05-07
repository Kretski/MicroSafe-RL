# MicroSafe-RL: Runtime Safety Layer for AI Control

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Language: C++11](https://img.shields.io/badge/Language-C++11-blue.svg)](#)
[![Compliance: MISRA:2012](https://img.shields.io/badge/Compliance-MISRA:2012-green.svg)](#)
[![RAM: 24 bytes](https://img.shields.io/badge/RAM-24_bytes-orange.svg)](#)
[![Latency: <1.2µs](https://img.shields.io/badge/Latency-%3C1.2µs-red.svg)](#)

**MicroSafe-RL** is an ultra-lightweight, deterministic safety interceptor designed for RL agents and LLM-driven control systems deployed on embedded hardware. 

It acts as a "hard shield" between your AI policy and your physical actuators, ensuring that every command is validated and corrected in real-time before it can cause hardware damage.

---

## ⚡ Why MicroSafe-RL?

Reinforcement Learning policies and LLMs are "black boxes". When deployed on real hardware (robotics, edge devices), they can produce out-of-range or erratic commands. 

**The Problem:** Traditional safety checks are often too slow or consume too much memory for MCU deployment.
**The Solution:** MicroSafe-RL provides a mathematically sound interceptor with a near-zero footprint.

- **Minimalist:** Only **24 bytes of RAM**, zero dynamic allocation.
- **Ultra-fast:** **< 1.2 µs** execution time (tested on Cortex-M3 @ 72MHz).
- **Industrial Grade:** MISRA-C:2012 compliant for safety-critical applications.
- **Explainable:** Provides a quantifiable `safety_score` for every single control cycle.

---

## ⚙️ How It Works

MicroSafe-RL sits directly in your control loop:
`AI Policy (LLM/RL) → MicroSafe-RL → Actuator`

It uses a **Gravity-based Clipping** mechanism. Instead of a hard binary cut, it scales unsafe actions back toward the safe zone proportionally, based on signal coherence and velocity.

```cpp
// 1. AI proposes an action
float raw_action = ai_model.predict(state);

// 2. MicroSafe-RL validates and corrects it
float safe_action = safety.apply_safe_control(raw_action, sensor_feedback);

// 3. Execution with feedback
actuator.write(safe_action);
float penalty = safety.get_current_penalty(); // Use this to retrain your AI!

Benchmark ResultsTested against standard Kalman-filter detectors across 120 adversarial scenarios (runaway conditions, sensor noise, adversarial LLM outputs).MetricMicroSafe-RLKalman FilterPLC ThresholdDetection Margin19.2 steps11.0 steps8.0 stepsRAM Usage24 bytes~1.2 KB~128 bytesWorst-Case Latency1.18 µs45.20 µs3.10 µs🔧 Installation & Quick StartFor Arduino/STM32:Download MicroSafeRL.h and include it in your project.Initialize with your hardware constraints:C++#include "MicroSafeRL.h"

// MicroSafeRL(kappa, alpha, decay, min_limit, max_limit)
MicroSafeRL safety(0.078f, 0.55f, 2.2f, -1.5f, 1.5f);

void loop() {
    float safe_val = safety.apply_safe_control(ai_output, mpu_data);
    motor.set(safe_val);
}
For Python (Gymnasium):Pythonfrom wrappers.microsafe_gym import MicroSafeWrapper
env = MicroSafeWrapper(gym.make("Pendulum-v1"))
📁 Project Structure/include - Core C++ implementation (header-only)./python - Gymnasium wrappers and training scripts./tools - microsafe_profiler.py for automatic parameter tuning./examples - Ready-to-run demos for Arduino and ROS2.📄 Academic Work & LicenseThis project is based on the research:Kretski, D. — "Optimal and Stable FDIR Architecture for Autonomous Spacecraft and Critical Systems" — Zenodo, 2026.License: MIT for non-commercial/academic use. For production deployment in safety-critical systems, contact kretski1@gmail.com.