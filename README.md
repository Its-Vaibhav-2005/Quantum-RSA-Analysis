# Quantum RSA Analysis

## Overview

This project investigates the potential impact of quantum computing on the security of RSA encryption, particularly focusing on RSA-2048. Recent advancements suggest that quantum computers could pose a significant threat to current cryptographic standards sooner than previously anticipated.

---
> **Important Note**: Recent research highlights the rapidly advancing capabilities of quantum computers. While **IBM** continues to be a leading force in quantum computing hardware and software development, actively pushing the boundaries of qubit count and coherence, a notable study by **Craig Gidney** of the **Google Quantum AI** team indicates that a quantum computer with fewer than a million noisy qubits could potentially factor a 2048-bit RSA modulus in under a week. This research, irrespective of its origin, underscores the urgent need for the development and adoption of quantum-safe cryptographic solutions, a field both IBM and Google are heavily invested in. [Source](https://www.computerweekly.com/news/366625113/Noisy-quantum-hardware-could-crack-RSA-2048-in-seven-days)

## Project Scope

This research project aims to:

* **Assess Current Quantum Computing Capabilities:** Analyze the state-of-the-art in quantum hardware and software development.
* **Explore Quantum Algorithms for Cryptanalysis:** Investigate algorithms like Shor's algorithm that could undermine current public-key cryptography.
* **Evaluate Implications for Existing Cryptographic Systems:** Examine the vulnerabilities of widely used algorithms, such as RSA, in the face of quantum threats.
* **Demonstrate Basic Quantum Computing Concepts:** Utilize the Qiskit framework to illustrate fundamental principles of quantum computation.

## Current Implementation

The project currently encompasses:

* A foundational quantum computing environment established using Qiskit.
* An analysis of contemporary quantum computing capabilities and their limitations.
* An exploration of quantum algorithms with direct relevance to breaking RSA encryption.

## Dependencies

The project relies on the following Qiskit components and their specified versions:

* `qiskit-terra`: 0.21.0
* `qiskit-aer`: 0.10.4
* `qiskit-ibmq-provider`: 0.19.2
* `qiskit`: 0.37.0

## Installation

To set up the project locally, follow these steps:

1.  **Clone the repository:**
    ```bash
    git clone [repository-url]
    cd [repository-name]
    ```

2.  **Create a virtual environment (highly recommended):**
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Linux/macOS
    # or
    .\venv\Scripts\activate  # On Windows
    ```

3.  **Install the required dependencies:**
    ```bash
    pip install qiskit==0.37.0
    pip install qiskit-terra==0.21.0
    pip install qiskit-aer==0.10.4
    pip install qiskit-ibmq-provider==0.19.2
    ```

## Quantum Computing Implications

The ongoing development of quantum computers presents significant challenges to the security of current cryptographic systems:

* **RSA-2048 Vulnerability:** RSA-2048, a cornerstone of secure web communications and financial transactions, faces a credible threat from sufficiently advanced quantum computers.
* **Urgency for Quantum-Safe Cryptography:** The potential for quantum-powered attacks necessitates an immediate shift towards quantum-safe (or post-quantum) cryptographic standards.
* **Migration imperative:** Organizations are strongly advised to begin strategizing and implementing migration plans to post-quantum cryptography to mitigate future risks.

## RSA Implementation Details

This project includes a basic implementation of the RSA algorithm to demonstrate its core functionalities.

### Key Generation

The RSA key generation process involves:

1.  **Prime Number Generation:**
    ```python
    def genPrime(bits):
        while True:
            num = random.getrandbits(bits)
            num |= (1 << bits - 1) | 1  # Ensure high bit is set and it's odd
            if isprime(num):
                return num
    ```
    This function generates cryptographically secure prime numbers of a specified bit length. It ensures the number is odd and its highest bit is set for proper length.

2.  **Key Pair Generation:**
    ```python
    def genKeyPairs(keysize):
        half = keysize // 2
        p = genPrime(half)
        q = genPrime(half)
        n = p * q
        phi = (p - 1) * (q - 1)
        e = 65537  # Standard public exponent
        d = modinv(e, phi)
        return (e, n), (d, n)
    ```
    This function generates the RSA key pair:
    * It generates two large, distinct prime numbers, $p$ and $q$, each approximately half the `keysize`.
    * It computes the modulus $n = p \times q$.
    * It calculates Euler's totient function $\phi(n) = (p-1)(q-1)$.
    * It uses a standard public exponent $e = 65537$.
    * It computes the private exponent $d$ as the modular multiplicative inverse of $e$ modulo $\phi(n)$.

### Encryption and Decryption

```python
def encrypt(public_key, plaintext):
    e, n = public_key
    plaintextInt = int.from_bytes(plaintext.encode(), byteorder='big')
    return pow(plaintextInt, e, n)

def decrypt(private_key, ciphertext_int):
    d, n = private_key
    decryptedInt = pow(ciphertext_int, d, n)
    return decryptedInt.to_bytes((decryptedInt.bit_length() + 7) // 8, byteorder='big')
```
* **Encryption:** The `encrypt` function takes the public key $(e, n)$ and the plaintext. It converts the plaintext to an integer and computes the ciphertext as $C = M^e \pmod{n}$.
* **Decryption:** The `decrypt` function takes the private key $(d, n)$ and the ciphertext. It computes the decrypted integer as $M = C^d \pmod{n}$ and converts it back to bytes.

## Shor's Algorithm and RSA Breaking

### How Shor's Algorithm Breaks RSA

1.  **Problem Reduction:**
    The security of RSA hinges on the computational difficulty of factoring large composite numbers $N = p \times q$. Shor's algorithm transforms this factoring problem into a period-finding problem, which can be efficiently solved by a quantum computer.

2.  **Quantum Steps (Conceptual):**
    ```python
    from qiskit.algorithms import Shor
    
    # Initialize quantum instance (e.g., using a simulator)
    quantum_instance = QuantumInstance(Aer.get_backend('aer_simulator'))
    shor = Shor(quantum_instance=quantum_instance)
    
    # Factor the number N (demonstration for small N)
    result = shor.factor(N)
    p, q = result.factors[0]
    ```
    This snippet conceptually shows how Shor's algorithm could be invoked using Qiskit. In practice, factoring large numbers like RSA-2048's modulus is not yet feasible on current quantum hardware.

3.  **Algorithm Steps:**
    Shor's algorithm consists of both classical and quantum components:

    a.  **Classical Preprocessing:**
        * Choose a random integer $a$ such that $1 < a < N$.
        * Calculate $\text{gcd}(a, N)$. If $\text{gcd}(a, N) > 1$, then a non-trivial factor of $N$ has been found, and the algorithm terminates.

    b.  **Quantum Phase Estimation (Core of Shor's):**
        * Prepare a quantum register in a superposition of states.
        * Apply a quantum oracle that implements the modular exponentiation function $f(x) = a^x \pmod{N}$. This function is periodic.
        * Apply a Quantum Fourier Transform (QFT) to measure the period $r$ of $f(x)$.

    c.  **Classical Post-processing:**
        * If the period $r$ is found (from the quantum measurement), check if $r$ is even. If $r$ is odd, or if $a^{r/2} \equiv -1 \pmod{N}$, select a different random $a$ and repeat the process.
        * Otherwise, compute $\text{gcd}(a^{r/2} - 1, N)$ and $\text{gcd}(a^{r/2} + 1, N)$. These calculations will yield the prime factors $p$ and $q$ of $N$.

4.  **Key Recovery:**
    Once the prime factors $p$ and $q$ are known, the private key $d$ can be easily computed:
    ```python
    def generatePrivateKey(p, q, e):
        n = p * q
        phi = (p - 1) * (q - 1)
        d = modInverse(e, phi) # modInverse is the modular multiplicative inverse function
        return d, n
    ```
    With $p$ and $q$ in hand, the private exponent $d$ can be derived from the public exponent $e$ and Euler's totient function $\phi(n) = (p-1)(q-1)$, effectively compromising the RSA system.

### Current Limitations

Despite the theoretical power of Shor's algorithm, several practical limitations hinder its immediate threat to RSA-2048:

1.  **Quantum Hardware Limitations:**
    * **NISQ Era:** Current quantum computers operate in the Noisy Intermediate-Scale Quantum (NISQ) era, characterized by a limited number of physical qubits (typically fewer than a hundred).
    * **High Error Rates:** Quantum operations are prone to errors, leading to decoherence and loss of quantum information.
    * **Decoherence Times:** The fragile nature of qubits means they can only maintain their quantum properties for a very short duration.
    * **Lack of Fault Tolerance:** Present hardware lacks robust error correction mechanisms necessary for complex algorithms like Shor's on large inputs.

2.  **Algorithm Implementation Challenges:**
    * **Quantum Circuit Depth:** Factoring large numbers requires extremely deep quantum circuits, which are challenging to implement and execute reliably on current hardware.
    * **Resource Requirements:** The qubit count and coherence times required for Shor's algorithm to factor RSA-2048 are far beyond current capabilities (estimated in the millions of stable qubits).
    * **Error Correction:** Practical application of Shor's algorithm necessitates highly efficient quantum error correction codes, which are still an active area of research.
    * **Classical Post-processing Complexity:** While the quantum part is critical, the classical components also require significant computational resources.

3.  **Practical Considerations:**
    * **Small Number Factorization:** Current quantum computers can only factor very small numbers (e.g., 15 into 3 and 5), primarily for demonstration purposes.
    * **Scale for RSA-2048:** Factoring a 2048-bit RSA integer would require millions of stable, fault-tolerant qubits, a technology that is still decades away according to many estimates.
    * **Classical Efficiency for Small Numbers:** For numbers within the reach of current quantum computers, classical algorithms remain vastly more efficient.

4.  **Security Implications:**
    * **Current RSA-2048 Security:** RSA-2048 remains secure against attacks from existing classical computers and current, limited quantum computers.
    * **Need for Quantum-Resistant Cryptography:** The long-term threat necessitates the development and standardization of new cryptographic algorithms that are resistant to both classical and quantum attacks.
    * **Post-Quantum Cryptography Standards:** Organizations like NIST are actively working on standardizing post-quantum cryptographic algorithms to prepare for the quantum era.
    * **Uncertain Timeline:** While research points to accelerating timelines, the exact date when a fault-tolerant quantum computer capable of breaking RSA-2048 will exist remains uncertain.

## Future Work

This project can be extended in several directions:

* **Advanced Quantum Algorithm Implementations:** Implementations of more sophisticated quantum algorithms relevant to cryptanalysis or other computational problems.
* **Integration with Quantum Simulators and Hardware:** Further integration and experimentation with advanced quantum simulators and, where accessible, real quantum hardware.
* **Development of Quantum-Resistant Cryptographic Solutions:** Exploration and prototyping of cryptographic schemes designed to withstand quantum attacks.
* **Comparative Analysis of Quantum Approaches:** Analysis of different quantum computing paradigms and their potential impact on cryptographic security.

## Contributing

Contributions to this research project are highly encouraged. Please feel free to submit pull requests for new features, bug fixes, or improvements, or open issues to discuss ideas and challenges.

## Contributors

* **[Vaibhav Pandey](https://github.com/Its-Vaibhav-2005)** - Quantum Computing & Algorithms Lead
    * Implemented conceptual aspects of Shor's algorithm for quantum factorization.
    * Developed basic quantum circuit simulations using Qiskit.
    * Contributed to the analysis of quantum computing capabilities and optimization strategies.
    * Researched the landscape of quantum-resistant cryptography.

* **[Mahi Saxena](https://github.com/Mahitechlead)** - Cryptography Researcher
    * Implemented classical RSA encryption and decryption algorithms.
    * Developed cryptographic key generation systems.
    * Conducted research on post-quantum cryptography standards and proposals.
    * Performed analysis of general cryptographic vulnerabilities and their mitigation.

---

## References

1.  [Noisy quantum hardware could crack RSA-2048 in seven days](https://www.computerweekly.com/news/366625113/Noisy-quantum-hardware-could-crack-RSA-2048-in-seven-days)
2.  [Qiskit Documentation](https://qiskit.org/documentation/)
3.  [Post-Quantum Cryptography Standards (NIST)](https://www.nist.gov/pqcrypto)
4.  [Shor's Algorithm Implementation (Qiskit)](https://qiskit.org/documentation/stubs/qiskit.algorithms.Shor.html)
5.  [RSA (cryptosystem) - Wikipedia](https://en.wikipedia.org/wiki/RSA_(cryptosystem))

---

## Contact

### Vaibhav Pandey 

- **[pandey.vaibhav311005@gmail.com](pandey.vaibhav311005@gmail.com)**
- **[LinkedIn](https://www.linkedin.com/in/vaibhav-pandey-233ab62a8/)**

### Mahi Saxena

- **[mahi.cs.tech@gmail.com](mahi.cs.tech@gmail.com)**
- **[LinkedIn](https://www.linkedin.com/in/mahi-saxena-9b5308329/)**