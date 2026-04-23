# Ring Attractor Network

> A biologically plausible spiking neural network model of working memory and head-direction encoding — built from scratch during and after [Neuromatch Academy 2020](https://neuromatch.io/).

**Authors:** Stefano Bettani, Nikitas Kalaitzidis, Pranjal Gupta, Angela (Orion) — mentored by [Mehrdad Jazayeri](https://jazlab.org/) (MIT), TA: Peter Vincent.

---

## Overview

Ring attractor networks are a canonical model in computational neuroscience for explaining how the brain maintains continuous variables — like **head direction**, **working memory**, or **visual orientation** — through persistent, localised neural activity ("bumps"). This project implements a full spiking ring attractor from the ground up, characterises its stability properties, and studies how fixed points and noise interact to produce robust, biologically realistic dynamics.

The codebase began in **Python** and was later ported to **Julia** for a ~10× speedup in parameter sweeps.

---

## Biological Motivation

The Drosophila melanogaster ellipsoid body contains a ring of E-PG neurons whose activity encodes the fly's current head direction as a localised bump of firing. A bump that persists without continuous input is a form of **attractor dynamics** — the network "remembers" a direction through its own recurrent connectivity.

This same architecture generalises to other cyclic quantities (colour, spatial phase, orientation) and is thought to underlie aspects of **spatial working memory** in mammals.

---

## Model Architecture

| Component | Specification |
|-----------|--------------|
| Neuron model | Integrate-and-fire (IAF) with conductance-based synapses |
| Network size | 128 neurons arranged in a ring |
| Connectivity | 4-4 topology (each neuron connects to 4 excitatory + 4 inhibitory neighbours) |
| Fixed points | Groups of 3 adjacent neurons with reinforced E/I weights |
| Noise | Additive Gaussian noise at every voltage update |
| Language | Python (prototyping) → Julia (large-scale simulation) |

### Connectivity kernel

The recurrent weight matrix uses a **Mexican-hat** profile: short-range excitation flanked by long-range inhibition. This is the minimal structure needed to support a stable, spatially localised bump.

```
W(Δθ) = J_E · exp(−Δθ²/2σ_E²)  −  J_I · exp(−Δθ²/2σ_I²)
```

---

## Key Results

### 1. Bump formation and persistence

After a transient input, the network settles into a stable bump of activity that persists indefinitely — confirming the attractor property.

![Bump activity raster](https://github.com/stebett/ring-attractor/blob/master/plots/github/fp_crit1.png)

### 2. Noise-induced drift and fixed-point stabilisation

Without fixed points, Gaussian noise causes the bump to perform a **random walk** around the ring. Strategic placement of fixed points — groups of neurons with slightly strengthened weights — anchors the bump and reduces mean drift error.

![Fixed point stabilisation](https://github.com/stebett/ring-attractor/blob/master/plots/github/fixed_point.png)

### 3. Parameter landscape

Systematic sweeps over `(J_E, J_I)` space reveal three qualitative regimes:
- **Silent** — no persistent activity
- **Bump** — stable localised activity ✓
- **Global** — all neurons fire (runaway excitation)

![Parameter sweep](https://github.com/stebett/ring-attractor/blob/master/plots/github/spike_number_by_weight_seed%3D2020.png)

### 4. Error metric

We tracked bump position error as the mean absolute deviation of the median active neuron over the last 100 timesteps. This metric guided the optimisation of fixed-point placement and noise tolerance.

---

## Repository Structure

```
ring-attractor/
├── src/               # Core Julia simulation code
│   ├── network.jl     # IAF neuron + connectivity
│   ├── simulate.jl    # Main simulation loop
│   └── analysis.jl    # Bump tracking & error metrics
├── scripts/           # Parameter sweep runners
├── notebooks/         # Python prototyping notebooks
├── data/              # Simulation outputs (.jld2)
├── plots/             # Generated figures
├── docs/              # Extended methods & derivations
└── _research/         # Literature notes & references
```

---


## What We Built

- **IAF model** with conductance-based synapses (biophysically grounded)  
- **128-neuron ring** with 4-4 connectivity — sufficient to support a stable bump  
- **Noise characterisation** — established thresholds at which noise ignites spurious bumps  
- **Fixed-point mechanism** — groups of 3 adjacent neurons with up-regulated weights anchor bump position  
- **Weight-scaling equation** — automatically adjusts weights as a function of the number of fixed points to preserve network dynamics  
- **Error metric** — mean of median active neuron position over last 100 timesteps; used to rank fixed-point configurations  
- **Julia port** — full rewrite for ~10× faster simulation, enabling larger parameter sweeps  
- **Multi-sweep study** — independently swept weights for bump stability and fixed-point stability

---

## References

- Dayan P. & Abbott L.F. — *Theoretical Neuroscience* (MIT Press, 2001)  
- Vergani A.A. & Huyck C.R. — *Critical Limits in a Bump Attractor Network of Spiking Neurons*  
- Kilpatrick Z.P. & Ermentrout B. — *Wandering Bumps in Stochastic Neural Fields*  
- Veltz R. & Faugeras O. — *Local/Global Analysis of the Stationary Solutions of some Neural Field Equations*

---

## Future Work

- [ ] Hebbian / spike-timing-dependent plasticity (STDP) learning rules  
- [ ] Continuous attractor dynamics for path integration  
- [ ] Extension to 2D attractor maps (grid cells)

---

*Built at [Neuromatch Academy 2020](https://neuromatch.io/) — a global computational neuroscience summer school.*
