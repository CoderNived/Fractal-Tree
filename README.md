# ConceptLab: Interactive Visualization & Simulation Engine (Raylib + C++)

## Overview

**ConceptLab** is a modular, extensible, and performance-aware desktop simulation engine built with **C++20** and **Raylib**.

It is designed to visualize and simulate technical concepts commonly found in academic, systems, and engineering documentation, including:

- Algorithms and data structures
- Signal processing (FFT, sampling theory, aliasing)
- Circuit dynamics (RC and RLC transient behavior)
- Graph traversal and system flows
- Physics-oriented numerical simulations

ConceptLab is intentionally architected as a **production-grade simulation engine**, not a toy visualizer. The core philosophy emphasizes:

- Strict separation between simulation and rendering
- Deterministic memory behavior
- Modular architecture
- Extensibility and testability
- Numerical and performance rigor

---

## Why This Project Exists

ConceptLab was designed as an engineering artifact to:

- Bridge abstract theory and visual intuition
- Demonstrate strong C++ systems architecture practices
- Showcase DSP and numerical methods implementation ability
- Illustrate real-time graphics integration in a maintainable codebase
- Serve as a portfolio-grade project suitable for technical review

This project is particularly suitable for:

- Internship applications
- Research portfolios
- GSoC proposals
- Systems / Graphics / DSP-focused roles

---

## Core Capabilities

### 1) Algorithm Visualizations

ConceptLab includes algorithm modules with step-oriented visualization and execution controls:

- QuickSort
- MergeSort
- HeapSort
- Breadth-First Search (BFS)
- Depth-First Search (DFS)
- Stack and Queue simulation modules

Execution UX features:

- Step-by-step progression
- Replay mode
- Adjustable simulation speed
- Deterministic state progression for reproducibility

---

### 2) Signal & Systems Visualization

Signal modules support both educational and engineering-oriented DSP workflows:

- Time-domain signal rendering
- Real-time parameter manipulation (amplitude, frequency, phase, sampling rate)
- Sampling theorem demonstrations
- Aliasing visualization
- Windowing support:
  - Rectangular
  - Hamming
  - Hanning
- Zero-padding support
- In-house iterative **Radix-2 Cooley–Tukey FFT**
- Frequency spectrum rendering with optional log-scale visualization

---

### 3) Circuit Simulation

Circuit modules model continuous-time dynamics using numerical integration:

- RC transient response simulation
- RLC simulation (numerical)
- Explicit Euler integrator pipeline
- Voltage-time graphing and response inspection

The simulation path is designed to keep numerical behavior observable and configurable for educational and analysis use cases.

---

## Engineering Architecture

ConceptLab follows a layered design with explicit responsibility boundaries:

```text
Application Layer
        ↓
State Manager
        ↓
Simulation Engine
        ↓
Renderer
```

### Architectural Principles

- **Simulation and rendering are fully decoupled.**
  - The simulation layer produces immutable or snapshot-style state.
  - The renderer consumes state and issues draw calls.

- **No simulation logic in the rendering layer.**
  - Rendering remains stateless and presentation-focused.

- **Deterministic update behavior.**
  - Core simulation steps avoid hidden side effects.
  - FFT and DSP operations are deterministic for identical inputs.

- **Memory discipline.**
  - No per-frame dynamic vector growth inside update loops.
  - Buffers are preallocated where practical.

- **Testable core.**
  - Mathematical and DSP logic are designed to run independently of graphical dependencies.

### System Patterns Used

- ECS-inspired modular composition
- State-machine-driven application flow
- Observer-based event dispatch
- Config-driven simulation presets

---

## Repository Structure

```text
ConceptLab/
├── CMakeLists.txt
├── README.md
│
├── external/
│   ├── raylib/
│   ├── raygui/
│   └── nlohmann_json/
│
├── assets/
│   ├── fonts/
│   ├── shaders/
│   └── icons/
│
├── config/
│   ├── app_config.json
│   └── signal_presets.json
│
├── include/
│   ├── core/
│   ├── engine/
│   ├── math/
│   ├── dsp/
│   ├── simulation/
│   ├── modules/
│   └── ui/
│
├── src/
│   ├── core/
│   ├── engine/
│   ├── math/
│   ├── dsp/
│   ├── simulation/
│   ├── modules/
│   ├── ui/
│   └── main.cpp
│
├── tests/
│   ├── test_fft.cpp
│   ├── test_signal.cpp
│   └── test_circuit.cpp
│
└── docs/
    ├── architecture.md
    ├── fft_design.md
    └── simulation_design.md
```

---

## DSP + FFT Module (Deep Dive)

### Mathematical Foundation

The FFT pipeline is built around the iterative **Radix-2 Cooley–Tukey** algorithm with:

- Iterative in-place transform stages
- Bit-reversal permutation
- Precomputed twiddle factors
- Magnitude spectrum extraction

Continuous-time signal model:

\[
x(t) = A\sin(2\pi f t + \phi)
\]

Discrete sampled form:

\[
x[n] = A\sin\left(\frac{2\pi f n}{F_s} + \phi\right)
\]

### FFT Strategy

Supported processing behavior:

- Power-of-two sample sizes
- Optional zero-padding for improved bin interpolation display
- Window application prior to transform
- Optional log-scale spectrum interpretation

### Processing Pipeline

```text
Time-Domain Signal
        ↓
Windowing
        ↓
Zero Padding (optional)
        ↓
FFT (in-place)
        ↓
Magnitude Computation
        ↓
Renderer
```

### Windowing

Supported window families:

- Rectangular
- Hamming
- Hanning

Example Hamming window equation:

\[
w[n] = 0.54 - 0.46\cos\left(\frac{2\pi n}{N-1}\right)
\]

### Complexity & Performance Profile

| Operation | Complexity |
|---|---|
| Signal generation | O(N) |
| Windowing | O(N) |
| FFT | O(N log N) |
| Spectrum magnitude | O(N) |

Performance assumptions and guarantees:

- Memory buffers are preallocated
- No per-frame growth in hot update loops
- Deterministic execution path for reproducible signal analysis

---

## Circuit Simulation Model

For RC circuit dynamics, the module models:

\[
\frac{dV_c}{dt} = \frac{1}{RC}(V_{in} - V_c)
\]

Numerical integration uses explicit Euler:

```cpp
Vc += dt * (1.0 / (R * C)) * (Vin - Vc);
```

Stability and timestep sensitivity are intended to be documented and tunable, with additional notes placed in:

- `/docs/simulation_design.md`

---

## Rendering System Design

Rendering is intentionally stateless and presentation-only.

The simulation layer publishes state objects conceptually similar to:

```cpp
struct StateObject {
    std::vector<float> waveform;
    std::vector<float> spectrum;
};
```

The renderer consumes these snapshots and emits draw calls. This avoids tight coupling and enables independent testing of simulation logic.

---

## Build Instructions

### Requirements

- CMake >= 3.20
- C++20-compatible compiler
- Raylib (installed globally or bundled in `external/`)

### Build

```bash
mkdir build
cd build
cmake ..
cmake --build .
```

### Run

```bash
./ConceptLab
```

---

## Testing

Unit testing is expected to be implemented with GoogleTest.

Coverage targets include:

- FFT correctness and bin behavior
- Signal generation consistency
- Circuit ODE convergence behavior
- Window function correctness

Run tests with:

```bash
ctest
```

---

## Performance Notes

Current optimization themes:

- Iterative in-place FFT
- Bit-reversal table precomputation
- Twiddle-factor caching
- No dynamic allocation in update loops
- Profiling hook compatibility (e.g., Tracy)

Potential future optimization tracks:

- SIMD acceleration
- Multi-threaded FFT decomposition
- GPU shader-based FFT path

---

## Advanced Extension Roadmap

Planned or possible extensions:

- Real-time microphone input FFT
- Spectrogram / waterfall visualization
- GPU-accelerated frequency analysis
- Plugin-based simulation loading
- Lua scripting support
- MP4 export pipeline
- Dear ImGui integration layer

---

## Engineering Highlights

ConceptLab demonstrates applied competency in:

- C++ systems architecture
- Numerical stability and integration reasoning
- End-to-end DSP implementation
- Real-time rendering pipeline design
- Memory-aware simulation loop construction
- Modular extensibility for long-term growth

---

## Development Roadmap

### Phase 1

- Core engine
- Algorithm visualization foundation

### Phase 2

- Signal processing modules
- Sampling theorem and aliasing demonstrations

### Phase 3

- Circuit and physics simulation modules

### Phase 4

- Plugin architecture
- Scripting and advanced interoperability

---

## Benchmarks (Example Targets)

Example profile for a 1024-point FFT target:

- Average execution time: ~0.3 ms
- Frame rate target: stable 60 FPS
- Memory footprint target: < 20 MB

> Note: Actual runtime values depend on hardware, compiler settings, and build configuration.

---

## Screenshots

Add screenshots here as modules are finalized:

- Waveform view
- Spectrum view
- Sorting animation view

---

## License

This project is licensed under the **MIT License**.

---

## Author Intent

ConceptLab is developed as an advanced systems-level C++ visualization and simulation platform intended to demonstrate practical engineering capability across:

- DSP
- Numerical simulation
- Real-time graphics
- Modular software architecture

If you're evaluating this repository for portfolio, internship, or collaboration purposes, this README is intended to provide both technical depth and architectural intent at a glance.
