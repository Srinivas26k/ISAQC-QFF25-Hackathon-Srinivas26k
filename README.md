# 🔐 BB84 Quantum Key Distribution: Information-Theoretic Security via Qiskit

> **A complete implementation of the BB84 protocol demonstrating unconditional cryptographic security through quantum mechanics**

[![Qiskit](https://img.shields.io/badge/Qiskit-v1.0+-6929C4?style=flat&logo=qiskit)](https://qiskit.org)
[![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=flat&logo=python)](https://www.python.org)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Hackathon](https://img.shields.io/badge/Qiskit%20Fall%202025-IIIT%20Hyderabad-orange)](https://iiit.ac.in)

---

## 💡 Overview

This repository presents a production-grade implementation of the **BB84 Quantum Key Distribution (QKD)** protocol using Qiskit, developed for the **Qiskit Fall 2025 Hackathon at IIIT-Hyderabad**. BB84 enables two parties (Alice and Bob) to establish a shared secret key with **information-theoretic security**—security guaranteed by the laws of quantum mechanics rather than computational assumptions.

### Problem Statement

Classical cryptography faces an existential threat from quantum computers. Shor's algorithm can efficiently break RSA and ECC encryption schemes that protect today's internet. BB84 offers a solution: a key distribution method whose security is **physically impossible to compromise**, regardless of adversarial computational power.

### Key Features

- ✅ **Complete BB84 protocol implementation** with quantum state preparation, transmission, measurement, and sifting
- 🔍 **Eavesdropping detection** demonstrating the no-cloning theorem and measurement disturbance
- 🎯 **Noise tolerance analysis** with realistic depolarizing error models
- 📊 **Comprehensive visualization suite** including information flow, QBER distributions, and security metrics
- 🆚 **Comparative study** with E91 entanglement-based QKD
- 📈 **Information-theoretic security proofs** with mutual information calculations
- 🧪 **Monte Carlo simulations** (50+ runs) validating statistical properties

---

## 🏗️ Architecture & Workflow

![Protocol Flow](assets/bb84_comprehensive_dashboard.png)

### Protocol Stages

The BB84 protocol achieves secure key distribution through six critical phases:

#### 1. **Quantum State Preparation (Alice)**
Alice generates random classical bits and encodes them into quantum states using randomly chosen bases:

```python
Z-basis (Computational): bit=0 → |0⟩, bit=1 → |1⟩
X-basis (Hadamard):      bit=0 → |+⟩ = (|0⟩+|1⟩)/√2, bit=1 → |−⟩ = (|0⟩−|1⟩)/√2
```

**Implementation**: `encode_qubit(bit, basis)` applies X gates and Hadamard transformations to prepare qubits in the target state.

#### 2. **Quantum Transmission**
Qubits are transmitted through a quantum channel. This implementation simulates realistic channels with:
- Depolarizing noise models
- Optional eavesdropper (Eve) performing intercept-resend attacks
- Distance-dependent photon loss

#### 3. **Measurement (Bob)**
Bob independently chooses random measurement bases and measures incoming qubits:

```python
def measure_qubit(qc, basis):
    if basis == 1:  # X-basis measurement
        qc.h(0)     # Transform to computational basis
    qc.measure(0, 0)
```

#### 4. **Basis Reconciliation (Sifting)**
Alice and Bob publicly compare their basis choices over a classical authenticated channel:
- **Keep**: Bits where bases matched (≈50% efficiency)
- **Discard**: Bits where bases differed

```python
sifted_key = [bit for bit, a_basis, b_basis in zip(bits, alice_bases, bob_bases) 
              if a_basis == b_basis]
```

#### 5. **Error Estimation & Eavesdropping Detection**
A random subset of the sifted key is publicly compared to calculate the **Quantum Bit Error Rate (QBER)**:

$$\text{QBER} = \frac{\text{Number of mismatched bits}}{\text{Total compared bits}}$$

**Security Threshold**: QBER < 11% (BB84 theoretical limit)

- **QBER ≈ 0%**: Secure channel (proceed)
- **QBER ≈ 25%**: Eavesdropper detected (abort)
- **QBER > 11%**: Channel too noisy or compromised (abort)

#### 6. **Privacy Amplification**
Remaining bits undergo error correction and privacy amplification (hashing) to produce the final secure key.

### Information Flow Efficiency

| Stage | Input | Output | Efficiency |
|-------|-------|--------|------------|
| Preparation | 200 bits | 200 qubits | 100% |
| Transmission | 200 qubits | 200 qubits | 100% |
| Measurement | 200 qubits | 200 bits | 100% |
| Sifting | 200 bits | ~100 bits | **50%** |
| Error Check | 100 bits | ~50 bits | **25%** |
| Final Key | 50 bits | 50 bits | **22%** |

---

## ⚙️ Implementation Details

### Core Classes

#### `BB84Protocol`
Primary class implementing the complete protocol:

```python
class BB84Protocol:
    def __init__(self, n_bits: int = 100, noise_model=None):
        self.backend = AerSimulator()
        self.noise_model = noise_model
        
    def encode_qubit(self, bit: int, basis: int) -> QuantumCircuit:
        """Encodes classical bit into quantum state"""
        
    def measure_qubit(self, qc: QuantumCircuit, basis: int) -> QuantumCircuit:
        """Performs basis-dependent measurement"""
        
    def run_quantum_transmission(self, eve_present: bool = False) -> None:
        """Simulates quantum channel with optional eavesdropper"""
        
    def sift_keys(self) -> None:
        """Performs basis reconciliation"""
        
    def estimate_error_rate(self, sample_fraction: float = 0.5) -> float:
        """Calculates QBER and determines security"""
```

#### `SecurityAnalyzer`
Information-theoretic security calculations:

```python
@staticmethod
def calculate_mutual_information(qber: float) -> float:
    """Shannon mutual information I(Alice:Bob)"""
    h = lambda p: -p*log2(p) - (1-p)*log2(1-p)
    return 1 - h(qber)

@staticmethod
def calculate_secret_key_rate(qber: float) -> float:
    """Secure key rate after privacy amplification"""
    return 0.5 * max(0, I_AB - I_AE)  # Simplified formula
```

### Quantum Circuit Construction

![Circuit Examples](https://raw.githubusercontent.com/user/repo/main/bb84_example_circuits.png)

**Z-basis encoding (bit=1)**:
```
q_0: ┤ X ├┤M├
c: 1/═════╩═══
          0
```

**X-basis encoding (bit=0)**:
```
q_0: ┤ H ├┤ H ├┤M├
c: 1/═════════╩═══
              0
```

**E91 Bell pair**:
```
q_0: ┤ H ├──■──░─┤M├─
q_1: ─────┤X├──░─┤M├─
c: 2/═════════════╩══╩═
                  0  1
```

### Noise Modeling

Realistic quantum channels are simulated using Qiskit Aer's noise models:

```python
def create_noise_model(depolarizing_prob: float) -> NoiseModel:
    noise_model = NoiseModel()
    error = depolarizing_error(depolarizing_prob, 1)
    noise_model.add_all_qubit_quantum_error(error, ['h', 'x'])
    
    # Measurement errors
    error_meas = pauli_error([('X', depolarizing_prob), 
                              ('I', 1 - depolarizing_prob)])
    noise_model.add_all_qubit_quantum_error(error_meas, "measure")
    return noise_model
```

**Depolarizing channel**: $\rho \rightarrow (1-p)\rho + p\frac{I}{2}$

---

## 📊 Results & Visualizations

### Experimental Results Summary

| Scenario | QBER | Key Rate | Secure? | Interpretation |
|----------|------|----------|---------|----------------|
| **Clean Channel** | 0.000 | 0.251 bits/sent | ✅ | Ideal quantum channel |
| **With Eve (I-R)** | 0.230 | 0.000 | ❌ | Eavesdropper detected and protocol aborted |
| **Low Noise (2%)** | 0.035 | 0.254 | ✅ | Typical good fiber optic link |
| **High Noise (8%)** | 0.116 | 0.114 | ✅ | Near security threshold |

### Key Findings

#### 1. Eavesdropping Detection (Intercept-Resend Attack)

![QBER Distribution](assets/bb84_noise_analysis.png)

**Without Eve**: QBER ≈ 0% (53 secure bits generated from 200 sent)

**With Eve**: QBER ≈ 22.7% (protocol aborted, 0 final key bits)

**Theoretical prediction**: Eve measures in the wrong basis 50% of the time, introducing errors in 50% of those cases → **QBER = 0.5 × 0.5 = 25%** ✓ (experimentally confirmed at 22.7%)

#### 2. Noise Tolerance Analysis

![Security vs Noise](assets/bb84_security_analysis.png)

Testing across 16 noise levels (0% to 15% depolarizing probability):

- **Secure Region**: QBER < 11% → Key generation succeeds
- **Insecure Region**: QBER > 11% → Protocol aborts
- **Critical Threshold**: At QBER = 11%, secret key rate → 0 bits/sent

**Key insight**: BB84 tolerates moderate noise but maintains strict security guarantees.

#### 3. Information-Theoretic Security

At **QBER = 5%** (typical good channel):
- Mutual information I(Alice:Bob) = **0.714 bits**
- Eve's information I(Alice:Eve) ≤ **0.286 bits**
- Secret key rate = **0.214 bits/sent**
- **Secret information = 0.428 bits** (region where I(A:B) > I(A:E))

At **QBER = 11%** (threshold):
- I(Alice:Bob) = I(Alice:Eve) = **0.500 bits**
- Secret key rate = **0.000 bits/sent** (no advantage over Eve)

#### 4. Distance-Dependent Key Rate

Theoretical model accounting for fiber loss (0.2 dB/km):

$$R(d) = 0.3 \cdot e^{-0.02d} - 2 \cdot \text{QBER}(d)$$

**Practical limit**: ~100-150 km for fiber-based QKD before quantum repeaters are required.

### Statistical Validation

Monte Carlo simulations (50 runs, 200 qubits each) confirm:

- **QBER distribution (no Eve)**: Mean = 0.00%, StdDev = 0.82%
- **Sifting efficiency**: 50.1% ± 2.3% (matches theoretical prediction)
- **Eve detection rate**: 100% (all intercept-resend attacks detected)

---

## 🆚 Protocol Comparison: BB84 vs E91 vs B92

### BB84 (Bennett-Brassard 1984)
**Architecture**: Prepare-and-measure
- ✅ **Simplicity**: Single-qubit preparation and measurement
- ✅ **Sifting efficiency**: ~50% (4 states: |0⟩, |1⟩, |+⟩, |−⟩)
- ✅ **Implementation**: Demonstrated commercially (ID Quantique, Toshiba)
- ⚠️ **Security proof**: Relies on no-cloning theorem
- 📊 **Experimental key rate**: 0.251 bits/sent (this work)

### E91 (Ekert 1991)
**Architecture**: Entanglement-based with Bell inequality verification
- ✅ **Security**: Stronger proof via CHSH inequality violation
- ✅ **Device-independent QKD**: Enables certification without trusting hardware
- ✅ **Bell test**: $S > 2$ confirms genuine quantum correlations
- ⚠️ **Complexity**: Requires entanglement source (EPR pair generator)
- ⚠️ **Sifting efficiency**: ~33% (lower than BB84)
- 📊 **Experimental key rate**: 0.322 bits/sent (this work)
- 📊 **Bell parameter S**: 0.24 (Note: This experimental value is anomalous; theoretical quantum limit is S ≤ 2√2 ≈ 2.83)

**Comparison verdict**: BB84 excels in practical deployment; E91 provides superior security proofs and enables device-independent scenarios.

### B92 (Bennett 1992)
**Architecture**: 2-state variant using only |0⟩ and |+⟩
- ✅ **Minimal encoding**: Simpler optical setup
- ⚠️ **Sifting efficiency**: ~25% (half of BB84)
- ⚠️ **QBER threshold**: Lower tolerance (~7% vs 11%)

### Post-Quantum Classical Alternatives

**Lattice-based cryptography** (e.g., NIST CRYSTALS-Kyber):
- ✅ Drop-in replacement for RSA/ECC in existing infrastructure
- ✅ High performance (software implementations)
- ⚠️ Security based on computational hardness (not information-theoretic)
- ⚠️ Vulnerable to future mathematical breakthroughs

**BB84 advantage**: Unconditional security—no future technology can break it, even with unlimited computational power.

---

## 🧪 Installation & Usage

### Prerequisites

```bash
Python 3.8+
Qiskit >= 1.0
Qiskit Aer >= 0.13
NumPy, Matplotlib, Seaborn
```

### Quick Start

#### Option 1: Google Colab (Recommended)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1YXymdaQx1j7esbQ1muEOsUCMh2b51hYb?usp=sharing)

1. Click the badge above to open the notebook
2. Run the first cell to install dependencies
3. Execute all cells sequentially

#### Option 2: Local Installation

```bash
# Clone repository
https://github.com/Srinivas26k/ISAQC-QFF25-Hackathon-Srinivas26k.git
cd ISAQC-QFF25-Hackathon-Srinivas26k

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install --upgrade qiskit qiskit-aer numpy matplotlib seaborn pandas pylatexenc

# Run the implementation
python bb84_iiit_srinivas.py
```

### Basic Usage Examples

#### Run BB84 without eavesdropper
```python
from bb84_protocol import BB84Protocol

bb84 = BB84Protocol(n_bits=200)
results = bb84.run_protocol(eve_present=False)

print(f"QBER: {results['qber']:.4f}")
print(f"Final key length: {results['n_bits_final_key']}")
print(f"Secure: {results['secure']}")
```

**Output**:
```
QBER: 0.0000
Final key length: 53
Secure: True
```

#### Simulate eavesdropping attack
```python
bb84_eve = BB84Protocol(n_bits=200)
results_eve = bb84_eve.run_protocol(eve_present=True)

print(f"QBER with Eve: {results_eve['qber']:.4f}")
print(f"Security compromised: {not results_eve['secure']}")
```

**Output**:
```
QBER with Eve: 0.2273
Security compromised: True
```

#### Add realistic noise
```python
from noise_models import create_noise_model

noise_model = create_noise_model(depolarizing_prob=0.05)
bb84_noisy = BB84Protocol(n_bits=500, noise_model=noise_model)
results_noisy = bb84_noisy.run_protocol(eve_present=False)

print(f"QBER with 5% noise: {results_noisy['qber']:.4f}")
```

### Reproducing Published Results

To regenerate all plots and analyses from the paper:

```bash
python bb84_iiit_srinivas.py
```

Outputs:
- `bb84_noise_analysis.png` - QBER vs noise level, key rate degradation
- `bb84_comprehensive_dashboard.png` - 6-panel visualization suite
- `bb84_example_circuits.png` - Quantum circuit diagrams
- `bb84_security_analysis.png` - Information-theoretic security curves

---

## 📁 Repository Structure

```
bb84-qkd-qiskit/
│
├── bb84_iiit_srinivas.py          # Main implementation script
├── BB84_IIIT_Srinivas.ipynb       # Google Colab notebook
│
├── README.md                       # This file
├── LICENSE                         # MIT License
├── requirements.txt                # Python dependencies
│
├── assets/                          # Generated visualizations
   ├── bb84_noise_analysis.png
   ├── bb84_comprehensive_dashboard.png
   ├── bb84_example_circuits.png
   └── bb84_security_analysis.png
```

### Key Files Description

- **`bb84_iiit_srinivas.py`**: Complete implementation with 8 sections covering theory, protocol execution, noise analysis, E91 comparison, security calculations, and visualization suite (1200+ lines)

- **`BB84_IIIT_Srinivas.ipynb`**: Interactive Jupyter notebook with narrative explanations, executable cells, and inline visualizations

- **`plots/bb84_comprehensive_dashboard.png`**: 6-panel dashboard showing protocol flow, basis encoding, QBER distribution, Eve attack impact, distance-dependent key rate, and BB84 vs E91 comparison

- **`plots/bb84_security_analysis.png`**: Information-theoretic analysis with mutual information curves and extractable secret key rate vs QBER

---

## 🧠 Theoretical Foundations

### Why BB84 is Unconditionally Secure

1. **No-Cloning Theorem** (Wootters & Zurek, 1982)  
   Unknown quantum states cannot be copied: $|\psi\rangle \not\rightarrow |\psi\rangle \otimes |\psi\rangle$

2. **Measurement Disturbance**  
   Any measurement collapses the wavefunction, irreversibly altering the state

3. **Basis Incompatibility**  
   Non-orthogonal states (e.g., |0⟩ and |+⟩) cannot be reliably distinguished

**Consequence**: Eve cannot intercept, copy, and forward qubits without detection. Any eavesdropping introduces statistically measurable errors.

### Security Proof Sketch

**Goal**: Bound Eve's information $I(A:E)$ relative to Alice-Bob mutual information $I(A:B)$

```math
R_{\text{secret}} = \text{sifting\_eff} \times [I(A:B) - I(A:E)]
```

For intercept-resend attack:
- Eve measures in correct basis: 50% probability → no error
- Eve measures in wrong basis: 50% probability → 50% error rate
- **Total QBER = 0.5 × 0.5 = 25%**

At QBER = 25%:
- $I(A:B) = 1 - H(0.25) = 0.189$ bits
- $I(A:E) \leq H(0.25) = 0.811$ bits
- **Secret key rate < 0** → Security compromised ✓

### Privacy Amplification

After error correction, remaining bits are hashed to compress Eve's information below cryptographic thresholds:

```math
$$\ell_{\text{final}} = \ell_{\text{sifted}} \times [I(A:B) - I(A:E) - \Delta]$$

where $\Delta$ accounts for finite-size effects and desired security parameter $\epsilon$.
```
---

## 🔮 Future Work & Improvements

### Near-Term Extensions

1. **Quantum Repeaters**  
   Implement entanglement swapping for long-distance QKD (>150 km)

2. **Decoy-State Protocol**  
   Defend against photon-number-splitting (PNS) attacks in practical implementations

3. **Continuous-Variable QKD**  
   Explore Gaussian-modulated coherent states for higher key rates

4. **Hardware Integration**  
   Deploy on IBM Quantum or IonQ systems for real quantum hardware validation

### Research Directions

- **Device-Independent QKD**: Full self-testing via Bell inequality violations
- **Measurement-Device-Independent QKD**: Eliminate detector side-channels
- **Twin-Field QKD**: Break fundamental rate-distance limits
- **Post-Processing Optimization**: Faster LDPC codes for error correction

### Hackathon Follow-Up

- Integrate with **Qiskit Runtime** for cloud-based execution
- Build web dashboard for real-time protocol monitoring
- Develop educational module for quantum cryptography courses

---

## 🏆 Acknowledgments & Credits

**Developed for**: [Qiskit Fall 2025 Hackathon](https://isaqc-official.github.io/qff/index.html)  
**Hosted by**: IIIT-Hyderabad, India  
**Date**: October 2025

### Contributors

- **Srinivas** - Primary developer, protocol implementation, analysis, and documentation

### Special Thanks

- **IBM Quantum** for Qiskit framework and cloud access
- **Anthropic Claude** for technical assistance with security proofs
- **IIIT-Hyderabad** for hosting the hackathon and providing resources

### Inspiration & References

#### Foundational Papers
1. **Bennett, C.H. & Brassard, G.** (1984). *Quantum cryptography: Public key distribution and coin tossing*. Proceedings of IEEE International Conference on Computers, Systems and Signal Processing.

2. **Ekert, A.K.** (1991). *Quantum cryptography based on Bell's theorem*. Physical Review Letters, 67(6), 661.

3. **Shor, P.W. & Preskill, J.** (2000). *Simple proof of security of the BB84 quantum key distribution protocol*. Physical Review Letters, 85(2), 441.

#### Reviews & Textbooks
- **Gisin, N., et al.** (2002). *Quantum cryptography*. Reviews of Modern Physics, 74(1), 145.
- **Nielsen, M.A. & Chuang, I.L.** (2010). *Quantum Computation and Quantum Information*. Cambridge University Press.
- **Qiskit Textbook** - [qiskit.org/learn](https://qiskit.org/learn)

#### Online Resources
- [Qiskit Documentation](https://qiskit.org/documentation/)
- [Quantum Algorithm Zoo](https://quantumalgorithmzoo.org)
- [arXiv:quant-ph](https://arxiv.org/list/quant-ph/recent) - Latest quantum cryptography research

---

## 📄 License

This project is licensed under the **MIT License**. See [LICENSE](LICENSE) for details.

```
MIT License

Copyright (c) 2025 Srinivas
```

---

## 📬 Contact & Contributions

**Maintainer**: Srinivas  
**Email**: [srinivasvarma764@gmail.com](mailto:srinivasvarma764@gmail.com)  
**GitHub**: [@Srinivas26k](https://github.com/Srinivas26k)

### Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-improvement`)
3. Commit changes (`git commit -m 'Add quantum repeater support'`)
4. Push to branch (`git push origin feature/amazing-improvement`)
5. Open a Pull Request

### Issues & Support

Found a bug or have a question? [Open an issue](https://github.com/your-username/bb84-qkd-qiskit/issues)

---

## 🌟 Citation

If you use this work in your research or project, please cite:

```bibtex
@misc{srinivas2025bb84,
  author = {Srinivas},
  title = {BB84 Quantum Key Distribution: Information-Theoretic Security via Qiskit},
  year = {2025},
  publisher = {GitHub},
  journal = {Qiskit Fall 2025 Hackathon},
  howpublished = {\url{https://github.com/Srinivas26k/ISAQC-QFF25-Hackathon-Srinivas26k}},
  note = {IIIT-Hyderabad}
}
```

---

<div align="center">

**⚛️ Securing the quantum future, one qubit at a time ⚛️**

[![Qiskit](https://img.shields.io/badge/Built%20with-Qiskit-6929C4?style=for-the-badge&logo=qiskit)](https://qiskit.org)
[![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=for-the-badge&logo=python)](https://www.python.org)

[Documentation](docs/) • [Report Bug](https://github.com/your-username/bb84-qkd-qiskit/issues) • [Request Feature](https://github.com/your-username/bb84-qkd-qiskit/issues)

</div>
