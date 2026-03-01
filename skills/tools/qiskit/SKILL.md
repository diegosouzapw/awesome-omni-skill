---
name: qiskit
description: Comprehensive guide for Qiskit - IBM's quantum computing framework. Use for quantum circuit design, quantum algorithms (VQE, QAOA, Grover, Shor), quantum simulation, noise modeling, quantum machine learning, and quantum chemistry calculations. Essential for quantum computing research and applications.
version: 1.0
license: Apache-2.0
---

# Qiskit - Quantum Computing Framework

Open-source quantum computing framework for building, simulating, and running quantum algorithms on quantum computers and simulators.

## When to Use

- Building quantum circuits and gates
- Running quantum algorithms (VQE, QAOA, Grover, Shor)
- Quantum chemistry calculations (integration with PySCF)
- Quantum machine learning
- Quantum simulation and noise modeling
- Transpiling circuits for real quantum hardware
- Quantum optimization problems
- Quantum error correction
- Quantum cryptography
- Educational quantum computing demonstrations

## Reference Documentation

**Official docs**: https://qiskit.org/documentation/  
**Search patterns**: `qiskit.circuit.QuantumCircuit`, `qiskit.algorithms.VQE`, `qiskit.quantum_info`, `qiskit_nature`

## Core Principles

### Use Qiskit For

| Task | Module | Example |
|------|--------|---------|
| Circuit building | `qiskit` | `QuantumCircuit(2, 2)` |
| Quantum algorithms | `qiskit.algorithms` | `VQE(ansatz, optimizer)` |
| Quantum simulation | `qiskit.providers.aer` | `AerSimulator()` |
| Quantum chemistry | `qiskit_nature` | `GroundStateEigensolver()` |
| Noise modeling | `qiskit.providers.aer.noise` | `NoiseModel()` |
| Transpilation | `qiskit.transpiler` | `transpile(circuit, backend)` |
| Quantum ML | `qiskit_machine_learning` | `VQC(feature_map, ansatz)` |
| Visualization | `qiskit.visualization` | `plot_histogram(counts)` |

### Do NOT Use For

- Classical machine learning (use scikit-learn, PyTorch)
- Classical optimization (use SciPy)
- General numerical computing (use NumPy)
- Classical cryptography (use cryptography package)
- Large-scale classical simulation (use classical simulators)

## Quick Reference

### Installation

```bash
# Core Qiskit
pip install qiskit

# With visualization tools
pip install qiskit[visualization]

# Quantum chemistry extension
pip install qiskit-nature qiskit-nature-pyscf

# Machine learning extension
pip install qiskit-machine-learning

# Optimization extension
pip install qiskit-optimization

# Full installation
pip install 'qiskit[all]' qiskit-nature qiskit-machine-learning qiskit-optimization
```

### Standard Imports

```python
# Core imports
from qiskit import QuantumCircuit, QuantumRegister, ClassicalRegister
from qiskit import transpile, assemble
from qiskit.providers.aer import AerSimulator
from qiskit.visualization import plot_histogram, plot_bloch_multivector

# Quantum algorithms
from qiskit.algorithms import VQE, QAOA, Grover, Shor
from qiskit.algorithms.optimizers import SLSQP, COBYLA, SPSA

# Quantum info
from qiskit.quantum_info import Statevector, DensityMatrix, Operator
from qiskit.quantum_info import entropy, entanglement_of_formation

# Circuit library
from qiskit.circuit.library import QFT, RealAmplitudes, EfficientSU2
```

### Basic Pattern - Circuit Building

```python
from qiskit import QuantumCircuit
from qiskit.providers.aer import AerSimulator

# Create circuit
qc = QuantumCircuit(2, 2)

# Add gates
qc.h(0)              # Hadamard on qubit 0
qc.cx(0, 1)          # CNOT from 0 to 1

# Measure
qc.measure([0, 1], [0, 1])

# Simulate
simulator = AerSimulator()
job = simulator.run(qc, shots=1000)
result = job.result()
counts = result.get_counts()

print(f"Results: {counts}")
```

### Basic Pattern - Quantum Algorithm

```python
from qiskit import QuantumCircuit
from qiskit.algorithms import VQE
from qiskit.algorithms.optimizers import SLSQP
from qiskit.circuit.library import RealAmplitudes
from qiskit.primitives import Estimator
from qiskit.quantum_info import SparsePauliOp

# Define Hamiltonian
hamiltonian = SparsePauliOp(['ZZ', 'IZ', 'ZI'], coeffs=[1.0, -0.5, -0.5])

# Create ansatz
ansatz = RealAmplitudes(num_qubits=2, reps=1)

# Setup VQE
optimizer = SLSQP(maxiter=100)
estimator = Estimator()
vqe = VQE(estimator, ansatz, optimizer)

# Run
result = vqe.compute_minimum_eigenvalue(hamiltonian)
print(f"Ground state energy: {result.eigenvalue:.6f}")
```

## Critical Rules

### ✅ DO

- **Use simulators for development** - Test on simulators before real hardware
- **Transpile for target backend** - Always transpile circuits for specific hardware
- **Handle measurement statistics** - Work with shot counts, not single results
- **Use primitives for algorithms** - Use Estimator/Sampler primitives
- **Check circuit depth** - Monitor gate count and depth for real hardware
- **Implement error mitigation** - Use error mitigation for noisy hardware
- **Validate quantum states** - Check state validity and normalization
- **Use appropriate basis gates** - Match hardware native gates
- **Set random seed for reproducibility** - Use seed for consistent results
- **Monitor job status** - Check if quantum jobs complete successfully

### ❌ DON'T

- **Ignore hardware constraints** - Real quantum computers have limitations
- **Use too many qubits on simulators** - Memory grows exponentially
- **Forget to measure** - Quantum states collapse on measurement
- **Mix classical and quantum incorrectly** - Understand measurement timing
- **Ignore decoherence** - Quantum states decay over time
- **Over-transpile** - Unnecessary transpilation adds gates
- **Assume perfect gates** - Real gates have errors
- **Ignore topology** - Not all qubits are connected
- **Use deprecated APIs** - Qiskit evolves rapidly
- **Run without error handling** - Quantum jobs can fail

## Anti-Patterns (NEVER)

```python
from qiskit import QuantumCircuit
from qiskit.providers.aer import AerSimulator
from qiskit.primitives import Estimator

# ❌ BAD: No measurement
qc = QuantumCircuit(2, 2)
qc.h(0)
qc.cx(0, 1)
# Forgot qc.measure()!

# ✅ GOOD: Always measure when needed
qc = QuantumCircuit(2, 2)
qc.h(0)
qc.cx(0, 1)
qc.measure([0, 1], [0, 1])

# ❌ BAD: Using deprecated execute()
from qiskit import execute
result = execute(qc, backend, shots=1024).result()

# ✅ GOOD: Use new run() method
simulator = AerSimulator()
job = simulator.run(qc, shots=1024)
result = job.result()

# ❌ BAD: Assuming perfect measurement
counts = result.get_counts()
# Assuming exactly 50/50 split!
assert counts['00'] == 512

# ✅ GOOD: Handle statistical variation
counts = result.get_counts()
ratio = counts.get('00', 0) / sum(counts.values())
print(f"Measured |00⟩ with probability {ratio:.3f}")

# ❌ BAD: Not checking circuit properties
qc = QuantumCircuit(20)  # Many qubits!
# Adding many gates...
# Trying to simulate without checking depth/size!

# ✅ GOOD: Check circuit properties
qc = QuantumCircuit(20)
# ... add gates ...
print(f"Circuit depth: {qc.depth()}")
print(f"Gate count: {len(qc.data)}")
print(f"Qubits: {qc.num_qubits}")

# ❌ BAD: Ignoring transpilation
job = backend.run(qc)  # May fail on real hardware!

# ✅ GOOD: Transpile for backend
from qiskit import transpile
transpiled_qc = transpile(qc, backend=backend, optimization_level=3)
job = backend.run(transpiled_qc)
```

## Quantum Circuits (qiskit.QuantumCircuit)

### Basic Circuit Construction

```python
from qiskit import QuantumCircuit, QuantumRegister, ClassicalRegister
import numpy as np

# Method 1: Simple initialization
qc = QuantumCircuit(3, 3)  # 3 qubits, 3 classical bits

# Method 2: Using registers
qr = QuantumRegister(3, 'q')
cr = ClassicalRegister(3, 'c')
qc = QuantumCircuit(qr, cr)

# Method 3: Multiple registers
qr1 = QuantumRegister(2, 'data')
qr2 = QuantumRegister(1, 'ancilla')
cr = ClassicalRegister(2, 'meas')
qc = QuantumCircuit(qr1, qr2, cr)

print(f"Number of qubits: {qc.num_qubits}")
print(f"Number of classical bits: {qc.num_clbits}")
print(f"Circuit depth: {qc.depth()}")
```

### Single-Qubit Gates

```python
from qiskit import QuantumCircuit
import numpy as np

qc = QuantumCircuit(1)

# Pauli gates
qc.x(0)   # Pauli X (NOT gate)
qc.y(0)   # Pauli Y
qc.z(0)   # Pauli Z

# Hadamard gate
qc.h(0)   # Creates superposition

# Phase gates
qc.s(0)   # S gate (π/2 phase)
qc.t(0)   # T gate (π/4 phase)
qc.sdg(0) # S dagger
qc.tdg(0) # T dagger

# Rotation gates
qc.rx(np.pi/4, 0)  # Rotation around X
qc.ry(np.pi/4, 0)  # Rotation around Y
qc.rz(np.pi/4, 0)  # Rotation around Z

# General rotation
qc.u(np.pi/4, np.pi/2, np.pi, 0)  # U gate

# Identity (wait)
qc.id(0)

print(f"Gate count: {len(qc.data)}")
```

### Two-Qubit Gates

```python
from qiskit import QuantumCircuit
import numpy as np

qc = QuantumCircuit(2)

# CNOT (Controlled-NOT)
qc.cx(0, 1)  # Control: 0, Target: 1

# Other controlled gates
qc.cy(0, 1)  # Controlled-Y
qc.cz(0, 1)  # Controlled-Z
qc.ch(0, 1)  # Controlled-Hadamard

# SWAP gate
qc.swap(0, 1)

# Controlled phase
qc.cp(np.pi/4, 0, 1)

# Controlled-U
qc.cu(np.pi/4, np.pi/2, np.pi, 0, 0, 1)

# Toffoli (CCX) - needs 3 qubits
qc_3 = QuantumCircuit(3)
qc_3.ccx(0, 1, 2)  # Controls: 0,1, Target: 2

print(qc.draw())
```

### Creating Entanglement

```python
from qiskit import QuantumCircuit
from qiskit.providers.aer import AerSimulator
from qiskit.quantum_info import Statevector

# Bell state (maximally entangled)
def create_bell_state():
    qc = QuantumCircuit(2)
    qc.h(0)
    qc.cx(0, 1)
    return qc

bell = create_bell_state()
state = Statevector.from_instruction(bell)
print(f"Bell state: {state}")

# GHZ state (3-qubit entanglement)
def create_ghz_state(n):
    qc = QuantumCircuit(n)
    qc.h(0)
    for i in range(n-1):
        qc.cx(i, i+1)
    return qc

ghz = create_ghz_state(3)
state_ghz = Statevector.from_instruction(ghz)
print(f"GHZ state: {state_ghz}")

# W state (another type of 3-qubit entanglement)
def create_w_state():
    qc = QuantumCircuit(3)
    qc.ry(1.9106, 0)
    qc.ch(0, 1)
    qc.x(0)
    qc.cy(0, 1)
    qc.ccx(0, 1, 2)
    qc.x(0)
    return qc

w = create_w_state()
print(f"W state circuit depth: {w.depth()}")
```

### Parameterized Circuits

```python
from qiskit import QuantumCircuit
from qiskit.circuit import Parameter, ParameterVector
import numpy as np

# Single parameter
theta = Parameter('θ')
qc = QuantumCircuit(1)
qc.ry(theta, 0)

# Bind parameter
bound_qc = qc.bind_parameters({theta: np.pi/4})
print(f"Unbound: {qc}")
print(f"Bound: {bound_qc}")

# Multiple parameters
params = ParameterVector('θ', 4)
qc_param = QuantumCircuit(2)
qc_param.ry(params[0], 0)
qc_param.ry(params[1], 1)
qc_param.cx(0, 1)
qc_param.ry(params[2], 0)
qc_param.ry(params[3], 1)

# Bind all parameters
values = [np.pi/4, np.pi/3, np.pi/2, np.pi/6]
bound = qc_param.bind_parameters(dict(zip(params, values)))

print(f"Number of parameters: {qc_param.num_parameters}")
```

## Quantum Algorithms

### Variational Quantum Eigensolver (VQE)

```python
from qiskit import QuantumCircuit
from qiskit.algorithms import VQE
from qiskit.algorithms.optimizers import SLSQP, COBYLA
from qiskit.circuit.library import RealAmplitudes, EfficientSU2
from qiskit.primitives import Estimator
from qiskit.quantum_info import SparsePauliOp
import numpy as np

# Define Hamiltonian (e.g., H2 molecule)
# H = -1.05 * ZZ + 0.39 * XX - 0.39 * YY - 0.01 * ZI
hamiltonian = SparsePauliOp(
    ['ZZ', 'XX', 'YY', 'ZI', 'IZ'],
    coeffs=[-1.05, 0.39, -0.39, -0.01, -0.01]
)

# Create ansatz (variational form)
ansatz = RealAmplitudes(num_qubits=2, reps=2)

# Alternative: EfficientSU2
# ansatz = EfficientSU2(num_qubits=2, reps=2)

# Setup optimizer
optimizer = SLSQP(maxiter=100)

# Create estimator primitive
estimator = Estimator()

# Setup and run VQE
vqe = VQE(estimator, ansatz, optimizer)
result = vqe.compute_minimum_eigenvalue(hamiltonian)

print(f"Ground state energy: {result.eigenvalue:.6f}")
print(f"Optimal parameters: {result.optimal_parameters}")
print(f"Optimizer evaluations: {result.cost_function_evals}")

# Get optimal circuit
optimal_circuit = ansatz.bind_parameters(result.optimal_point)
print(f"\nOptimal circuit depth: {optimal_circuit.depth()}")
```

### QAOA (Quantum Approximate Optimization Algorithm)

```python
from qiskit import QuantumCircuit
from qiskit.algorithms import QAOA
from qiskit.algorithms.optimizers import COBYLA
from qiskit.primitives import Sampler
from qiskit.quantum_info import SparsePauliOp
import numpy as np

# Max-Cut problem on a triangle graph
# Cost Hamiltonian: H = (1-Z0*Z1)/2 + (1-Z1*Z2)/2 + (1-Z0*Z2)/2
cost_hamiltonian = SparsePauliOp(
    ['ZZ', 'ZI', 'IZ'],
    coeffs=[0.5, -0.5, -0.5]
)

# Setup QAOA
optimizer = COBYLA(maxiter=100)
sampler = Sampler()
qaoa = QAOA(sampler, optimizer, reps=2)

# Run QAOA
result = qaoa.compute_minimum_eigenvalue(cost_hamiltonian)

print(f"Optimal objective: {result.eigenvalue:.6f}")
print(f"Optimal parameters: {result.optimal_parameters}")

# Most probable solution
from qiskit.result import QuasiDistribution
prob_dist = result.eigenstate
if isinstance(prob_dist, QuasiDistribution):
    most_likely = max(prob_dist, key=prob_dist.get)
    print(f"Most likely solution: {bin(most_likely)[2:].zfill(2)}")
```

### Grover's Algorithm (Quantum Search)

```python
from qiskit import QuantumCircuit
from qiskit.algorithms import Grover, AmplificationProblem
from qiskit.circuit.library import GroverOperator
from qiskit.primitives import Sampler
import numpy as np

def grover_search(marked_states, n_qubits):
    """
    Grover's algorithm to find marked states.
    
    Args:
        marked_states: List of marked states (e.g., ['101', '110'])
        n_qubits: Number of qubits
    """
    # Create oracle that marks the target states
    oracle = QuantumCircuit(n_qubits)
    
    # Mark state |101⟩ by flipping phase
    if '101' in marked_states:
        oracle.x([0, 2])  # Flip to make 101 -> 111
        oracle.h(2)
        oracle.ccx(0, 1, 2)
        oracle.h(2)
        oracle.x([0, 2])  # Flip back
    
    # Grover diffusion operator
    diffusion = QuantumCircuit(n_qubits)
    diffusion.h(range(n_qubits))
    diffusion.x(range(n_qubits))
    diffusion.h(n_qubits - 1)
    diffusion.mcx(list(range(n_qubits - 1)), n_qubits - 1)
    diffusion.h(n_qubits - 1)
    diffusion.x(range(n_qubits))
    diffusion.h(range(n_qubits))
    
    # Complete Grover iteration
    grover_op = oracle.compose(diffusion)
    
    # Setup and run
    problem = AmplificationProblem(oracle, is_good_state=marked_states)
    grover = Grover(sampler=Sampler())
    result = grover.amplify(problem)
    
    return result

# Example: Search for |101⟩ in 3-qubit space
result = grover_search(['101'], 3)
print(f"Top measurement: {result.top_measurement}")
print(f"Oracle evaluations: {result.oracle_evaluation}")

# Simpler example: 2-qubit search
qc = QuantumCircuit(2, 2)
qc.h([0, 1])  # Superposition

# Oracle marks |11⟩
qc.cz(0, 1)

# Diffusion
qc.h([0, 1])
qc.x([0, 1])
qc.cz(0, 1)
qc.x([0, 1])
qc.h([0, 1])

qc.measure([0, 1], [0, 1])

from qiskit.providers.aer import AerSimulator
simulator = AerSimulator()
job = simulator.run(qc, shots=1000)
counts = job.result().get_counts()
print(f"Grover results: {counts}")
```

### Quantum Phase Estimation (QPE)

```python
from qiskit import QuantumCircuit
from qiskit.circuit.library import QFT
from qiskit.providers.aer import AerSimulator
import numpy as np

def quantum_phase_estimation(unitary, n_counting_qubits):
    """
    Estimate the phase of an eigenstate of a unitary operator.
    
    Args:
        unitary: Unitary operator as a QuantumCircuit
        n_counting_qubits: Precision qubits
    """
    n_system_qubits = unitary.num_qubits
    
    qc = QuantumCircuit(n_counting_qubits + n_system_qubits, n_counting_qubits)
    
    # Initialize eigenstate (simplified: use |1⟩)
    qc.x(n_counting_qubits)
    
    # Hadamard on counting qubits
    for i in range(n_counting_qubits):
        qc.h(i)
    
    # Controlled-U^(2^i) operations
    repetitions = 1
    for counting_qubit in range(n_counting_qubits):
        for _ in range(repetitions):
            qc.append(unitary.control(), [counting_qubit] + 
                     list(range(n_counting_qubits, n_counting_qubits + n_system_qubits)))
        repetitions *= 2
    
    # Inverse QFT
    qc.append(QFT(n_counting_qubits, inverse=True), range(n_counting_qubits))
    
    # Measure counting qubits
    qc.measure(range(n_counting_qubits), range(n_counting_qubits))
    
    return qc

# Example: Phase estimation for T gate (phase π/4)
t_gate = QuantumCircuit(1)
t_gate.t(0)

qpe_circuit = quantum_phase_estimation(t_gate, n_counting_qubits=3)

simulator = AerSimulator()
job = simulator.run(qpe_circuit, shots=1000)
counts = job.result().get_counts()
print(f"Phase estimation results: {counts}")

# Most common result gives phase
most_common = max(counts, key=counts.get)
phase_estimate = int(most_common, 2) / (2**3)
print(f"Estimated phase: {phase_estimate * 2 * np.pi:.4f} rad")
print(f"True phase: {np.pi/4:.4f} rad")
```

## Quantum Chemistry with Qiskit Nature

### Molecular Ground State Calculation

```python
# Requires: pip install qiskit-nature qiskit-nature-pyscf

from qiskit_nature.units import DistanceUnit
from qiskit_nature.second_q.drivers import PySCFDriver
from qiskit_nature.second_q.mappers import JordanWignerMapper, ParityMapper
from qiskit_nature.second_q.circuit.library import HartreeFock, UCCSD
from qiskit.algorithms import VQE
from qiskit.algorithms.optimizers import SLSQP
from qiskit.primitives import Estimator
import numpy as np

# Define molecule (H2)
driver = PySCFDriver(
    atom='H 0 0 0; H 0 0 0.735',
    basis='sto3g',
    charge=0,
    spin=0,
    unit=DistanceUnit.ANGSTROM
)

# Get problem
problem = driver.run()
hamiltonian = problem.hamiltonian.second_q_op()

# Map to qubits
mapper = JordanWignerMapper()
qubit_op = mapper.map(hamiltonian)

print(f"Number of qubits required: {qubit_op.num_qubits}")

# Initial state (Hartree-Fock)
num_particles = problem.num_particles
num_spatial_orbitals = problem.num_spatial_orbitals

init_state = HartreeFock(
    num_spatial_orbitals=num_spatial_orbitals,
    num_particles=num_particles,
    qubit_mapper=mapper
)

# Ansatz (UCCSD)
ansatz = UCCSD(
    num_spatial_orbitals=num_spatial_orbitals,
    num_particles=num_particles,
    qubit_mapper=mapper,
    initial_state=init_state
)

# VQE calculation
optimizer = SLSQP(maxiter=100)
estimator = Estimator()

vqe = VQE(estimator, ansatz, optimizer)
result = vqe.compute_minimum_eigenvalue(qubit_op)

print(f"\nVQE Ground State Energy: {result.eigenvalue:.6f} Ha")
print(f"Reference HF energy: {problem.reference_energy:.6f} Ha")
```

### Excited States Calculation

```python
from qiskit_nature.second_q.algorithms import GroundStateEigensolver, ExcitedStatesEigensolver
from qiskit_nature.second_q.algorithms import QEOM
from qiskit_nature.second_q.mappers import JordanWignerMapper
from qiskit_nature.second_q.drivers import PySCFDriver
from qiskit.algorithms import VQE
from qiskit.algorithms.optimizers import SLSQP
from qiskit.primitives import Estimator

# Setup molecule
driver = PySCFDriver(atom='H 0 0 0; H 0 0 0.735', basis='sto3g')
problem = driver.run()

# Mapper
mapper = JordanWignerMapper()

# Ground state solver
optimizer = SLSQP(maxiter=100)
estimator = Estimator()

# Simple ansatz for demonstration
from qiskit.circuit.library import RealAmplitudes
ansatz = RealAmplitudes(num_qubits=4, reps=2)

vqe = VQE(estimator, ansatz, optimizer)
ground_solver = GroundStateEigensolver(mapper, vqe)

# Calculate ground state
ground_result = ground_solver.solve(problem)
print(f"Ground state energy: {ground_result.total_energies[0]:.6f} Ha")

# Excited states with QEOM
qeom = QEOM(ground_solver, 'sd')  # singles and doubles
excited_solver = ExcitedStatesEigensolver(mapper, qeom)

# Calculate excited states
excited_result = excited_solver.solve(problem)
print(f"\nExcited state energies:")
for i, energy in enumerate(excited_result.total_energies):
    print(f"State {i}: {energy:.6f} Ha")
```

### Molecular Potential Energy Surface

```python
from qiskit_nature.second_q.drivers import PySCFDriver
from qiskit_nature.second_q.mappers import JordanWignerMapper
from qiskit_nature.second_q.algorithms import GroundStateEigensolver
from qiskit.algorithms import VQE
from qiskit.algorithms.optimizers import SLSQP
from qiskit.primitives import Estimator
from qiskit.circuit.library import RealAmplitudes
import numpy as np

def calculate_pes(distances, molecule_template, basis='sto3g'):
    """
    Calculate potential energy surface for a diatomic molecule.
    
    Args:
        distances: Array of interatomic distances
        molecule_template: Molecule string with {} for distance
        basis: Basis set
    """
    energies = []
    
    for distance in distances:
        # Setup molecule at this distance
        atom_string = molecule_template.format(distance)
        driver = PySCFDriver(atom=atom_string, basis=basis)
        problem = driver.run()
        
        # Map to qubits
        mapper = JordanWignerMapper()
        
        # Simple VQE calculation
        ansatz = RealAmplitudes(num_qubits=4, reps=1)
        optimizer = SLSQP(maxiter=50)
        estimator = Estimator()
        
        vqe = VQE(estimator, ansatz, optimizer)
        solver = GroundStateEigensolver(mapper, vqe)
        
        # Calculate energy
        result = solver.solve(problem)
        energies.append(result.total_energies[0])
        
        print(f"Distance {distance:.2f} Å: Energy = {energies[-1]:.6f} Ha")
    
    return np.array(energies)

# Calculate PES for H2
distances = np.linspace(0.5, 2.5, 5)  # Angstroms
molecule_template = 'H 0 0 0; H 0 0 {}'

energies = calculate_pes(distances, molecule_template)

# Find equilibrium bond length
min_idx = np.argmin(energies)
print(f"\nEquilibrium bond length: {distances[min_idx]:.2f} Å")
print(f"Minimum energy: {energies[min_idx]:.6f} Ha")
```

## Quantum Simulation

### Hamiltonian Simulation with Trotter

```python
from qiskit import QuantumCircuit
from qiskit.quantum_info import SparsePauliOp, Statevector
from qiskit.synthesis import SuzukiTrotter
import numpy as np

def trotter_simulation(hamiltonian, time, n_steps):
    """
    Simulate time evolution under Hamiltonian using Trotter decomposition.
    
    Args:
        hamiltonian: SparsePauliOp Hamiltonian
        time: Total evolution time
        n_steps: Number of Trotter steps
    """
    dt = time / n_steps
    
    # Create Trotter circuit
    n_qubits = hamiltonian.num_qubits
    qc = QuantumCircuit(n_qubits)
    
    # Initial state (e.g., |0⟩)
    # Apply Trotter steps
    for _ in range(n_steps):
        # For each Pauli term in Hamiltonian
        for pauli, coeff in zip(hamiltonian.paulis, hamiltonian.coeffs):
            # Apply exp(-i * coeff * dt * pauli)
            angle = 2 * float(coeff.real) * dt
            
            # Simple implementation for single Pauli terms
            pauli_str = str(pauli)
            
            # Apply appropriate rotation
            if 'Z' in pauli_str and pauli_str.count('I') == n_qubits - 1:
                idx = pauli_str.index('Z')
                qc.rz(angle, n_qubits - 1 - idx)
            elif 'X' in pauli_str and pauli_str.count('I') == n_qubits - 1:
                idx = pauli_str.index('X')
                qc.rx(angle, n_qubits - 1 - idx)
    
    return qc

# Example: Ising model Hamiltonian
hamiltonian = SparsePauliOp(['ZZ', 'XI', 'IX'], coeffs=[1.0, -0.5, -0.5])

# Simulate for time t=1.0 with 10 Trotter steps
qc = trotter_simulation(hamiltonian, time=1.0, n_steps=10)

print(f"Trotter circuit depth: {qc.depth()}")
print(f"Gate count: {len(qc.data)}")

# Get final state
state = Statevector.from_instruction(qc)
print(f"Final state vector: {state[:4]}")  # First 4 amplitudes
```

### Variational Quantum Simulation

```python
from qiskit import QuantumCircuit
from qiskit.algorithms import VQE
from qiskit.algorithms.optimizers import SLSQP
from qiskit.circuit.library import RealAmplitudes
from qiskit.primitives import Estimator
from qiskit.quantum_info import SparsePauliOp
import numpy as np

def simulate_dynamics(hamiltonian, initial_state_circuit, times, ansatz_reps=2):
    """
    Simulate quantum dynamics using VQE at different times.
    
    Args:
        hamiltonian: Time-evolution Hamiltonian
        initial_state_circuit: Circuit preparing initial state
        times: Array of times to simulate
        ansatz_reps: Repetitions in ansatz
    """
    n_qubits = hamiltonian.num_qubits
    results = []
    
    for t in times:
        # Time-evolved Hamiltonian: H(t) = exp(-iHt) ρ(0) exp(iHt)
        # Approximate with VQE
        
        ansatz = RealAmplitudes(num_qubits=n_qubits, reps=ansatz_reps)
        
        # Combine initial state + ansatz
        full_circuit = initial_state_circuit.copy()
        full_circuit.compose(ansatz, inplace=True)
        
        optimizer = SLSQP(maxiter=100)
        estimator = Estimator()
        
        vqe = VQE(estimator, full_circuit, optimizer)
        result = vqe.compute_minimum_eigenvalue(hamiltonian)
        
        results.append({
            'time': t,
            'energy': result.eigenvalue,
            'parameters': result.optimal_parameters
        })
        
        print(f"Time {t:.2f}: Energy = {result.eigenvalue:.6f}")
    
    return results

# Example: Simulate Heisenberg model
hamiltonian = SparsePauliOp(
    ['XX', 'YY', 'ZZ'],
    coeffs=[0.5, 0.5, 0.5]
)

# Initial state: |01⟩
initial_state = QuantumCircuit(2)
initial_state.x(1)

# Simulate at different times
times = np.linspace(0, 2, 5)
results = simulate_dynamics(hamiltonian, initial_state, times)
```

## Quantum Machine Learning

### Quantum Kernel Method

```python
# Requires: pip install qiskit-machine-learning

from qiskit_machine_learning.kernels import FidelityQuantumKernel
from qiskit.circuit.library import ZZFeatureMap
from qiskit.primitives import Sampler
from sklearn.svm import SVC
import numpy as np

# Generate synthetic data
np.random.seed(42)
X_train = np.random.rand(20, 2) * 2 * np.pi
y_train = (X_train[:, 0] + X_train[:, 1] > np.pi).astype(int)

X_test = np.random.rand(10, 2) * 2 * np.pi
y_test = (X_test[:, 0] + X_test[:, 1] > np.pi).astype(int)

# Create feature map
feature_map = ZZFeatureMap(feature_dimension=2, reps=2)

# Create quantum kernel
sampler = Sampler()
quantum_kernel = FidelityQuantumKernel(feature_map=feature_map, fidelity=sampler)

# Compute kernel matrix
kernel_matrix_train = quantum_kernel.evaluate(x_vec=X_train)
kernel_matrix_test = quantum_kernel.evaluate(x_vec=X_test, y_vec=X_train)

print(f"Training kernel matrix shape: {kernel_matrix_train.shape}")
print(f"Test kernel matrix shape: {kernel_matrix_test.shape}")

# Use with SVM
svc = SVC(kernel='precomputed')
svc.fit(kernel_matrix_train, y_train)

# Predict
predictions = svc.predict(kernel_matrix_test)
accuracy = np.mean(predictions == y_test)

print(f"\nQuantum Kernel SVM Accuracy: {accuracy:.2%}")
```

### Variational Quantum Classifier (VQC)

```python
from qiskit_machine_learning.algorithms import VQC
from qiskit.circuit.library import RealAmplitudes, ZZFeatureMap
from qiskit.algorithms.optimizers import COBYLA
from qiskit.primitives import Sampler
import numpy as np

# Generate training data
np.random.seed(42)
X_train = np.random.rand(40, 2) * 2 - 1  # [-1, 1]
y_train = (X_train[:, 0]**2 + X_train[:, 1]**2 < 0.5).astype(int)

X_test = np.random.rand(20, 2) * 2 - 1
y_test = (X_test[:, 0]**2 + X_test[:, 1]**2 < 0.5).astype(int)

# Feature map
feature_map = ZZFeatureMap(2)

# Ansatz
ansatz = RealAmplitudes(2, reps=2)

# Create VQC
optimizer = COBYLA(maxiter=100)
sampler = Sampler()

vqc = VQC(
    sampler=sampler,
    feature_map=feature_map,
    ansatz=ansatz,
    optimizer=optimizer,
)

# Train
vqc.fit(X_train, y_train)

# Predict
train_score = vqc.score(X_train, y_train)
test_score = vqc.score(X_test, y_test)

print(f"Training accuracy: {train_score:.2%}")
print(f"Test accuracy: {test_score:.2%}")

# Predictions
predictions = vqc.predict(X_test)
print(f"Sample predictions: {predictions[:5]}")
print(f"True labels: {y_test[:5]}")
```

### Quantum Neural Network (QNN)

```python
from qiskit_machine_learning.neural_networks import SamplerQNN
from qiskit_machine_learning.algorithms import NeuralNetworkClassifier
from qiskit.circuit import QuantumCircuit, Parameter
from qiskit.circuit.library import RealAmplitudes
from qiskit.algorithms.optimizers import COBYLA
from qiskit.primitives import Sampler
import numpy as np

# Create parameterized quantum circuit
def create_qnn_circuit(num_qubits, num_features):
    qc = QuantumCircuit(num_qubits)
    
    # Feature map parameters
    feature_params = [Parameter(f'x{i}') for i in range(num_features)]
    
    # Encode features
    for i, param in enumerate(feature_params):
        if i < num_qubits:
            qc.ry(param, i)
    
    # Variational part
    ansatz = RealAmplitudes(num_qubits, reps=1)
    qc.compose(ansatz, inplace=True)
    
    return qc, feature_params, ansatz.parameters

num_qubits = 2
num_features = 2

qc, feature_params, weight_params = create_qnn_circuit(num_qubits, num_features)

# Create QNN
sampler = Sampler()
qnn = SamplerQNN(
    circuit=qc,
    input_params=feature_params,
    weight_params=weight_params,
    sampler=sampler,
)

# Create classifier
optimizer = COBYLA(maxiter=50)
classifier = NeuralNetworkClassifier(
    neural_network=qnn,
    optimizer=optimizer,
)

# Training data
X_train = np.random.rand(30, 2)
y_train = (X_train[:, 0] > X_train[:, 1]).astype(int)

# Train
classifier.fit(X_train, y_train)

# Test
X_test = np.random.rand(10, 2)
y_test = (X_test[:, 0] > X_test[:, 1]).astype(int)

score = classifier.score(X_test, y_test)
print(f"QNN Classifier accuracy: {score:.2%}")
```

## Noise Modeling and Error Mitigation

### Noise Models

```python
from qiskit.providers.aer import AerSimulator
from qiskit.providers.aer.noise import NoiseModel
from qiskit.providers.aer.noise import depolarizing_error, thermal_relaxation_error
from qiskit import QuantumCircuit
import numpy as np

# Create noise model
noise_model = NoiseModel()

# Depolarizing error on single-qubit gates
error_1q = depolarizing_error(0.001, 1)  # 0.1% error
noise_model.add_all_qubit_quantum_error(error_1q, ['u1', 'u2', 'u3', 'rx', 'ry', 'rz'])

# Depolarizing error on two-qubit gates
error_2q = depolarizing_error(0.01, 2)  # 1% error
noise_model.add_all_qubit_quantum_error(error_2q, ['cx', 'cz', 'swap'])

# Thermal relaxation error
# T1 = 50 microseconds, T2 = 70 microseconds, gate_time = 0.1 microseconds
t1 = 50e3  # ns
t2 = 70e3  # ns
gate_time = 100  # ns

error_thermal = thermal_relaxation_error(t1, t2, gate_time)
noise_model.add_all_qubit_quantum_error(error_thermal, ['id'])

# Measurement error
from qiskit.providers.aer.noise import ReadoutError
prob_meas0_prep1 = 0.05  # Prob of measuring 0 when prepared in 1
prob_meas1_prep0 = 0.10  # Prob of measuring 1 when prepared in 0

readout_error = ReadoutError([[1 - prob_meas1_prep0, prob_meas1_prep0],
                               [prob_meas0_prep1, 1 - prob_meas0_prep1]])
noise_model.add_all_qubit_readout_error(readout_error)

print(f"Noise model: {noise_model}")

# Run circuit with noise
qc = QuantumCircuit(2, 2)
qc.h(0)
qc.cx(0, 1)
qc.measure([0, 1], [0, 1])

# Noiseless simulation
simulator_ideal = AerSimulator()
job_ideal = simulator_ideal.run(qc, shots=1000)
counts_ideal = job_ideal.result().get_counts()

# Noisy simulation
simulator_noisy = AerSimulator(noise_model=noise_model)
job_noisy = simulator_noisy.run(qc, shots=1000)
counts_noisy = job_noisy.result().get_counts()

print(f"\nIdeal results: {counts_ideal}")
print(f"Noisy results: {counts_noisy}")
```

### Zero-Noise Extrapolation (ZNE)

```python
from qiskit import QuantumCircuit, transpile
from qiskit.providers.aer import AerSimulator
from qiskit.providers.aer.noise import NoiseModel, depolarizing_error
import numpy as np
from scipy.optimize import curve_fit

def run_circuit_with_noise(circuit, noise_level, shots=1000):
    """Run circuit with scaled noise."""
    # Create noise model
    noise_model = NoiseModel()
    error = depolarizing_error(noise_level, 1)
    noise_model.add_all_qubit_quantum_error(error, ['u1', 'u2', 'u3'])
    
    # Simulate
    simulator = AerSimulator(noise_model=noise_model)
    job = simulator.run(circuit, shots=shots)
    result = job.result()
    
    # Extract expectation value (simplified)
    counts = result.get_counts()
    expectation = sum(counts.get(bitstring, 0) * (-1)**bitstring.count('1') 
                     for bitstring in counts) / shots
    
    return expectation

def zero_noise_extrapolation(circuit, noise_levels, shots=1000):
    """
    Perform zero-noise extrapolation.
    
    Args:
        circuit: Quantum circuit
        noise_levels: List of noise scaling factors
        shots: Number of shots per simulation
    """
    expectations = []
    
    for noise in noise_levels:
        exp_val = run_circuit_with_noise(circuit, noise, shots)
        expectations.append(exp_val)
        print(f"Noise {noise:.4f}: Expectation = {exp_val:.4f}")
    
    # Fit exponential model: E(λ) = A + B * exp(-C * λ)
    def exp_model(x, a, b, c):
        return a + b * np.exp(-c * x)
    
    # Fit
    popt, _ = curve_fit(exp_model, noise_levels, expectations)
    
    # Extrapolate to zero noise
    zero_noise_value = exp_model(0, *popt)
    
    print(f"\nZero-noise extrapolated value: {zero_noise_value:.4f}")
    
    return zero_noise_value, expectations

# Example circuit
qc = QuantumCircuit(2, 2)
qc.h(0)
qc.cx(0, 1)
qc.measure([0, 1], [0, 1])

# Run ZNE
noise_levels = np.array([0.001, 0.005, 0.01, 0.02])
zne_value, measured_values = zero_noise_extrapolation(qc, noise_levels)
```

### Measurement Error Mitigation

```python
from qiskit import QuantumCircuit
from qiskit.providers.aer import AerSimulator
from qiskit.providers.aer.noise import NoiseModel, ReadoutError
from qiskit.result import marginal_counts
from qiskit.ignis.mitigation.measurement import (
    complete_meas_cal,
    CompleteMeasFitter
)

# Create readout noise
prob_meas0_prep1 = 0.1
prob_meas1_prep0 = 0.05

readout_error = ReadoutError([[1 - prob_meas1_prep0, prob_meas1_prep0],
                               [prob_meas0_prep1, 1 - prob_meas0_prep1]])

noise_model = NoiseModel()
noise_model.add_all_qubit_readout_error(readout_error)

# Generate calibration circuits
n_qubits = 2
meas_calibs, state_labels = complete_meas_cal(qr=list(range(n_qubits)))

# Run calibration
simulator = AerSimulator(noise_model=noise_model)
cal_results = []

for circuit in meas_calibs:
    job = simulator.run(circuit, shots=1000)
    cal_results.append(job.result())

# Fit the calibration
meas_fitter = CompleteMeasFitter(cal_results, state_labels)

# Get mitigation matrix
print("Calibration matrix:")
print(meas_fitter.cal_matrix)

# Run actual circuit
qc = QuantumCircuit(2, 2)
qc.h(0)
qc.cx(0, 1)
qc.measure([0, 1], [0, 1])

job = simulator.run(qc, shots=1000)
result = job.result()
noisy_counts = result.get_counts()

# Apply mitigation
mitigated_counts = meas_fitter.filter.apply(noisy_counts)

print(f"\nNoisy counts: {noisy_counts}")
print(f"Mitigated counts: {mitigated_counts}")
```

## Transpilation and Optimization

### Basic Transpilation

```python
from qiskit import QuantumCircuit, transpile
from qiskit.providers.fake_provider import FakeMontreal
from qiskit.visualization import plot_circuit_layout
import matplotlib.pyplot as plt

# Create circuit
qc = QuantumCircuit(3)
qc.h(0)
qc.cx(0, 1)
qc.cx(1, 2)
qc.cx(0, 2)

# Get fake backend (simulates real hardware)
backend = FakeMontreal()

# Transpile with different optimization levels
for level in range(4):
    transpiled = transpile(qc, backend=backend, optimization_level=level)
    
    print(f"\nOptimization level {level}:")
    print(f"  Depth: {transpiled.depth()}")
    print(f"  Gate count: {len(transpiled.data)}")
    print(f"  CNOT count: {transpiled.count_ops().get('cx', 0)}")

# Best transpilation
best_transpiled = transpile(qc, backend=backend, optimization_level=3)

print(f"\nOriginal circuit:")
print(f"  Depth: {qc.depth()}")
print(f"  Gates: {len(qc.data)}")

print(f"\nTranspiled circuit:")
print(f"  Depth: {best_transpiled.depth()}")
print(f"  Gates: {len(best_transpiled.data)}")
```

### Custom Transpilation Pass

```python
from qiskit import QuantumCircuit
from qiskit.transpiler import PassManager
from qiskit.transpiler.passes import Optimize1qGates, CommutativeCancellation
from qiskit.transpiler.passes import UnitarySynthesis, Unroll3qOrMore

# Create circuit with redundant gates
qc = QuantumCircuit(2)
qc.h(0)
qc.h(0)  # Redundant - cancels previous H
qc.x(1)
qc.x(1)  # Redundant - cancels previous X
qc.cx(0, 1)

print(f"Original circuit depth: {qc.depth()}")
print(f"Original gate count: {len(qc.data)}")

# Create custom pass manager
pm = PassManager()

# Add optimization passes
pm.append(Optimize1qGates())           # Optimize single-qubit gates
pm.append(CommutativeCancellation())   # Cancel commuting gates
pm.append(Unroll3qOrMore())           # Decompose 3+ qubit gates
pm.append(UnitarySynthesis())         # Synthesize unitaries

# Run pass manager
optimized_qc = pm.run(qc)

print(f"\nOptimized circuit depth: {optimized_qc.depth()}")
print(f"Optimized gate count: {len(optimized_qc.data)}")

print("\nOriginal circuit:")
print(qc.draw())

print("\nOptimized circuit:")
print(optimized_qc.draw())
```

### Layout and Routing

```python
from qiskit import QuantumCircuit, transpile
from qiskit.providers.fake_provider import FakeMontreal
from qiskit.transpiler import CouplingMap

# Create circuit that requires routing
qc = QuantumCircuit(5)
qc.h(0)
qc.cx(0, 4)  # Long-range connection
qc.cx(1, 3)
qc.cx(2, 4)

# Fake backend with specific topology
backend = FakeMontreal()
coupling_map = backend.configuration().coupling_map

print(f"Backend coupling map: {coupling_map[:10]}...")  # First 10 edges

# Transpile with layout awareness
transpiled = transpile(
    qc,
    backend=backend,
    optimization_level=3,
    seed_transpiler=42  # For reproducibility
)

# Get final layout
final_layout = transpiled.layout.final_index_layout()
print(f"\nFinal qubit layout: {final_layout}")

print(f"\nOriginal circuit:")
print(f"  Depth: {qc.depth()}")
print(f"  CNOT count: {qc.count_ops().get('cx', 0)}")

print(f"\nTranspiled circuit (with routing):")
print(f"  Depth: {transpiled.depth()}")
print(f"  CNOT count: {transpiled.count_ops().get('cx', 0)}")
print(f"  SWAP count: {transpiled.count_ops().get('swap', 0)}")
```

## Quantum Information Theory

### Entanglement Measures

```python
from qiskit.quantum_info import Statevector, DensityMatrix, partial_trace
from qiskit.quantum_info import entropy, entanglement_of_formation
from qiskit import QuantumCircuit
import numpy as np

# Create entangled state (Bell state)
qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)

state = Statevector.from_instruction(qc)
density_matrix = DensityMatrix(state)

print(f"Bell state: {state}")

# Von Neumann entropy (should be 0 for pure states)
entropy_full = entropy(density_matrix)
print(f"\nFull system entropy: {entropy_full:.6f}")

# Reduced density matrix (trace out qubit 1)
rho_0 = partial_trace(density_matrix, [1])
entropy_reduced = entropy(rho_0)
print(f"Reduced density matrix entropy: {entropy_reduced:.6f}")

# Entanglement of formation
# For Bell state, should be 1
eof = entanglement_of_formation(density_matrix)
print(f"Entanglement of formation: {eof:.6f}")

# Compare with separable state
qc_sep = QuantumCircuit(2)
qc_sep.h(0)
qc_sep.h(1)

state_sep = Statevector.from_instruction(qc_sep)
dm_sep = DensityMatrix(state_sep)

rho_0_sep = partial_trace(dm_sep, [1])
entropy_sep = entropy(rho_0_sep)
print(f"\nSeparable state reduced entropy: {entropy_sep:.6f}")
```

### Quantum Channels and Process Tomography

```python
from qiskit.quantum_info import Choi, SuperOp, Kraus
from qiskit import QuantumCircuit
import numpy as np

# Define a quantum channel (amplitude damping)
def amplitude_damping_channel(gamma):
    """
    Amplitude damping channel with damping parameter gamma.
    
    Models energy dissipation.
    """
    K0 = np.array([[1, 0], [0, np.sqrt(1 - gamma)]])
    K1 = np.array([[0, np.sqrt(gamma)], [0, 0]])
    
    return Kraus([K0, K1])

# Create channel
gamma = 0.3
channel = amplitude_damping_channel(gamma)

print(f"Amplitude damping channel (γ={gamma}):")
print(f"Number of Kraus operators: {len(channel)}")

# Convert to different representations
superop = SuperOp(channel)
choi = Choi(channel)

print(f"\nSuperoperator shape: {superop.dim}")
print(f"Choi matrix shape: {choi.dim}")

# Apply channel to a state
from qiskit.quantum_info import Statevector, DensityMatrix

# Initial state |1⟩
state = Statevector([0, 1])
dm = DensityMatrix(state)

# Apply channel
dm_evolved = dm.evolve(channel)

print(f"\nInitial state: {state}")
print(f"After channel: {dm_evolved}")

# Measure fidelity
from qiskit.quantum_info import state_fidelity
fidelity = state_fidelity(dm, dm_evolved)
print(f"Fidelity: {fidelity:.6f}")
```

### Quantum State Tomography

```python
from qiskit.quantum_info import Statevector, DensityMatrix
from qiskit import QuantumCircuit
from qiskit.providers.aer import AerSimulator
from qiskit.quantum_info.states import state_fidelity
import numpy as np

def perform_state_tomography(qc, shots=1000):
    """
    Simplified quantum state tomography.
    
    Measures in X, Y, Z bases to reconstruct density matrix.
    """
    simulator = AerSimulator()
    
    # Measurement circuits in different bases
    measurements = {
        'Z': QuantumCircuit(qc.num_qubits, qc.num_qubits),
        'X': QuantumCircuit(qc.num_qubits, qc.num_qubits),
        'Y': QuantumCircuit(qc.num_qubits, qc.num_qubits)
    }
    
    # X basis: apply H before measurement
    measurements['X'].h(range(qc.num_qubits))
    
    # Y basis: apply S†H before measurement
    measurements['Y'].sdg(range(qc.num_qubits))
    measurements['Y'].h(range(qc.num_qubits))
    
    # Measure all
    for basis_circ in measurements.values():
        basis_circ.measure(range(qc.num_qubits), range(qc.num_qubits))
    
    # Run measurements
    results = {}
    for basis, meas_circ in measurements.items():
        full_circ = qc.copy()
        full_circ.compose(meas_circ, inplace=True)
        
        job = simulator.run(full_circ, shots=shots)
        results[basis] = job.result().get_counts()
    
    print(f"Z-basis measurements: {results['Z']}")
    print(f"X-basis measurements: {results['X']}")
    print(f"Y-basis measurements: {results['Y']}")
    
    # Simplified reconstruction (for single qubit)
    if qc.num_qubits == 1:
        # Extract Pauli expectation values
        z_exp = (results['Z'].get('0', 0) - results['Z'].get('1', 0)) / shots
        x_exp = (results['X'].get('0', 0) - results['X'].get('1', 0)) / shots
        y_exp = (results['Y'].get('0', 0) - results['Y'].get('1', 0)) / shots
        
        # Reconstruct density matrix: ρ = (I + x⟨X⟩ + y⟨Y⟩ + z⟨Z⟩)/2
        rho = np.array([
            [0.5 + 0.5*z_exp, 0.5*x_exp - 0.5j*y_exp],
            [0.5*x_exp + 0.5j*y_exp, 0.5 - 0.5*z_exp]
        ])
        
        return DensityMatrix(rho)
    
    return None

# Test on a known state
qc = QuantumCircuit(1)
qc.ry(np.pi/3, 0)  # Rotate to create superposition

# True state
true_state = Statevector.from_instruction(qc)
true_dm = DensityMatrix(true_state)

# Tomography
reconstructed_dm = perform_state_tomography(qc, shots=10000)

if reconstructed_dm is not None:
    fidelity = state_fidelity(true_dm, reconstructed_dm)
    print(f"\nReconstruction fidelity: {fidelity:.6f}")
    print(f"\nTrue density matrix:\n{np.array(true_dm)}")
    print(f"\nReconstructed density matrix:\n{np.array(reconstructed_dm)}")
```

## Visualization

### Circuit Visualization

```python
from qiskit import QuantumCircuit
from qiskit.visualization import circuit_drawer
import matplotlib.pyplot as plt

# Create complex circuit
qc = QuantumCircuit(3, 3)
qc.h(0)
qc.cx(0, 1)
qc.cx(1, 2)
qc.barrier()
qc.measure([0, 1, 2], [0, 1, 2])

# Different drawing styles
print("Text representation:")
print(qc.draw(output='text'))

# Matplotlib style (most common)
print("\nMatplotlib style:")
qc.draw(output='mpl')
plt.show()

# LaTeX style (for publications)
# qc.draw(output='latex')

# With gate labels
qc_labeled = QuantumCircuit(2, 2)
qc_labeled.h(0)
qc_labeled.cx(0, 1)
qc_labeled.measure_all()

qc_labeled.draw(output='mpl', style={'name': 'bw'})
plt.show()
```

### Result Visualization

```python
from qiskit import QuantumCircuit
from qiskit.providers.aer import AerSimulator
from qiskit.visualization import plot_histogram, plot_bloch_multivector
from qiskit.quantum_info import Statevector
import matplotlib.pyplot as plt

# Run circuit
qc = QuantumCircuit(2, 2)
qc.h(0)
qc.cx(0, 1)
qc.measure([0, 1], [0, 1])

simulator = AerSimulator()
job = simulator.run(qc, shots=1000)
counts = job.result().get_counts()

# Histogram
plot_histogram(counts, title='Bell State Measurement')
plt.show()

# Multiple result comparison
qc2 = QuantumCircuit(2, 2)
qc2.h(0)
qc2.h(1)
qc2.measure([0, 1], [0, 1])

job2 = simulator.run(qc2, shots=1000)
counts2 = job2.result().get_counts()

plot_histogram([counts, counts2], 
               legend=['Bell State', 'Product State'],
               title='State Comparison')
plt.show()

# Bloch sphere visualization
qc_bloch = QuantumCircuit(1)
qc_bloch.ry(np.pi/4, 0)

state = Statevector.from_instruction(qc_bloch)
plot_bloch_multivector(state)
plt.show()
```

### State Visualization

```python
from qiskit.quantum_info import Statevector, DensityMatrix
from qiskit.visualization import plot_state_qsphere, plot_state_city
from qiskit.visualization import plot_state_hinton, plot_state_paulivec
import matplotlib.pyplot as plt

# Create an interesting state
from qiskit import QuantumCircuit
qc = QuantumCircuit(2)
qc.h(0)
qc.ry(np.pi/4, 1)
qc.cx(0, 1)

state = Statevector.from_instruction(qc)

# Q-sphere
plot_state_qsphere(state, title='Q-sphere')
plt.show()

# City plot
plot_state_city(state, title='State City')
plt.show()

# Hinton plot
dm = DensityMatrix(state)
plot_state_hinton(dm, title='Density Matrix')
plt.show()

# Pauli vector representation
plot_state_paulivec(state, title='Pauli Vector')
plt.show()
```

## Practical Workflows

### Complete VQE Workflow for Molecule

```python
from qiskit_nature.second_q.drivers import PySCFDriver
from qiskit_nature.second_q.mappers import JordanWignerMapper
from qiskit_nature.second_q.circuit.library import UCCSD, HartreeFock
from qiskit.algorithms import VQE
from qiskit.algorithms.optimizers import SLSQP
from qiskit.primitives import Estimator
import numpy as np

def calculate_molecule_energy(atom_string, basis='sto3g', charge=0, spin=0):
    """
    Complete workflow for molecular energy calculation.
    
    Args:
        atom_string: Atomic coordinates (e.g., 'H 0 0 0; H 0 0 0.735')
        basis: Basis set
        charge: Molecular charge
        spin: Spin multiplicity
    """
    print(f"Calculating energy for: {atom_string}")
    print(f"Basis: {basis}, Charge: {charge}, Spin: {spin}\n")
    
    # 1. Setup driver
    driver = PySCFDriver(
        atom=atom_string,
        basis=basis,
        charge=charge,
        spin=spin
    )
    
    # 2. Get electronic structure problem
    problem = driver.run()
    
    # 3. Setup mapper
    mapper = JordanWignerMapper()
    
    # 4. Get Hamiltonian
    hamiltonian = problem.hamiltonian.second_q_op()
    qubit_op = mapper.map(hamiltonian)
    
    print(f"Number of qubits: {qubit_op.num_qubits}")
    print(f"Number of Hamiltonian terms: {len(qubit_op)}")
    
    # 5. Setup initial state and ansatz
    num_particles = problem.num_particles
    num_spatial_orbitals = problem.num_spatial_orbitals
    
    init_state = HartreeFock(
        num_spatial_orbitals=num_spatial_orbitals,
        num_particles=num_particles,
        qubit_mapper=mapper
    )
    
    ansatz = UCCSD(
        num_spatial_orbitals=num_spatial_orbitals,
        num_particles=num_particles,
        qubit_mapper=mapper,
        initial_state=init_state
    )
    
    print(f"Ansatz circuit depth: {ansatz.depth()}")
    print(f"Number of parameters: {ansatz.num_parameters}\n")
    
    # 6. Run VQE
    optimizer = SLSQP(maxiter=100)
    estimator = Estimator()
    
    vqe = VQE(estimator, ansatz, optimizer)
    result = vqe.compute_minimum_eigenvalue(qubit_op)
    
    # 7. Extract results
    vqe_energy = result.eigenvalue
    hf_energy = problem.reference_energy
    
    print(f"Hartree-Fock energy: {hf_energy:.6f} Ha")
    print(f"VQE energy: {vqe_energy:.6f} Ha")
    print(f"Correlation energy: {vqe_energy - hf_energy:.6f} Ha")
    print(f"Optimizer evaluations: {result.cost_function_evals}")
    
    return {
        'vqe_energy': vqe_energy,
        'hf_energy': hf_energy,
        'correlation': vqe_energy - hf_energy,
        'optimal_parameters': result.optimal_parameters,
        'num_qubits': qubit_op.num_qubits
    }

# Example: H2 molecule
results = calculate_molecule_energy('H 0 0 0; H 0 0 0.735')
```

### Quantum Circuit Optimization Pipeline

```python
from qiskit import QuantumCircuit, transpile
from qiskit.providers.fake_provider import FakeMontreal
from qiskit.transpiler import PassManager, CouplingMap
from qiskit.transpiler.passes import *
import matplotlib.pyplot as plt

def optimize_circuit_for_hardware(qc, backend=None, optimization_level=3):
    """
    Complete circuit optimization pipeline.
    
    Args:
        qc: Input quantum circuit
        backend: Target backend (or None for generic optimization)
        optimization_level: 0-3
    """
    print(f"Original circuit:")
    print(f"  Qubits: {qc.num_qubits}")
    print(f"  Depth: {qc.depth()}")
    print(f"  Gate count: {len(qc.data)}")
    print(f"  Operations: {qc.count_ops()}\n")
    
    if backend is None:
        backend = FakeMontreal()
    
    # Standard transpilation
    transpiled = transpile(
        qc,
        backend=backend,
        optimization_level=optimization_level,
        seed_transpiler=42
    )
    
    print(f"After transpilation (level {optimization_level}):")
    print(f"  Depth: {transpiled.depth()}")
    print(f"  Gate count: {len(transpiled.data)}")
    print(f"  Operations: {transpiled.count_ops()}")
    print(f"  SWAP gates: {transpiled.count_ops().get('swap', 0)}\n")
    
    # Custom optimization passes
    pm = PassManager()
    
    # Add passes
    pm.append(Optimize1qGates())
    pm.append(CommutativeCancellation())
    pm.append(CXCancellation())
    
    further_optimized = pm.run(transpiled)
    
    print(f"After custom optimization:")
    print(f"  Depth: {further_optimized.depth()}")
    print(f"  Gate count: {len(further_optimized.data)}")
    print(f"  Operations: {further_optimized.count_ops()}\n")
    
    # Calculate reduction
    depth_reduction = (1 - further_optimized.depth() / qc.depth()) * 100
    gate_reduction = (1 - len(further_optimized.data) / len(qc.data)) * 100
    
    print(f"Optimization results:")
    print(f"  Depth reduction: {depth_reduction:.1f}%")
    print(f"  Gate count reduction: {gate_reduction:.1f}%")
    
    return further_optimized

# Example: Complex circuit
qc = QuantumCircuit(4)
qc.h(range(4))
for i in range(3):
    qc.cx(i, i+1)
qc.barrier()
for i in range(4):
    qc.ry(np.pi/4, i)
qc.barrier()
for i in range(2):
    qc.cx(i, i+2)

optimized = optimize_circuit_for_hardware(qc)
```

### Quantum Algorithm Benchmarking

```python
from qiskit import QuantumCircuit
from qiskit.algorithms import VQE, QAOA
from qiskit.algorithms.optimizers import SLSQP, COBYLA, SPSA
from qiskit.circuit.library import RealAmplitudes
from qiskit.primitives import Estimator
from qiskit.quantum_info import SparsePauliOp
import time
import numpy as np

def benchmark_vqe_optimizers(hamiltonian, ansatz, optimizers_dict, trials=3):
    """
    Benchmark different optimizers for VQE.
    
    Args:
        hamiltonian: Problem Hamiltonian
        ansatz: Variational ansatz
        optimizers_dict: Dict of optimizer name -> optimizer instance
        trials: Number of trials per optimizer
    """
    results = {}
    
    for opt_name, optimizer in optimizers_dict.items():
        print(f"\nBenchmarking {opt_name}...")
        
        trial_results = []
        
        for trial in range(trials):
            estimator = Estimator()
            vqe = VQE(estimator, ansatz, optimizer)
            
            start_time = time.time()
            result = vqe.compute_minimum_eigenvalue(hamiltonian)
            elapsed_time = time.time() - start_time
            
            trial_results.append({
                'energy': result.eigenvalue,
                'time': elapsed_time,
                'evals': result.cost_function_evals
            })
            
            print(f"  Trial {trial + 1}: E = {result.eigenvalue:.6f}, "
                  f"Time = {elapsed_time:.2f}s, Evals = {result.cost_function_evals}")
        
        # Aggregate results
        energies = [r['energy'] for r in trial_results]
        times = [r['time'] for r in trial_results]
        evals = [r['evals'] for r in trial_results]
        
        results[opt_name] = {
            'mean_energy': np.mean(energies),
            'std_energy': np.std(energies),
            'mean_time': np.mean(times),
            'mean_evals': np.mean(evals),
            'best_energy': min(energies)
        }
    
    # Print summary
    print("\n" + "="*60)
    print("BENCHMARK SUMMARY")
    print("="*60)
    print(f"{'Optimizer':<15} {'Best Energy':>12} {'Mean Time':>12} {'Mean Evals':>12}")
    print("-"*60)
    
    for opt_name, res in results.items():
        print(f"{opt_name:<15} {res['best_energy']:>12.6f} "
              f"{res['mean_time']:>12.2f}s {res['mean_evals']:>12.0f}")
    
    return results

# Example: Benchmark for simple Hamiltonian
hamiltonian = SparsePauliOp(['ZZ', 'XI', 'IX'], coeffs=[1.0, -0.5, -0.5])
ansatz = RealAmplitudes(num_qubits=2, reps=2)

optimizers = {
    'SLSQP': SLSQP(maxiter=100),
    'COBYLA': COBYLA(maxiter=100),
    'SPSA': SPSA(maxiter=100)
}

benchmark_results = benchmark_vqe_optimizers(hamiltonian, ansatz, optimizers, trials=3)
```

## Common Pitfalls and Solutions

### Memory Management for Large Circuits

```python
from qiskit import QuantumCircuit
from qiskit.providers.aer import AerSimulator
import numpy as np

# Problem: Simulating too many qubits
def bad_simulation():
    # DON'T DO THIS - will use ~2^25 * 16 bytes = 512 MB per state vector
    qc = QuantumCircuit(25)
    for i in range(25):
        qc.h(i)
    
    # This will be very slow or crash
    # simulator = AerSimulator(method='statevector')
    # result = simulator.run(qc).result()

# Solution 1: Use sampling instead of statevector
def good_simulation_sampling():
    qc = QuantumCircuit(25, 25)
    for i in range(25):
        qc.h(i)
    qc.measure_all()
    
    # Much more efficient - doesn't store full state vector
    simulator = AerSimulator(method='automatic')  # Chooses best method
    result = simulator.run(qc, shots=1000).result()
    counts = result.get_counts()
    
    return counts

# Solution 2: Use Matrix Product State for certain circuits
def good_simulation_mps():
    qc = QuantumCircuit(50)  # Can handle more qubits!
    
    # MPS works well for circuits with limited entanglement
    for i in range(49):
        qc.h(i)
        qc.cx(i, i+1)  # Nearest-neighbor only
    
    simulator = AerSimulator(method='matrix_product_state')
    qc.measure_all()
    result = simulator.run(qc, shots=1000).result()
    
    return result.get_counts()

# Solution 3: Reduce circuit size
def good_simulation_reduced():
    # Use fewer qubits or simpler circuits
    qc = QuantumCircuit(10)  # More manageable
    for i in range(10):
        qc.h(i)
    
    simulator = AerSimulator()
    qc.measure_all()
    result = simulator.run(qc, shots=1000).result()
    
    return result.get_counts()

# Test solutions
counts = good_simulation_sampling()
print(f"Sampling method: {len(counts)} unique outcomes")
```

### Handling Convergence Issues in VQE

```python
from qiskit.algorithms import VQE
from qiskit.algorithms.optimizers import SLSQP, COBYLA
from qiskit.circuit.library import RealAmplitudes
from qiskit.primitives import Estimator
from qiskit.quantum_info import SparsePauliOp
import numpy as np

def robust_vqe(hamiltonian, n_qubits, max_attempts=5):
    """
    VQE with multiple restart attempts and parameter initialization.
    
    Args:
        hamiltonian: Problem Hamiltonian
        n_qubits: Number of qubits
        max_attempts: Maximum restart attempts
    """
    best_result = None
    best_energy = float('inf')
    
    for attempt in range(max_attempts):
        print(f"\nAttempt {attempt + 1}/{max_attempts}")
        
        # Create fresh ansatz
        ansatz = RealAmplitudes(num_qubits=n_qubits, reps=2)
        
        # Initialize parameters randomly (but bounded)
        initial_point = np.random.uniform(-np.pi, np.pi, ansatz.num_parameters)
        
        # Try different optimizers
        if attempt < max_attempts // 2:
            optimizer = COBYLA(maxiter=200)
        else:
            optimizer = SLSQP(maxiter=200)
        
        estimator = Estimator()
        vqe = VQE(estimator, ansatz, optimizer, initial_point=initial_point)
        
        try:
            result = vqe.compute_minimum_eigenvalue(hamiltonian)
            
            print(f"  Energy: {result.eigenvalue:.6f}")
            print(f"  Evaluations: {result.cost_function_evals}")
            
            if result.eigenvalue < best_energy:
                best_energy = result.eigenvalue
                best_result = result
                print(f"  ✓ New best energy!")
            
        except Exception as e:
            print(f"  ✗ Failed: {e}")
            continue
    
    if best_result is None:
        raise RuntimeError("All VQE attempts failed")
    
    print(f"\nBest energy found: {best_energy:.6f}")
    return best_result

# Example usage
hamiltonian = SparsePauliOp(['ZZ', 'XI', 'IX'], coeffs=[1.0, -0.5, -0.5])
result = robust_vqe(hamiltonian, n_qubits=2, max_attempts=3)
```
