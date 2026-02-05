# Kepler-1047 Orbital Stability Analysis

## Project Overview

This project is a numerical investigation into the dynamical stability of the Kepler-1047 planetary system. The objective was to determine if stable asteroid orbits could exist within the circumplanetary environment given the gravitational influence of the two massive inner gas giants.

The analysis utilizes a custom N-body integrator to run a Monte Carlo simulation across a parameter space ranging from 0.1 to 9.0 AU. The results indicate that the system is dynamically hostile; the inner planets act as gravitational sweepers, destabilizing minor bodies through secular perturbations even at significant distances.

## Astrophysical Context

Kepler-1047 is a circumbinary planetary system characterized by two massive gas giants in close proximity to the host star:

- **Planet B**: 1.43 M_J at 0.23 AU (orbital period ≈ 49 days)
- **Planet C**: 0.77 M_J at 0.42 AU (orbital period ≈ 93 days)
- **Host Star**: Solar-type main sequence star (M ≈ 1.0 M_☉)

The large planetary masses and tight orbital configuration create a gravitationally turbulent environment. This analysis investigates whether small bodies (asteroids, comets, or trojans) could maintain stable orbits on astronomical timescales (>10⁴ years) within this system.

## Mathematical Framework

### N-Body Gravitational Dynamics

The system evolves according to Newton's law of universal gravitation. For N gravitating bodies with positions $\mathbf{r}_i$ and masses $m_i$, the equations of motion are:

$$\ddot{\mathbf{r}}_i = -G \sum_{j \neq i} \frac{m_j (\mathbf{r}_i - \mathbf{r}_j)}{|\mathbf{r}_i - \mathbf{r}_j|^3}$$

where $G = 6.674 \times 10^{-11} \, \text{m}^3 \text{kg}^{-1} \text{s}^{-2}$ is the gravitational constant.

This forms a system of coupled second-order ordinary differential equations. For three bodies (star, Planet B, Planet C, and test asteroid), this expands to 18 coupled ODEs (6 per body: 3 position + 3 velocity components).

### Initial Conditions: Keplerian Orbits

Test particles are initialized on circular orbits satisfying Kepler's third law. For a body at radius $r$ orbiting a central mass $M_\star$, the circular orbital velocity is:

$$v_{\text{circ}} = \sqrt{\frac{GM_\star}{r}}$$

The angular frequency corresponding to this motion is:

$$\omega = \sqrt{\frac{GM_\star}{r^3}}$$

For the Monte Carlo sweep, initial semi-major axes $a$ are uniformly sampled from [0.1, 9.0] AU, with the remaining orbital elements set to:
- Eccentricity: $e = 0$ (circular)
- Inclination: $i = 0°$ (coplanar)
- Argument of periapsis: $\omega = 0°$
- Longitude of ascending node: $\Omega = 0°$
- Mean anomaly: $M_0$ uniformly distributed in $[0, 2\pi)$

### Hamiltonian Formulation and Energy Conservation

The total energy of the N-body system is given by the Hamiltonian:

$$H = \sum_{i=1}^{N} \frac{1}{2} m_i |\dot{\mathbf{r}}_i|^2 - G \sum_{i=1}^{N} \sum_{j>i}^{N} \frac{m_i m_j}{|\mathbf{r}_i - \mathbf{r}_j|}$$

where the first term is kinetic energy and the second is gravitational potential energy. In the absence of external forces, $H$ is a conserved quantity:

$$\frac{dH}{dt} = 0$$

Monitoring the relative energy error:

$$\epsilon_E = \frac{|H(t) - H(0)|}{|H(0)|}$$

provides a diagnostic for numerical accuracy. Our simulation maintains $\epsilon_E \approx 10^{-7}$, confirming that observed instabilities are physical rather than artifacts of integration error.

### Stability Criterion: Radial Drift

A test particle is classified as **unstable** if its orbital radius deviates by more than 10% from its initial value over the integration period:

$$\text{Unstable if:} \quad \frac{|r(t_{\text{final}}) - r(0)|}{r(0)} > 0.1$$

This criterion captures both ejections (large positive drift) and planetary collisions (large negative drift), while allowing for small-amplitude epicyclic oscillations characteristic of perturbed Keplerian motion.

### Perturbation Theory Context

In the absence of Planets B and C, test particles would maintain perfect circular orbits. The planets introduce **secular perturbations**—slow, cumulative changes to orbital elements that arise from orbit-averaging the time-dependent gravitational potential.

For small perturbations, Lagrange's planetary equations predict that eccentricity $e$ evolves as:

$$\frac{de}{dt} = \frac{\sqrt{1-e^2}}{na^2e} \frac{\partial \mathcal{R}}{\partial \omega}$$

where $n$ is the mean motion and $\mathcal{R}$ is the **disturbing function** (the perturbing potential). Even for initially circular orbits, nearby massive planets can excite non-zero eccentricity, leading to eventual instability.

Our simulation resolves these dynamics numerically without perturbative approximations, capturing both secular and short-period effects.

## Technical Implementation

### Vectorized Physics Engine

The gravitational acceleration logic is implemented using vectorized NumPy broadcasting. By calculating all O(N²) interactions through linear algebra rather than iterative loops, the simulation efficiently handles the high-throughput requirements of a 10,000-sample Monte Carlo run.

**Algorithmic Complexity:**
- Naive loop-based approach: O(N² × N_steps × N_samples) ≈ 10¹² operations
- Vectorized approach: ~100× speedup via SIMD instructions

### Integration and Precision

Trajectories are integrated using an **adaptive RK45 (Runge-Kutta)** method from `scipy.integrate.solve_ivp`. This fifth-order method with fourth-order error estimation automatically adjusts the timestep to maintain accuracy while maximizing efficiency.

**Numerical Parameters:**
- **Integration timespan**: 1,000 years ≈ 3.65 × 10⁵ days
- **Relative tolerance**: rtol = 1×10⁻¹⁰
- **Absolute tolerance**: atol = 1×10⁻¹²
- **Average timestep**: ~0.1 days (adaptive)
- **Energy conservation**: $\epsilon_E \approx 10^{-7}$

To ensure results represent physical reality rather than numerical artifacts, the solver was configured with tight tolerances (rtol=1e-10, atol=1e-12). Hamiltonian conservation was monitored throughout, maintaining a relative energy error of approximately 1e-7.

### Validation: The Null-Planet Test

A key part of the workflow involved distinguishing between numerical integration drift and physical dynamical chaos. A control simulation (Null-Planet Test) was conducted by setting the masses of Planets B and C to zero. 

**Null-Planet Results:**
- Mean radial drift: $< 10^{-8}$ (purely numerical noise)
- Energy error: $\epsilon_E < 10^{-9}$
- Conclusion: Observed 10% drift in full simulation is **physical**, not numerical

The resulting drift was negligible (<1e-8), confirming that the 10% radial drift observed in the main simulation is a consequence of planetary perturbations and not a failure of the integrator.

This validates that the instabilities observed in the main simulation are **genuine dynamical chaos**, arising from the nonlinear gravitational interactions between the test particle and the massive planets.

## Physical Interpretation of Results

The stability map reveals several key features:

1. **Inner Clearing Zone** (r < 0.5 AU): Complete instability due to strong tidal forces and mean-motion resonances with Planet B.

2. **Resonance Gaps** (r ≈ 0.6 AU, 0.9 AU): Narrow bands of instability corresponding to low-order mean-motion resonances (3:1, 2:1) with Planets B and C.

3. **Chaotic Sea** (0.5 < r < 3.0 AU): Widespread instability dominated by overlapping secular resonances. The Hill sphere of Planet C extends to ~1.2 AU, within which test particles are immediately destabilized.

4. **Outer Stability Window** (r > 5.0 AU): Weak stability emerges as perturbations decay with distance. However, even at 9 AU, ~20% of test particles remain unstable over 1,000 years due to long-period secular effects.

The lack of any substantial stable zones suggests that **Kepler-1047 is unlikely to host a significant asteroid population** within 3 AU of the host star.

## Data Visualization

### Stability Mapping
![Stability Map](plots/stability_map.png)
*Figure 1: Binary stability classification across the Monte Carlo parameter space. Red indicates unstable orbits (>10% radial drift), blue indicates stable orbits. Note the complex resonance structure and complete clearing within ~1.5 AU.*

### Numerical Validation
![Energy Conservation](plots/energy_conservation.png)
*Figure 2: Hamiltonian conservation test showing relative energy error remains below 10⁻⁷ throughout integration. This confirms that observed instabilities are physical, not numerical artifacts.*

### Orbital Trajectories
![Orbit Animation](animations/stable_orbit.gif)
*Figure 3: Sample trajectory animation showing test particle evolution in the rotating frame. Color indicates time progression. Stable orbits maintain quasi-circular geometry, while unstable orbits exhibit growing eccentricity before ejection or collision.*

## Computational Performance

- **Sample count**: 10,000 initial conditions
- **Runtime**: ~6 hours on 12-core workstation (Intel i7-12700K)
- **Memory footprint**: ~2.3 GB peak (trajectory storage)
- **Integration steps per sample**: ~3.5 × 10⁶ (adaptive, varies with dynamics)

## Future Work

### Chaos Indicators

Current stability classification uses a simple radial drift threshold. More sophisticated approaches include:

1. **Maximum Lyapunov Exponent (MLE)**:
   $$\lambda = \lim_{t \to \infty} \frac{1}{t} \ln \frac{|\delta(t)|}{|\delta_0|}$$
   where $\delta(t)$ is the separation of nearby trajectories. Positive $\lambda$ indicates exponential divergence (chaos).

2. **MEGNO (Mean Exponential Growth factor of Nearby Orbits)**:
   A fast chaos indicator that distinguishes regular from chaotic motion on shorter timescales.

3. **Frequency Analysis**: Fourier decomposition of orbital elements to identify resonant interactions.

### Extended Parameter Space

- **Non-zero eccentricity**: Test stability of eccentric initial orbits
- **Inclined orbits**: Investigate 3D dynamics and Kozai-Lidov resonances
- **Planetary migration**: Simulate inward migration of Planets B and C
- **Extended integration**: 10⁶ year simulations to capture long-period instabilities


---

**Author**: Finn Ditum  
**Contact**: [ap25019@qmul.ac.uk]  
**License**: MIT

---




