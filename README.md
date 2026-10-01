# Distributed Order Management : QAOA + MILP Optimization

## Executive Summary

Nestlé’s Distributed Order Management (DOM) maximizes net profit by rerouting orders to alternate distribution centers during stockouts or warehouse bottlenecks, balancing sales revenue against extra shipping costs and missed-order penalties.

This project tests and compares two optimization approaches:

1. **Classical Model (MILP):** A full-scale, 7-constraint mathematical model (built with PuLP/CBC) that handles all real-world demand-solving 170 to 450 orders per planning date.
2. **Quantum Algorithm (QAOA):** A simulated quantum circuit designed for complex combinatorial searches on smaller, targeted sets of orders. It uses specialized quantum mixers and fine-tuning to find valid fulfillment routes efficiently.

Both approaches are benchmarked against standard industry rules (never rerouting vs. greedy rerouting) and verified against the true, mathematically proven optimal solution.

---

## Key Results & Findings

* **Fast and Proven at Full Scale (MILP):** The classical MILP model solves real-world workloads (up to 426 orders) to guaranteed mathematical perfection in under 2.5 minutes. By virtually wiping out late/missed-order penalties, it boosts net profit by **8% to 15%** over standard baseline rules.


* **100% Accurate on Small Tests (QAOA):** On smaller benchmark tests (5 orders), the simulated quantum algorithm (QAOA) proved completely reliable-hitting the exact optimal solution every single time (a **1.00 F1-score** and **0.00% error gap**).


* **Quantum Bottlenecks & Hardware Limits:** Each order needs about 4 qubits. While QAOA works well on test subset ($\le$20 qubits), it runs **15× to 50× slower** than classical solvers. Scaling this to full production demand would require 1,700+ qubits-which is too large for simulators to run and would require quantum hardware.

---

## Repository Structure

```text
├── Notebook/                      
│   ├── Dom_Milp_Optimization.ipynb
│   └── Nestle_DOM_QAOA_MILP.ipynb
├── Report
│   └── DOM_Technical_Report.pdf
├── Results
│   ├── HYBRID (QAOA+MILP)/  
│   │   ├── figures/               # Comparison, runsummary, etc. plots
│   │   ├── comparison_all_dates.csv
│   │   ├── noise_study_all_dates.csv
│   │   ├── order_level_all_dates.csv
│   │   ├── qubit_scaling_all_dates.csv
│   │   ├── robustness_all_dates.csv
│   │   ├── robustness_summary_all_dates.csv
│   │   └── run_summary_all_dates.csv
│   └── MILP/                    
│       ├── Figures/               # Comparative performance charts
│       ├── Comparison_all_dates.csv
│       └── Run_Summary_all_dates.csv
├── .gitignore
├── README.md                  
└── requirements_root.txt
```

---

## Business Rules & Constraints

---

The model strictly follows Nestlé's 7 real-world fulfillment rules:

* **C1 - Single Sourcing:** Each order is sent to at most one distribution center (DC) or left unassigned.


* **C2 - Demand Cap:** Shipped amounts cannot exceed ordered quantities.


* **C3a - Stock Limits:** Sourced cases cannot exceed available inventory per DC, SKU, and date.


* **C3b - 5-Day Stock Buffer:** Rerouted orders cannot deplete stock that the alternate DC needs over the next 5 days.


* **C4 - Rerouting Hurdle:** An alternate DC is only used if it boosts fill rate by at least 5% of demand or 100 cases.


* **C5 - Warehouse Handling:** Orders are split into full pallets and individual case-picks based on daily handling capacities.


* **C6 - Dock Door Limits:** Total shipments cannot exceed daily dock appointment slots.


* **C7 - Service Penalties:** Shortfall penalties trigger if fulfillment drops below the customer's minimum fill-rate threshold.



---

## Solver Approaches

---

#### 1. Classical MILP Approach

The Mixed-Integer Linear Programming model treats the problem as a full mathematical optimization using binary variables for DC assignments and linear equations for capacities. Solved with PuLP/CBC, it evaluates the entire trade-off between shipping costs, penalties, and revenues simultaneously, proving the globally optimal solution across the full network.

#### 2. Quantum QAOA Approach

The problem is converted into a binary optimization format (QUBO / Ising Hamiltonian). Instead of using heavy penalty terms to enforce single sourcing (C1), it applies a specialized **XY-mixer** to keep the quantum circuit strictly within valid one-DC-per-order states. Remaining constraints are balanced using mathematical penalties, and a **CVaR objective** focuses the algorithm on the top 20% best outcomes.

---

## Solvers & Methodology

| Solver | Category | Implementation | Role |
|---|---|---|---|
| **Baseline 1** | Deterministic | Strict Default DC | Operational lower bound; serves orders only from default DCs. |
| **Baseline 2** | Heuristic | Greedy Sequential | Greedily diverts orders that clear the 5% / 100-case hurdle. |
| **MILP** | Exact Classical | PuLP + CBC | Global optimization of the full 7-constraint formulation. |
| **QAOA** | Gate Quantum | Variational Circuit | Evaluates circuit depth, mixer performance, and noise tolerance. |
| **Certified Optimum** | Brute Force | Combinatorial Search | Exhaustive enumeration of subinstances to establish ground truth. |

---

## Data Pipeline

The pipeline ingests 5 operational CSV datasets joined on DC location, Material Number (SKU), and planned Goods Issue (PGI) date:

* `input_order_data.csv`: Open customer orders, demand quantities, revenues, default DCs, and cut penalty rates.


* `input_capacity_planning.csv`: Daily on-hand available inventory per DC, SKU, and date.


* `input_dock_capacity.csv`: Daily available truck dock appointment limits per DC.


* `input_shipping_cost_data.csv`: Freight shipping costs per customer code and plant.


* `input_throughput_capacity.csv`: Daily warehouse pallet-pick and case-pick handling limits per DC.

> *Note: Data files are not included in this repo (proprietary to the challenge data pack). Upload them when prompted by the notebook.*
---

## Installation & Setup

Ensure Python 3.10 or higher version is installed.

```bash
# Clone the repository
git clone https://github.com/indujakodavati/dom-quantum-optimization.git
cd dom-quantum-optimization

# Install project dependencies
pip install -r requirements.txt
```

### Core Dependencies (`requirements.txt`)
```text
qiskit==2.5.1
qiskit-aer==0.17.2
pulp==2.9.0
pandas>=2.0.0
numpy>=1.24.0
scipy>=1.10.0
matplotlib>=3.7.0
```

---

## Running the Notebooks

### 1. Full-Scale MILP Pipeline
Open `Notebook/Dom_Milp_Optimization.ipynb` in Google Colab or Jupyter:
1. Mount Google Drive or set `DATA_ROOT` to the directory containing your CSVs.
2. Set `TARGET_DATES` to your desired planning horizon (e.g., `2024-06-20` through `2024-06-24`).
3. Run all cells. Summary tables and comparative figures are written directly to `Results/MILP/'.

### 2. Hybrid QAOA + MILP Benchmark Pipeline
Open `Notebook/Nestle_DOM_QAOA_MILP.ipynb` in Google Colab or Jupyter:
1. Mount Google Drive or verify `DATA_ROOT`.
2. Set `cfg.current_date` in the configuration cell (e.g., `2024-06-17`, `2024-06-19`, `2024-06-21`, `2024-06-23`, `2024-06-25`).
3. Run all cells. Outputs, robustness seeds, and noise evaluations are saved to `Results/HYBRID (QAOA+MILP)/`.

---

## Deliverables

* 📓 **Notebooks:** 
  * [`Notebook/Dom_Milp_Optimization.ipynb`](Notebook/Dom_Milp_Optimization.ipynb): Full-scale classical enterprise MILP solver.
  * [`Notebook/Nestle_DOM_QAOA_MILP.ipynb`](Notebook/Nestle_DOM_QAOA_MILP.ipynb): Subinstance QAOA, MILP, and brute-force benchmark workflow.
* 📄 **Technical Report:** [`DOM_Technical_Report.pdf`](Report/Nestle_DOM_Technical_Report.pdf): Complete technical and mathematical formulation report.
* 📊 **Empirical Results:** [`Results/`](Results/): Raw CSV outputs backing all tables, sensitivity curves, and noise evaluations.

---

## Development Team

* Harika Srilakshmi Durga Yanamadala - [harikayanamadala1411@gmail.com](mailto:harikayanamadala1411@gmail.com) 
* Induja Bhanu Kodavati - [indujabhanu555@gmail.com](mailto:indujabhanu555@gmail.com)
