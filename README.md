# Noise Adaptation in NISQ Quantum Machine Learning

This repository contains code and scripts to reproduce the experimental results and plots from the paper

Noise Adaptation in NISQ Quantum Machine Learning: From Mitigation Limits to Benefit Characterization

It examines when measured noise regimes are harmful, when they are tolerable, and when they exhibit narrow transient benefits under explicit resource budgets. The repository supports two goals.

1. Reproduce the simulator based case studies that show how different noise channels can change early optimization dynamics.
2. Reproduce the cost aware, profile aggregated evaluation metric, Noise Adapted Performance Ratio, together with robustness diagnostics.

## Repository layout

* `NAPR/` contains scripts that compute Noise Adapted Performance Ratio and generate the NAPR plots for VQE, QAOA, and VQC.
* `noise_benefits/` contains simulator case studies that visualize how noise can change early convergence for VQE and VQC.
* `real_machine/` contains optional IBM Quantum hardware snapshots used to illustrate mitigation overhead and unavoidable error accumulation.
* `requirements.txt` contains a minimal pinned set of Python dependencies for the simulator workflows.

## Environment setup

This code was developed with Python 3.10 and Qiskit 1.4.4. A clean virtual environment is recommended.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -r requirements.txt
```

Some scripts require additional optional packages.

* For VQC scripts in `noise_benefits/` and `NAPR/VQC_napr.py`

```bash
pip install qiskit-machine-learning scikit-learn
```

* For IBM Quantum hardware scripts in `real_machine/`

```bash
pip install qiskit-ibm-runtime
```

## Quick start

### 1. Compute NAPR on simulator profiles

Run these from the repository root.

VQE NAPR

```bash
cd NAPR
mkdir -p napr_plots
python VQE_napr.py
```

Expected output

* `napr_plots/noise_metric.png` and `napr_plots/VQE_Energy_Metric.png`

QAOA NAPR

```bash
cd NAPR
python QAOA_napr.py
```

Expected output

* `napr_metric/qaoa_napr.png`

VQC NAPR

```bash
cd NAPR
python VQC_napr.py
```

Expected output

* `napr_metric/vqc_napr.png`

### 2. Visualize early optimization effects on simulator

VQE energy traces under different noise models

```bash
cd noise_benefits
python vqe_noise.py
```

VQC loss traces under different noise models

```bash
cd noise_benefits
python vqc_noise.py
```

These scripts open Matplotlib figures. If you want files on disk, add a `plt.savefig(...)` line in the plotting block.

## Reproducing paper figures

This section maps the main scripts to the common figures used in the paper.

### Figure: Unavoidable error accumulation on hardware

This script runs a short depth sweep on an IBM backend and plots how a simple success probability decays as the circuit depth grows.

```bash
cd real_machine
python Unavoidable_noise.py
```

It saves a plot as `xx.png` in the current directory.

### Figure: Mitigation overhead snapshot on hardware

This script compares baseline execution with randomized compiling, zero noise extrapolation, and their combination on a Bell state witness task.

Important note: the file name in this repository includes a leading space. Use quotes exactly as shown, or rename the file.

```bash
cd real_machine
python " zne_rc_snapshot.py"
```

### Figure: Benefit characterization with NAPR

NAPR plots are produced by the scripts in `NAPR/`. Each script evaluates a workload across a set of calibrated noise profiles and reports

* Per profile ratios `r_i`
* Mean NAPR across profiles
* Minimal robustness score `r_min`
* Ratio variance as a sensitivity diagnostic

The scripts also generate the corresponding bar plots.

## Hardware access for `real_machine/`

Hardware runs require an IBM Quantum account and access to Qiskit Runtime.

One time login and save credentials

```bash
python -c "from qiskit_ibm_runtime import QiskitRuntimeService as S; S.save_account(channel='ibm_quantum', token='YOUR_TOKEN_HERE', overwrite=True)"
```

Then you can run the hardware scripts. Each script has a `BACKEND_NAME` or `BACKEND_PREFERRED` variable near the top that you can edit.

Notes

* Hardware calibrations change over time. Exact numeric results will vary.
* Queuing time can dominate wall clock time.
* The paper treats hardware runs as snapshots that illustrate cost and drift, not as fixed baselines.

## Noise profiles and budgets

The simulator scripts represent noise with common channel abstractions, such as depolarizing noise, amplitude damping, phase damping, thermal relaxation, gate error, measurement error, and a composite full model.

The key evaluation principle in the paper is budget grounding.

* Each run uses a fixed number of optimizer iterations.
* Each circuit evaluation uses a fixed number of shots.
* When mitigation is enabled on hardware, its extra sampling and runtime cost is treated as part of the cost.

The NAPR scripts implement the profile aggregated view by evaluating the same protocol across multiple profiles, instead of relying on a single calibration snapshot.

## Output files

By default the repository writes plots into these locations.

* `NAPR/napr_plots/` contains VQE NAPR plots.
* `NAPR/napr_metric/` contains QAOA and VQC NAPR plots.
* `real_machine/xx.png` is produced by `Unavoidable_noise.py`.

If you prefer a single output directory, edit the `outdir` variable in the corresponding script.

## Reproducibility notes

* Most scripts set a fixed random seed. If you change seeds, expect small shifts in early iteration traces.
* The VQC scripts download MNIST through OpenML the first time they run.
* Hardware scripts depend on device availability, calibration drift, and queue time.
