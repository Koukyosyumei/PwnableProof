# PwnableProof

`PwnableProof` is a collection of vulnerable zero-knowledge proof circuits and their exploits. This repository also serves as an educational resource for developers and security researchers interested in understanding and mitigating vulnerabilities in zero-knowledge proof system.

**Contents**

1. [Core Concepts of ZKP](#1-core-concepts-of-zkp)
2. [ZKP System Architecture](#2-zkp-system-architecture)
3. [Implementing a ZKP System: A Practical Example](#3-implementing-a-zkp-system-a-practical-example)
4. [Security Considerations in ZKP Systems](#4-security-considerations-in-zkp-systems)
5. [Common Vulnerabilities](#5-common-vulnerabilities)
6. [Analysis Tools for ZKP Systems](#6-analysis-tools-for-zkp-systems)
7. [Circom: A Closer Look](#7-circom-a-closer-look)
8. [Papers](#8-papers)
9. [Resource](#9-resource)

## 1. Core Concepts of ZKP

### 1.1 *Zero-Knowledge Proofs (ZKPs)*

Zero-knowledge proofs are a cryptographic method that allows one party (the prover) to prove to another party (the verifier) that they know a piece of information without revealing it. ZKPs have three fundamental properties:

- *Completeness*: An honest prover can convince an honest verifier of a true statement.
- *Soundness*: A dishonest prover cannot convince the verifier of a false statement.
- *Zero-knowledge*: The verifier learns nothing beyond the truth of the statement.

Real-world applications of ZKPs include:

- Identity verification without disclosing personal data
- Proving sufficient funds for a transaction without revealing the exact balance
- Validating blockchain transactions while maintaining privacy

### 1.2 *zk-SNARKs*

zk-SNARK (Zero-Knowledge Succinct Non-Interactive Argument of Knowledge) is a popular type of ZKP with two distinguishing features:

- **Succinct**: Proofs are small and quick to verify, regardless of the statement's complexity.
- **Non-interactive**: The proof requires only one message from the prover to the verifier.

Notable zk-SNARK variants include:

- **Groth16**: Requires a trusted setup for each circuit.
- **PLONK**: Can use one trusted setup for multiple circuits.

### 1.3 Key Concepts in ZKP Implementation

1. *Finite fields*

ZKP arithmetic operates in finite fields, with the field size determined by the underlying elliptic curve. This means all operations are computed modulo a specific value.

2. *Constraints* and *Arithmetization*

In ZKP systems, statements to be proved are represented as constraints rather than instructions. Then, arithmetization is a crucial process in zero-knowledge proof systems that transforms computational logic into polynomial constraints. Several tools facilitate the arithmetization process:

- [Circom](https://docs.circom.io/): A compiler that translates high-level DSL into constraints (suppport R1CS). 
- [Halo2](https://zcash.github.io/halo2/): eDSL translates the specification written in Rust into constraints (support Plonkish).
- [CARIO](): A zero-knowledge virtual machine designed for arithmetization (support AIR).

While arithmetization is powerful, it does introduce computational overhead:

- Computation time can increase by nearly two orders of magnitude for SNARK-friendly operations.
- Non-friendly operations may experience even greater overhead.

To address these challenges, several optimization techniques have been developed:

- Lookup tables: Pre-computed values for common operations.
- SNARK-friendly cryptographic primitives: Algorithms like Rescue, SAVER, and Poseidon.
- Concurrent proof generation: Parallelizing the proof creation process.
- Hardware acceleration: Utilizing GPUs for faster computation.

There are several types of arithmetization, such as *R1CS*, *AIR*, and *Plonkish*. RICS only supports linear and quadratic polynomials, while AIR and Plonkish support polynomials with any order. In the case of AIR and Plonkish, we need to get the program's execution trace, establish the relationship between the rows, and interpolate polynomials.


### 1.4. Format of Arithmetization

#### 1.4.1 AIR

AIR stands for Algebraic Intermediate Representation and consists of three main elements:

- *Execution Trace*: This is a two-dimensional matrix where each row represents the state of a computation at a specific time point, and each column corresponds to an algebraic register tracked throughout the computation steps.

- *Transition Constraints*: These define algebraic relations between two or more rows of the execution trace, ensuring the correct evolution of the computation's state.

- *Boundary Constraints*: These enforce equality between specific cells of the execution trace and a set of constant values, effectively defining the input and output values of the computation.

AIR translates computational integrity statements into relations and properties of polynomials. It does this by expressing constraints as polynomials composed over the execution trace. This process allows complex computations to be represented in a form that can be efficiently verified using zero-knowledge proof systems.

Let's look at some simple examples to illustrate how AIR works:

**Example 1: Sum Constraint**

Consider a simple computation where we want to calculate the sum of the first n integers. We can represent this using AIR as follows:

*1. Execution Trace:*

- Column 1: Current number (i)
- Column 2: Running sum (s)

*2. Transition Constraint:*

- $s_{i+1} = s_{i} + (i + 1)$

*3. Boundary Constraints:*

- Initial state: $s_0 = 0, i_0 = 0$
- FInal state: $i_n = n$

|Step|	Current Number (i)|	Running Sum (s)|
|---|---|---|
|0|	0|	0|
|1|	1|	1|
|2|	2|	3|
|3|	3|	6|
|...|	...|	...|
|n|	n|	(n * (n + 1)) / 2|

**Example 2: Fibonacci Sequence**

For a slightly more complex example, let's consider the Fibonacci sequence:

*1. Execution Trace:*

- Column 1: F(n-1)
- Column 2: F(n)

*2. Transition Constraint:*

- $F(n + 1) = F(n) + F(n - 1)$

*3. Boundary Constraints:*

- Initial State: $F(0) = 0, F(1) = 1$

|Step|	F(n-1)|	F(n)|
|---|---|---|
|0|	0|	1|
|1|	1|	1|
|2|	1|	2|
|3|	2|	3|
|4|	3|	5|
|...|	...|	...|


## 2. ZKP System Architecture

ZKP systems typically consist of four layers:

1. **Circuit Layer**: Developers write a statement to be proved, which is sotimes referred as a *circuit*.
2. **Frontend Layer**: Compiles the circuit into arithmetic constraints and generates witnesses.
3. **Backend Layer**: Offers core SNARK operations (Setup, Prove, Verify).
4. **Integration Layer**: Connects the ZKP system with external applications.
 
### 2.1 Circuit Layer

At the base of any ZKP system is the Circuit Layer, where developers define the core logic of the proof system. This can be achieved using DSLs, such as Circom or Halo2, which allow the creation of circuits that represent computations. Another approach is using Zero-Knowledge Virtual Machines like Cario, where the arithmetic circuit represents the loop of fetching
instructions from memory and successively executing them.

### 2.2 Frontend Layer

The Frontend Layer is the intermediary between the high-level specification defined in the Circuit Layer and the rest of the ZKP system. Its key responsibilities include:

- Compilation: Translates the circuit into constraints via arithmetization, typically in a format such as R1CS, Plonkish, or AIR, which are used to define the relationships between variables in a way that can be proved in zero-knowledge.

- Witness Generation: Produces a witness based on both public and private inputs. The witness is a set of concrete values of all intermediate values that satisfy the circuit's constraints for a given input.

For instance, Circom’s compiler can transform a circuit into its corresponding R1CS form and generate the witness based on the provided inputs.

### 2.3 Backend Layer

The Backend Layer is responsible for the core functionality of the zero-knowledge proof system. This layer is where key cryptographic operations take place. Using tools like the snarkjs toolchain, the following primary tasks are performed:

- Setup: Initializes the proving system, which may include generating cryptographic parameters (for example, with a trusted setup in systems like Groth16).
- Prove: Generates a proof based on the witness and the circuit's constraints, without revealing sensitive information.
- Verify: Validates that a given proof is correct and that the input satisfies the circuit’s constraints.

### 2.4 Integration Layer

At the top, the Integration Layer connects the ZKP system to external applications or platforms. This could involve integrating with smart contracts, decentralized applications, or traditional systems. In a blockchain context, smart contracts might verify proofs and take actions based on the results of the verification.

For example, a smart contract could use an on-chain verifier to:

- Validate a submitted proof.
- Trigger contract logic based on whether the proof is valid.

## 3. Implementing a ZKP System: A Practical Example

Let's walk through the process of implementing a simple ZKP system using the `IsZero` circuit, which checks if an input is zero or non-zero.

### 3.1. Circuit Implementation (using Circom)

```
template IsZero() {
 signal input in;    // Input signal to check if it's zero or non-zero.
 signal output out;  // Output signal: 1 if `in == 0`, 0 if `in != 0`.
 signal inv;         // Inverse of the input when `in != 0`, or 0 when `in == 0`.
    
 // Compute the inverse: if `in` is non-zero, `inv` is set to `1/in`, otherwise it's 0.
 inv <-- in!=0 ? 1/in : 0;

 // Constraint 1: Ensures that if `in != 0`, `out` is 0. If `in == 0`, `out` is 1.
 out <== -in*inv +1;

 // Constraint 2: Ensures that `in * out == 0`, forcing the output to 1 only when `in == 0`.
 in*out === 0;
}

// there is no public input
component main = IsZero();
```

### 3.2. Compilation

Compile the circuit using Circom, which generates the necessary constraint system and related files:

```bash
circom iszero.circom --r1cs --wasm --sym --c
```

This step translates the circuit into its R1CS form and creates files necessary for generating witnesses and proving the circuit.

### 3.3. Trusted Setup (if required)

In protocols like Groth16, a trusted setup is required. This involves generating the proving and verifying keys.

```bash
snarkjs powersoftau new bn128 12 pot12_0000.ptau -v
snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v
```

### 3.4. Witness Generation

The witness is a set of intermediate and output values for a given input, which will later be used to generate the proof:

```bash
node iszero_js/generate_witness.js iszero.wasm input.json iszero_witness.wtns
```

For Groth16, we also perform a circuit-specific setup:

```bash
snarkjs powersoftau prepare phase2 pot12_0001.ptau pot12_final.ptau -v
snarkjs groth16 setup iszero.r1cs pot12_final.ptau iszero_0000.zkey
snarkjs zkey contribute iszero_0000.zkey iszero_0001.zkey --name="1st Contributor Name" -v
snarkjs zkey export verificationkey iszero_0001.zkey iszero_verification_key.json
```

### 3.5. Proof Generation

After generating the witness, we can produce the zero-knowledge proof using Groth16 or another SNARK protocol:

```bash
snarkjs groth16 prove iszero_0001.zkey iszero_witness.wtns iszero_proof.json iszero_public.json
```

### 3.6. Verification

Finally, the proof can be verified using the verification key and public inputs:

```bash
snarkjs groth16 verify iszero_verification_key.json iszero_public.json iszero_proof.json
```

If the proof is valid, the system confirms that the provided input satisfies the `IsZero` circuit logic, ensuring correctness without revealing sensitive information.

## 4. Security Considerations in ZKP Systems

When evaluating the security of ZKP systems, three primary concerns emerge:

1. **Soundness**

- Threat: A malicious prover convincing a verifier of a false statement.
- Impact: Compromises the integrity of the entire system.

2. **Completeness**

- Threat: Inability to verify valid proofs or produce proofs for valid statements.
- Impact: Renders the system unreliable or unusable.

3. **Zero-Knowledge Property**

- Threat: Information leakage about the private witness.
- Impact: Violates privacy guarantees, potentially exposing sensitive data.


## 5. Common Vulnerabilities

### Circuit Layer

Most reported ZKP-related vulnerabilities occur in the Circuit Layer and can be categorized as follows:

1. **Underconstraint Vulnerabilities**

- Description: Insufficient constraints in the circuit.
- Impact: Can lead to critical soundness errors, allowing false proofs to be accepted.

2. **Overconstrained Vulnerabilities**

- Description: Excessive constraints in the circuit.
- Impact: May cause rejection of valid witnesses by honest provers or benign proofs by honest verifiers.

3. **Computation/Hint Errors**

- Description: Errors in the computational part of a circuit.
- Impact: Can lead to incorrect results or unexpected behavior.

You can find some demos in the [circuits](./circuits) folder.

### Integration Layer

1. **Passing Unchecked Data**

- Description: Implicit constraints on public inputs aren't enforced by the verifier.
- Impact: Can cause both soundness and completeness issues, undermining the system’s integrity.

2. **Proof Delegation Error**

- Description: Vulnerabilities when proof generation is delegated to untrusted parties.
- Impact: Could lead to data leaks or other malicious activities.

3. **Proof Composition Error**

- Description: Issues arise when logic is spread across multiple proofs without proper enforcement.
- Impact: Lack of coordination can result in undefined behaviors


4. **ZKP Complementary Logic Error**

- Description: Flaws in logic that supports ZKP, such as the improper management of nullifiers in privacy-focused apps like Tornado Cash.
- Impact: Can lead to exploits like repeated withdrawals, draining funds due to missing checks.

### Frontend Layer

1. **Incorrect Constraint Compilation**

- Description: Errors during constraint compilation, often due to arithmetization or optimization issues.
- Impact: Could result in invalid proofs being accepted or valid ones rejected, similar to a compiler bug.

2. **Witness Generation Error**

- Description: Mistakes during witness generation, such as producing invalid witnesses or crashes.
- Impact: May cause denial of service or incorrect witness generation.

### Backend Layer

1. **Setup Error**

- Description: Issues during the generation of public parameters.
- Impact: Can severely compromise system integrity if parameters are incorrect or easily compromised.


2. **Prover Error**

- Description: Problems with the prover's operation, like accepting invalid witnesses or violating zero-knowledge properties.
- Impact: May break the proof system's integrity.


3. **Unsafe Verifier**

- Description: Missing checks or incorrect operations in the verifier.
- Impact: Can allow invalid proofs or mathematical errors, jeopardizing the proof system, whether manually or framework-generated.

## 6. Analysis Tools for ZKP Systems

Several tools have been developed to analyze and verify ZKP systems:

### 6.1 Static Analysis Tools

These tools rely on pattern matching and heuristics to identify potential issues:

- [Circomspect](https://github.com/trailofbits/circomspect): Static analyzer for Circom.
- [ZKAP](https://github.com/whbjzzwjxq/ZKAP?tab=readme-ov-file): Static analyzer for Circom. See [Practical Security Analysis of Zero-Knowledge Proof Circuits](#practical-security-analysis-of-zero-knowledge-proof-circuits-usenix24).
- [halo2-analyzer](https://github.com/quantstamp/halo2-analyzer): Static analyzer for Halo2. See [Automated Analysis of Halo2 Circuits](#automated-analysis-of-halo2-circuits).

These tools are typically limited to specific vulnerability patterns and support only particular DSLs or eDSLs.

### 6.2 Dynamic Analysis and Fuzzing

- [SnarkProbe](https://github.com/BARC-Purdue/SNARKProbe): A security analysis framework for SNARKs that can analyze R1CS-based libraries and applications. It detects various issues such as edge case crashing, errors, and inconsistencies. See [SNARKProbe: An Automated Security Analysis Framework for zkSNARK Implementations](#snarkprobe-an-automated-security-analysis-framework-for-zksnark-implementations).
- [Cairo-Fuzzer](https://github.com/FuzzingLabs/cairo-fuzzer): Cairo/Starknet smart contract fuzzer.

### 6.3 Formal Verification Tools

These tools provide more rigorous analysis:

- [Picus](https://github.com/chyanju/Picus): Uses symbolic execution to verify that Circom circuits are not under-constrained. See [Automated Detection of {Under-Constrained} Circuits in {Zero-Knowledge} Proofs](#automated-detection-of-under-constrained-circuits-in-zero-knowledge-proofs-pldi23).
- [Horus](https://github.com/NethermindEth/horus-checker): Performs formal verification of Cairo smart contracts using SMT solvers.
- [Medjai](https://github.com/Veridise/Medjai): A symbolic execution tool for Cairo programs.
- [Code](https://github.com/Veridise/Coda): Certified circom circuits in Coq. See [Certifying Zero-Knowledge Circuits with Refinement Types](#certifying-zero-knowledge-circuits-with-refinement-types-sp24).
- [Leo](https://github.com/ProvableHQ/leo): A functional, statically-typed programming language built for writing private applications. See [LEO: A Programming Language for Formally Verified, Zero-Knowledge Applications](#leo-a-programming-language-for-formally-verifiedzero-knowledge-applications-iacr21).
- [Gnark Lean Extractor](https://github.com/reilabs/gnark-lean-extractor?tab=readme-ov-file#gnark-lean-extractor): Compiles zero-knowledge circuits from Gnark to the Lean theorem prover for formal verification.

## 7. Circom: A Closer Look

### Signals

Circuits in Circom work with signals, which are elements (or arrays of elements) from a finite field. These signals are declared using the keyword `signal`. Signals can be labeled as either `input` or `output`. If a signal is not explicitly marked as either, it is treated as an intermediate signal. When given an input signal, the witness generator produced by the Circom compiler calculates the values of all intermediate and output signals.

### Operators

The Circom compiler is responsible for generating both the witness calculator program and the Rank-1 Constraint System (R1CS) constraints. To do this, it provides several operators that are used to express both computations (for witness calculation) and constraints (for constraint generation).

For example, the operators `<--` and `-->` are used for signal assignment, which is critical for witness generation. Meanwhile, the `===` operator is used for constraint generation. A statement like `a === b` tells the Circom compiler to generate a circuit that ensures `a` and `b` are equal. Circom offers the additional operators `<==` and `==>`, which perform both assignment and constraint generation at the same time. Note that any constraint in Circom should be a quadratic equation.

### Execution vs Constraints

Circom requires all constraints to be expressed as quadratic equations. However, developers can implement more complex algorithms by separating computation from constraints. For example:

```
template Divider() {
 signal input a;
 signal input b;
 signal output c;
 c <-- a/b;
 a === b * c;
}
```

In this example, the division operation c = a / b is computed separately, while the constraint a === b * c ensures the correctness of the result within the quadratic constraint system. Such **deviant between the computation and the constraint** sometimes lead to under/over constraints vulnerabilities.

## 8. Papers

### Static Analysis

#### [Practical Security Analysis of Zero-Knowledge Proof Circuits (USENIX'24)](https://www.cs.utexas.edu/~isil/zkap.pdf)
  
>Tag: `{Type: static analysis, DSL:circom, Arithmetization:R1CS, Target:circuit (under-constrainted)}`

<details>

***Overview***: This paper presents **ZKAP**, the heuristic-based tool for detecting common vulnerabilities in Circom, a popular DSL for building zero-knowledge proof (ZKP) circuits. The design of ZKAP is based on a manual study of existing Circom vulnerabilities, which helped classify the root causes of bugs into three main categories:

1. Nondeterministic signals: Input or output signals are not properly constrained.
2. Unsafe component usage: Components are used incorrectly, leading to signal errors.
3. Constraint-computation discrepancies: Mismatches occur between witness generation and constraint enforcement like under/over constraints.

*Threat Model*: The analysis assumes a trustless environment where attackers have full access to public information, including blockchain states, deployed smart contracts, and ZK circuit source code. Attackers can also deploy their own contracts and ZK applications to interact with the target system.

***Method***: To detect these vulnerabilities, the authors introduced the *circuit dependence graph (CDG)*, an abstraction that captures key properties of Circom circuits to identify semantic vulnerability patterns. ZKAP uses this graph to implement static checkers that detect vulnerabilities through anti-patterns described in a Datalog-style language.

***Experiment***: The tool was evaluated on 258 Circom circuits from popular projects on GitHub, achieving an F1 score of 0.82.

</details>

------------------------------

### Dynamic Analysis

#### [SNARKProbe: An Automated Security Analysis Framework for zkSNARK Implementations](https://link.springer.com/chapter/10.1007/978-3-031-54773-7_14)

>Tag: `{Type: dynamic analysis, Arithmetization: R1CS, Target: compilier}`

<details>

***Overview*** This paper presents an automated security analysis framework for zkSNARKs that scans R1CS-based libraries and applications to detect issues such as edge case crashes, cryptographic operation errors, and protocol inconsistencies. By using dynamic analysis, we trace real-time data and variables across various zkSNARK libraries written in different languages, without requiring manual language-specific adaptations. Custom fuzzing techniques generate valid and invalid R1CS matrices to exercise different code paths in a library, while SMT solvers verify the consistency between user-specified proof equations and the R1CS matrix used in the application.

***Method*** The framework performs two key steps. First, the Constraint Checker uses an SMT solver to compare statement equations with the R1CS matrix, checking for errors in the conversion process. Second, the Branch Model fuzzes the input to generate R1CS matrices and monitors branch coverage during execution, providing feedback to improve fuzzing coverage. The Value Model then re-evaluates protocol calculations, detecting potential cryptographic logic errors caused by unexpected value changes or implementation mistakes. This end-to-end framework, SNARKProbe, assesses both the correctness and security of a zkSNARK library. Unlike traditional fuzzers, SnarkFuzzer can detect both software bugs (such as crashes or overflows) and cryptographic logic errors, which are harder to detect since they do not cause crashes but result in incorrect cryptographic outputs, such as miscomputed ciphertexts.

***Experiment*** This paper focus on two popular zkSNARK protocols, Pinocchio and Groth16, and evaluate four libraries: `libsnark` (C++), `Bellman` (Rust), `arkworks` (Rust), and `gnark` (Go). These libraries are widely used in real-world zkSNARK projects. Although our tool uncovered a few inconsistencies between gadget implementations and protocol specifications, none of these issues were exploitable, new, or of significant concern for disclosure, but they do require careful attention from developers when implementing proofs.

</details>

------------------------------

### Formal Method

#### [Automated Detection of Under-Constrained Circuits in Zero-Knowledge Proofs (PLDI'23)](https://dl.acm.org/doi/pdf/10.1145/3591282)

>Tag: `{Type: formal method, DSL:circom, Arithmetization:R1CS, Target:circuit (under-constrainted), Others:[SMT-solver]}`

<details>

***Overview***: This paper introduces a novel technique to detect under-constrained polynomial equations in ZKP circuits over finite fields. The method performs semantic analysis on the equations generated by the compiler to determine if each signal is uniquely constrained by the inputs. The approach combines SMT solving with lightweight inference to effectively identify under-constrained circuits.

***Method***: The process begins with lightweight inference rules to propagate uniqueness constraints and switches to SMT-based reasoning when necessary. The analysis ends when the SMT solver either finds a proof or counterexample, when all output variables are proven to be constrained, or when no further progress can be made. Because this method operates directly on arithmetic circuits, it is not tied to any specific DSL and can be applied to various ZK-snark-compatible DSLs. Notably, it produces no false positives.

***Experiment***: The method successfully analyzed 70% of the tested benchmarks (163 circuits), although it is still difficult to validate relatively larger circuits that contains more thatn 100 constraints.

</details>

------------------------------

#### [Certifying Zero-Knowledge Circuits with Refinement Types (S&P'24)](https://eprint.iacr.org/2023/547.pdf)

>Tag: `{Type: formal method, DSL: New DSL, Arithmetization: R1CS, Target: circuit}`

<details>

***Overview***: This paper introduces CODA, a statically typed language for building zero-knowledge applications. CODA allows developers to formally specify and verify properties of ZK applications using a powerful refinement type system. A major challenge in verifying ZK applications is reasoning about polynomial equations over large prime fields, which are often beyond the reach of automated theorem provers. CODA addresses this by generating Coq lemmas that can be interactively proven using a tactic library.

***Experiment***: The authors evaluated CODA on 77 ZK circuits from 9 widely-used libraries and projects in Circom. Because CODA is sound, any bugs in the program will result in unprovable lemmas in Coq. During testing, 6 benchmarks failed to discharge their proof obligations, leading to the discovery of subtle, previously unknown correctness bugs in the original Circom circuits.

</details>

------------------------------

#### [LEO: A Programming Language for Formally Verified,Zero-Knowledge Applications (IACR'21)](https://docs.zkproof.org/pages/standards/accepted-workshop4/proposal-leo.pdf)

>Tag: `{Type: formal method, DSL: New DSL, Arithmetization: R1CS, Target: circuit}`

<details>

***Overview***: LEO is a high-level, general-purpose programming language designed for circuit synthesis, particularly for zero-knowledge applications on the Aleo blockchain. It specifically targets R1CS arithmetization, with completeness proven using ACL2, an industrial-strength theorem prover. LEO offers two key benefits: it ensures formal verification of applications against their high-level specifications, and it allows anyone to succinctly verify these applications, regardless of their size.

</details>

------------------------------

#### [CLAP: a Semantic-Preserving Optimizing eDSL for Plonkish Proof Systems](https://arxiv.org/pdf/2405.12115)

>Tag: `{Type: formal method, DSL: New DSL, Arithmetization: Plonkish, Target:circuit}`

<details>

***Overview***: CLAP is the first Rust eDSL with a proof system-agnostic circuit format designed for extensibility, automatic optimization, and formal assurance of constraint systems. It treats the production of Plonkish constraint systems and witness generators as a semantic-preserving compilation problem, ensuring soundness and completeness to prevent under- or over-constraining errors.

***Method***: In traditional approaches, circuit developers work in an eDSL and later hand off the constraint system to proof engineers, leading to a disconnect between development and verification. CLAP solves this by integrating the entire process, offering a sound and complete architecture from the start.

CLAP also provides automatic safe optimizations, which can be applied only after the circuit is fully defined. This avoids premature optimization issues, such as missing constraints, and ensures that optimizations—like removing duplicate checks—are safe and context-aware. Additionally, CLAP allows for circuit reuse by generating arithmetic gates that can be automatically optimized for different proof systems, such as Boojum.

For custom gates, which trade prover time for verification efficiency, CLAP’s inlining optimizer can flatten complex logic (like a Poseidon hash round) into a custom gate of the required degree, saving time in development and review.

***Experiment***: CLAP was validated using Boojum circuits from ZKsync Era, which are manually optimized by experts and reviewed by auditors. Despite starting with more constraints, CLAP’s optimizations resulted in 10% fewer constraints compared to hand-optimized circuits.

</details>

------------------------------

#### [Compositional Formal Verification of Zero-Knowledge Circuits](https://eprint.iacr.org/2023/1278.pdf)

>Tag: `{Type: formal method, Target: circuit}`

------------------------------

#### [Formal Verification of Zero-Knowledge Circuits](https://arxiv.org/pdf/2311.08858)

>Tag: `{Type: formal method, Target: circuit}`

------------------------------

#### [Scalable Verification of Zero-Knowledge Protocols (S&P'24)](https://www.computer.org/csdl/proceedings-article/sp/2024/313000a133/1Ub23QzVaWA)

>Tag: `{Type: formal method, Target: circuit}`

------------------------------

#### [Automated Analysis of Halo2 Circuits](https://eprint.iacr.org/2023/1051.pdf)

>Tag: `{Type: formal method, Target: circuit}`

------------------------------

#### [Bounded Verification for Finite-Field-Blasting](https://link.springer.com/content/pdf/10.1007/978-3-031-37709-9_8.pdf)

>Tag: `{Type: formal method, Target: compilier}`

<details>

***Overview***: This paper addresses ZKP compiler correctness by partially verifying the field-blasting compiler pass, translating Boolean and bitvector logic into finite field operations. The contributions of this paper include:

1. Correctness Definition: It introduces a precise correctness definition for ZKP compilers, ensuring that the compiler preserves the soundness and completeness of the underlying ZK proof system. Specifically, suppose a ZK proof system is specified in a low-level language (L) and compiled from a high-level language (H) to L. In that case, the compiler must maintain these properties for statements in H. The definition is also compositional, meaning proving correctness for each compiler pass suffices to prove correctness for the whole compiler.

2. Verifiable Field-Blaster Architecture: The paper presents an architecture for a verifiable field-blaster, consisting of a set of encoding rules. It provides verification conditions (VCs) for these rules, and shows that if the VCs hold, the field-blaster is correct. These conditions can be automatically checked (in bounded form), reducing both the initial and ongoing costs of verification.

</details>

------------------------------

#### [The Ouroboros of ZK: Why Verifying the Verifier Unlocks Longer-Term ZK Innovation](https://eprint.iacr.org/2024/768.pdf)

>Tag: `{Type: formal method, Target: verifier}`

------------------------------

#### [Weak Fiat-Shamir Attacks on Modern Proof Systems](https://eprint.iacr.org/2023/691.pdf)

>Tag: `{Type: formal method, Target: Fiat-Shamir Transform}`

------------------------------
  
#### [The Last Challenge Attack: Exploiting a Vulnerable Implementation of the Fiat-Shamir Transform in a KZG-based SNARK](https://eprint.iacr.org/2024/398)

>Tag: `{Type: formal method, Target: Fiat-Shamir Transform}`

------------------------------

### SMT Solver for Finite Fields

There are some papers that propose SMT solvers specially designed to solve constraints in the finite fields.

- [An SMT-LIB Theory of Finite Fields](https://ceur-ws.org/Vol-3725/paper3.pdf)

- [Satisfiability Modulo Finite Fields](https://link.springer.com/content/pdf/10.1007/978-3-031-37703-7_8.pdf)

- [SMT Solving over Finite Field Arithmetic](https://arxiv.org/pdf/2305.00028)

------------------------------

### SoK

- [SoK: What Don’t We Know? Understanding Security Vulnerabilities in SNARKs (USENIX'24)](https://arxiv.org/pdf/2402.15293)

- [Zero-Knowledge Proof Vulnerability Analysis and Security Auditing](https://eprint.iacr.org/2024/514.pdf)

## 9. Resource

| **Category** | **Title** | **Link** |
|--------------|-----------|----------|
| **Documentation** | Circom | [Link](https://docs.circom.io/) |
|              | Cairo | [Link](https://docs.cairo-lang.org/) |
| **Curation** | Awesome Zero-Knowledge Proofs Security | [Link](https://github.com/Xor0v0/awesome-zero-knowledge-proofs-security?tab=readme-ov-file) |
|  | Awesome ZKP Security | [Link](https://github.com/StefanosChaliasos/Awesome-ZKP-Security?tab=readme-ov-file) |
|  | Awesome Starknet | [Link](https://github.com/keep-starknet-strange/awesome-starknet?tab=readme-ov-file) |
|  | Awesome Cairo               | [Link](https://github.com/auditless/awesome-cairo?tab=readme-ov-file) |
| **Audits**| zksecurity | [Link](https://www.zksecurity.xyz/reports/) |
|           | nullity00/zk-security-reviews | [Link](https://github.com/nullity00/zk-security-reviews) |
| **Blogs** | The State of Security Tools for ZKPs | [Link](https://www.zksecurity.xyz/blog/posts/zksecurity-tools/) |
|           | A beginner's intro to coding zero-knowledge proofs| [Link](https://dev.to/spalladino/a-beginners-intro-to-coding-zero-knowledge-proofs-c56) |
|           | Arithmetization schemes for ZK-SNARKs | [Link](https://blog.lambdaclass.com/arithmetization-schemes-for-zk-snarks/) |
|           | Medjai: Protecting Cairo code from Bugs| [Link](https://medium.com/veridise/medjai-protecting-cairo-code-from-bugs-d82ec852cd45) |
|           | Moving from Solidity to Cairo | [Link](https://medium.com/starkware/moving-from-solidity-to-cairo-7d44f9723c68) |
|           | An Introduction to AIR        | [Link](https://threesigma.xyz/blog/an-introduction-to-air) |
|           | Under-constrained computation, a new kind of bug | [Link](https://diligence.consensys.io/blog/2022/01/under-constrained-computation-a-new-kind-of-bug/) |    
| | Anatomy of a STARK | [Link](https://neptune.cash/learn/stark-anatomy/) |
| | Noir Explained: Features and Examples | [Link](https://oxor.io/blog/2024-06-18-noir-explained-features-and-examples/) |
|             | Discovering and Fixing a Critical Vulnerability in Polygon zkEVM | [Link](https://blog.verichains.io/p/discovering-and-fixing-a-critical) |
|             | Cairo Security Flaws | [Link](https://oxor.io/blog/2024-08-16-cairo-security-flaws/#security-practices) |
|             | Building zk-SNARKs (volume 1) | [Link](https://medium.com/iovlabs-innovation-stories/building-zk-snarks-volume-1-8e8bd4e4a012) |
|             | The RareSkills Book of Zero Knowledge | [Link](https://www.rareskills.io/zk-book) |
| **Courses** | STARK 101 | [Link](https://starkware.co/stark-101/) |
|             | ZK Whiteboard | [Link](https://zkhack.dev/whiteboard/) |
|             | [MIT IAP 2023] Modern Zero Knowledge Cryptography | [Link](https://zkiap.com/) |

