# GNSS Vulnerabilities, Modern Attacks, and Covert UAV Trajectory Spoofing

This repository hosts research and technical documentation analyzing the fundamental security vulnerabilities of Global Navigation Satellite Systems (GNSS), modern signal interference/spoofing vectors, integrated GNSS/INS navigation architectures, and covert trajectory control attacks targeting Unmanned Aerial Vehicles (UAVs).

---

## 📌 Executive Summary

Modern satellite navigation systems (GPS, GLONASS, Galileo, BeiDou) rely on extremely low-power signals (~ -160 dBW) received at the Earth's surface—well below the thermal noise floor (-144 dBW). This physical limitation makes GNSS highly susceptible to radio frequency (RF) interference, spoofing, and signal manipulation.

While integrating Inertial Navigation Systems (INS) via Error-State Kalman Filtering (ESKF) provides defense against simplistic jamming and conventional spoofing, sophisticated **covert spoofing** techniques can bypass sensor fusion checks. By mathematically inverting the ESKF observation update loop and employing real-time external radar tracking, an attacker can imperceptibly manipulate a UAV's physical flight trajectory while forcing onboard state estimators to report normal adherence to the original flight plan.

---

## 🎯 Key Topics & Technical Scope

### 1. GNSS Fundamentals & Signal Structure
* **3D Time-of-Arrival (TOA) Ranging:** Multilateration principles across Space, Control, and User segments.
* **Physical Layer Characteristics:** L-band carrier frequencies (1164–1300 MHz and 1559–1610 MHz), Binary Phase Shift Keying (BPSK) modulation, and Direct Sequence Spread Spectrum (DSSS) pseudo-random noise (PRN) codes.

### 2. Taxonomy of GNSS Attack Vectors
* **Jamming (Denial of Service):** Continuous Wave (CW), Chirp / FMCW, Band-Limited White Gaussian Noise, Multi-tone CW, and Pulsed Jamming (LNA saturation).
* **Meaconing:** Delayed retransmission of authentic signals with variable delay mechanisms to bypass clock-drift monitoring.
* **Spoofing Hierarchy:**
  * *Simplistic:* Unsynchronized, inconsistent, high-power signal injection.
  * *Intermediate:* Code/carrier phase-aligned signal takeover ("lift-off").
  * *Sophisticated:* Multi-antenna synthetic wavefront generation, Angle-of-Arrival (AoA) defense evasion.
* **Hybrid / Physical Layer Attacks:**
  * Relaying / Worm-hole attacks over high-speed digital radio links.
  * Security Code Estimation and Replay (SCER) utilizing high-gain directional antennas for chip-by-chip physical layer estimation.

### 3. Real-World Case Studies (2023–2025)
* **Civil Aviation (Iraq/Iran Air Corridors):** Inertial Reference System (IRS) mathematical poisoning causing primary instrument failures.
* **Urban Electronic Warfare (Tel Aviv):** Metropolitan-scale position spoofing redirecting receivers to false coordinates (Beirut Airport).
* **Cross-Border Interdiction (Kaliningrad / Baltic):** Widespread regional jamming leading to commercial flight suspensions (Tartu Airport, Estonia).
* **Maritime Auto-Spoofing (Red Sea / Strait of Hormuz):** SDR-driven AIS manipulation generating ghost vessel trajectories.

### 4. Integrated Navigation Systems (INS/GNSS & ESKF)
* **INS Error Dynamics:** Uncalibrated accelerometer ($O(\Delta t^2)$) and gyroscope ($O(\Delta t^3)$) error propagation models.
* **Architectural Evolution:** Loosely coupled vs. Tightly coupled vs. Ultra-tightly coupled sensor fusion.
* **Error-State Kalman Filter (ESKF):** 
  * Separation of True, Nominal, and Error states.
  * 3-phase cycle: High-frequency Prediction, Low-frequency Correction (Innovation residual calculation), and Error Injection/Reset.
  * Innovation vector ($\tilde{y}_k$) monitoring as a countermeasure against signal anomalies.

### 5. Covert Trajectory Spoofing on UAVs
Analysis of mathematical deception against loosely coupled UAV architectures (9-DOF error state: $\delta X = [\delta p, \delta v, \delta a]^T$):
1. **Radar State Acquisition:** External tracking of real kinematics ($\hat{X}_k^R$).
2. **Acceleration Control Generation:** Computing virtual control vector $a^* = a_k - a_k^s$ using proportional-derivative tracking matrices ($K_{PD}$).
3. **Apparent State Synthesis:** Constructing fake position/velocity estimates within filter tolerance margins.
4. **Kalman Gain Inversion:** Reversing steady-state Kalman Gain ($K^s = \lim_{n\to\infty} K_k$) to generate synthetic GNSS coordinates ($X_{k,gnss}^s$):
   $$X_{k,gnss}^s = \hat{X}_{k|k-1,ins}^s - (K^s)^{-1} \left( \hat{X}\_{k|k-1,ins}^s - \hat{X}_k^s \right)$$
5. **Physical Hardware Constraints:** Strict synchronization requirements ($\le 0.1$ chip code phase difference, $\le 10\text{ Hz}$ carrier Doppler offset; Radar accuracy thresholds: $\delta p < 30\text{ m}$, $\delta v < 1\text{ m/s}$).

---

## 🛡️ Countermeasures & Mitigations

* **Statistical Innovation Analysis:** Detection of subtle, persistent biases in the zero-mean innovation sequence ($\tilde{y}_k$).
* **Cumulative Sum (CUSUM) Test:** Sequential integration of continuous innovation residuals to expose monotonic directional drifts induced by covert spoofing before kinematic deviation becomes irreversible.
* **Tightly & Ultra-Tightly Coupled Fusion:** Direct raw pseudorange/Doppler filtering and joint receiver tracking loop pre-compensation.
