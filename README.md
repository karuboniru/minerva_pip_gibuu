# MINERvA Charged Pion Production — GiBUU Comparison

This project compares GiBUU (Giessen Boltzmann-Uehling-Uhlenbeck) neutrino-nucleus
simulation results with the MINERvA measurement of charged pion production in
$\nu_\mu$ CC interactions on hydrocarbon (CH) at $\langle E_\nu \rangle = 4.0$ GeV.

**Reference:** [arXiv:1406.6415](https://arxiv.org/abs/1406.6415) — B. Eberly et al. (MINERvA Collaboration), Phys. Rev. D **92**, 092008 (2015)

## Signal Definition

| Criterion | Value |
|---|---|
| Interaction | CC $\nu_\mu$ ($\nu_\mu + N \to \mu^- + X$) |
| Hadronic invariant mass | $W < 1.4$ GeV (isolates single-pion / $\Delta$(1232) resonance production) |
| Final-state pions | Exactly **one** charged pion ($\pi^+$ or $\pi^-$) |
| Neutrino energy | $1.5 < E_\nu < 10$ GeV (applied via GiBUU `nl_fluxcuts`) |

## Observables

Two flux-integrated differential cross sections are compared with MINERvA data:

- **$d\sigma/dT_\pi$** — pion kinetic energy distribution (7 bins, 35–350 MeV)
- **$d\sigma/d\theta_\pi$** — pion angle distribution w.r.t. beam direction (14 bins, 0–180°)

Statistical comparisons use the **full covariance matrix** from the MINERvA measurement
to compute $\chi^2$/NDF values.

## Target Composition

MINERvA uses a polystyrene (CH) scintillator target. GiBUU simulations are run
separately for **pure carbon** ($^{12}$C) and **pure hydrogen** ($^1$H) targets,
then combined to the CH cross section per nucleon:

$$\sigma_{\rm CH} = \frac{12\,\sigma_{\rm C} + 1\,\sigma_{\rm H}}{13}$$

where $\sigma_{\rm C}$ and $\sigma_{\rm H}$ are per-nucleon cross sections on each
target. The factor 12/13 comes from the carbon fraction in CH (8 carbon atoms with
12 nucleons each, plus 8 hydrogen atoms with 1 nucleon each).

## Analysis Workflow

### 1. Generate GiBUU Jobcards

```bash
./build_jobcard \
  --experiment minerva \
  --path-to-input /path/to/buuinput2025 \
  --output-file jobcard_C.job \
  --T 1 --2pibg false \
  --mediumSwitch false --flagInMedium false \
  --enu-min 1.5 --enu-max 10.0

./build_jobcard \
  --experiment minerva_H \
  --path-to-input /path/to/buuinput2025 \
  --output-file jobcard_H.job \
  --T 1 --2pibg false \
  --mediumSwitch false --flagInMedium false \
  --enu-min 1.5 --enu-max 10.0
```

### 2. Run GiBUU (C + H targets, N runs each)

```bash
for i in $(seq -w 1 $N_RUNS); do
  mkdir -p runs/C/run_$i
  (cd runs/C/run_$i && GiBUU.x < ../../jobcards/jobcard_C.job) &
done

for i in $(seq -w 1 $N_RUNS); do
  mkdir -p runs/H/run_$i
  (cd runs/H/run_$i && GiBUU.x < ../../jobcards/jobcard_H.job) &
done

wait
```

### 3. Convert FinalEvents.dat to ROOT

```bash
for i in $(seq -w 1 $N_RUNS); do
  gibuu2root runs/C/run_$i/FinalEvents.dat runs/C/run_$i/C_run_${i}.root &
  gibuu2root runs/H/run_$i/FinalEvents.dat runs/H/run_$i/H_run_${i}.root &
done
wait
```

### 4. Run the Analysis

```bash
./MINERvA_pi_single_charged_gibuu \
  --input-carbon runs/C/run_*/C_run_*.root \
  --input-hydrogen runs/H/run_*/H_run_*.root \
  --output result.root
```

Produces:
- `result.root` — histograms for both observables (data + GiBUU)
- `Tk_pi_charged.pdf` — $d\sigma/dT_\pi$ comparison plot
- `theta_pi_charged.pdf` — $d\sigma/d\theta_\pi$ comparison plot

## Build

```bash
cmake -B build -S .
cmake --build build -j$(nproc)
```

**Requirements:**
- C++23 compiler (GCC 14+ or Clang 18+)
- ROOT 6.30+ (RIO, Tree, Physics, RDataFrame components)
- Boost (program_options)

## Project Structure

```
minerva_pip_gibuu/
  CMakeLists.txt                          # Build configuration
  MINERvA_pi_single_charged_gibuu.cpp     # Main analysis executable
  build_jobcard.cpp                       # GiBUU jobcard generator
  gibuu2root.cpp                          # FinalEvents.dat → ROOT converter
  data/
    gibuudata.dat                         # GiBUU ID → PDG mapping
  include/
    EvtTracker2event.h                    # GiBUU event parser
    tkievent.h                            # NeutrinoEvent data structure
    dochi2.h                              # χ² computation with covariance
    plottools.hxx                         # Publication-quality plotting
  src/
    EvtTracker2event.cxx
    tkievent.cxx
    dochi2.cxx
    plottools.cxx
```
