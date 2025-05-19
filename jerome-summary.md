### Main Bottlenecks Hindering Scalability

1. Constraint Enforcement via Penalty Terms
Encoding constraints (like budget or diversification) as penalties in the cost Hamiltonian significantly increases complexity. This leads to harder optimization landscapes and larger circuits, which don’t scale well with increasing qubit counts.

2. Complex Mixer Circuits for Constraints
Constraint-preserving mixers such as XY-mixers require deep circuits with many two-qubit gates. On large-qubit systems, this quickly becomes unmanageable due to circuit depth limits and decoherence.

3. Inefficient Classical Optimization Loop
The classical optimizer (e.g., COBYLA) becomes a major bottleneck as the number of QAOA parameters increases with both qubit count and circuit depth (p). Optimization time and memory usage increase non-linearly.

4. Quantum Circuit Simulation Overhead
Simulating QAOA on a classical backend scales exponentially in the number of qubits. For portfolios >15 assets, simulation becomes infeasible unless strategies are applied to reduce circuit complexity or qubit count.



### Recommended Improvements (Scalable QAOA Execution)
1. Constraint-Preserving Mixer Redesign
Why: Avoids encoding constraints as costly penalty terms in the cost Hamiltonian.
What to do:
- Replace XY-mixers with custom hard-constraint-preserving mixers (e.g., using parity, subset-swap, or token-mixing).
- Directly encode constraints into the mixer evolution.

Test:
- Evaluate circuit depth and gate count as qubit number increases (N = 10, 15, 20).
- Compare success rate and convergence speed with and without constraint-aware mixers.

2.  Switch to Scalable Classical Optimizers
Why: Standard optimizers (like COBYLA or Nelder-Mead) don’t scale well for large parameter spaces.
What to do:
- Use L-BFGS-B with gradient approximation or Bayesian Optimization for fewer function calls.
- Add warm-start initialization from classical heuristics (e.g., greedy portfolio).

Test:
Benchmark optimization runtime and convergence stability for N = 10–25 assets.
Measure optimizer evaluation count and solution quality.

3. ✅ Circuit Depth Reduction via Compilation and Scheduling
Why: Large circuits quickly exceed depth limits on real hardware.
What to do:

-Use advanced transpilers (e.g., Qiskit’s Level 3, tket) to optimize layout and two-qubit gate schedules.
- Apply gate cancellation and routing-aware mapping for NISQ devices.

Test:

Track 2-qubit gate count and total depth before/after transpilation.
Run noise-aware simulations to measure fidelity loss for N = 15–25.

5. Incremental QAOA Depth (Adaptive Layering)
Why: Using a fixed high depth p for QAOA is wasteful and costly at scale.
What to do:

- Implement an adaptive QAOA scheme that increases depth (p) only if solution improvement justifies it.

- Stop when marginal gain drops below a threshold.

Test:

Benchmark performance vs. fixed-depth QAOA.
Analyze time to convergence and circuit depth scalability for large N.

Suggested Metrics to Track
- Execution Time per Iteration
- Circuit Depth and Width
- Optimizer Evaluations
- Qubit Count and Memory Footprint
- Approximation Ratio vs. Global Optimum


Tools & Platforms for Scaling
Qiskit + AerSimulator: Scalable simulation with noise models and runtime metrics.

Qiskit Runtime: For faster hybrid iterations with optimizer running on IBM Cloud.

TensorCircuit / Pennylane: Explore alternate frameworks with GPU acceleration.

cProfile + line_profiler: To profile Python code and optimizer bottlenecks.


Put in place the profiler: 

For the portfolio optimization use case, the bottlenecks are typically in:

File	Function	Role
qokit/algorithms/qaoa/qaoa_optimizer.py	solve	Main optimization loop
qokit/algorithms/qaoa/qaoa_optimizer.py	_run_qaoa	Core QAOA call per iteration
qokit/algorithms/qaoa/qaoa_optimizer.py	_evaluate_cost	Quantum circuit execution & expectation calculation
qokit/problems/portfolio/portfolio_qaoa.py	build_cost_operator, build_mixer_operator	Hamiltonian construction (scales with assets/qubits)

