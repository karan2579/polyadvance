# Polygon-zkEVM_module3_ZK-Circuit-Implementation-Challenge
# Custom zkSnark Circuit Verification with Zardkat

This project is based on a [hardhat-circom](https://github.com/projectsophon/hardhat-circom) template designed for generating zero-knowledge circuits, proofs, and Solidity verifiers.

## Description

### Quick Start

This project involves compiling the `VibCustomCircuit()` circuit and verifying it against a smart contract verifier.

**Implemented Circuit:**

![Circuit](https://drive.google.com/uc?export=view&id=1eyO6T2YSWV-WYpRLfRRBc6xMNtRGGhRt)

```circom
pragma circom 2.0.0;

/* This circuit template verifies that Q is the output of the custom circuit shown in the image above, based on inputs A and B. */ 

template VibCustomCircuit() {
   signal input A;
   signal input B;

   signal X;
   signal Y;

   signal output Q;

   component andGate = AND();
   component notGate = NOT();
   component orGate = OR();

   andGate.a <== A;
   andGate.b <== B;
   X <== andGate.out;
   notGate.in <== B;
   Y <== notGate.out;
   orGate.a <== X;
   orGate.b <== Y;
   Q <== orGate.out;
}

template AND() {
    signal input a;
    signal input b;
    signal output out;

    out <== a * b;
}

template NOT() {
    signal input in;
    signal output out;

    out <== 1 + in - 2 * in;
}

template OR() {
    signal input a;
    signal input b;
    signal output out;

    out <== a + b - a * b;
}

component main = VibCustomCircuit();
```

### Installation

To install the required dependencies, run:
```bash
npm install
```

### Compilation

Compile the circuit using:
```bash
npx hardhat circom --verbose
```
This command generates the necessary intermediate files in the **out** directory and creates the **MultiplierVerifier.sol** contract.

### Proving and Deploying

To generate a proof and deploy the verifier, use:
```bash
npx hardhat run scripts/deploy.ts --network zkEVM
```
This script performs the following actions:
1. Deploys the `MultiplierVerifier.sol` contract.
2. Generates a proof from the circuit intermediaries using `generateProof()`.
3. Prepares the calldata with `generateCallData()`.
4. Calls `verifyProof()` on the deployed verifier contract using the generated calldata.

With just two commands, you can compile a ZKP, generate a proof, deploy a verifier, and verify the proof. 🎉

## Configuration

### Directory Structure

**circuits**
```
├── multiplier
│   ├── circuit.circom
│   ├── input.json
│   └── out
│       ├── circuit.wasm
│       ├── multiplier.r1cs
│       ├── multiplier.vkey
│       └── multiplier.zkey
├── new-circuit
└── powersOfTau28_hez_final_12.ptau
```
Each circuit resides in its own directory. The circom file and its input file are located at the top level of each directory. The **out** directory, which is generated automatically, stores the compiled outputs, keys, and proofs. The Powers of Tau file is sourced from the Polygon Hermez ceremony to streamline the setup process.

**contracts**
```
contracts
└── MultiplierVerifier.sol
```
Verifier contracts are autogenerated and named after the circuit, e.g., **MultiplierVerifier** for the multiplier circuit.

## Hardhat Configuration

In `hardhat.config.ts`, the circom setup is configured as follows:
```javascript
circom: {
    inputBasePath: "./circuits",
    ptau: "powersOfTau28_hez_final_12.ptau",
    circuits: JSON.parse(JSON.stringify(circuits))
}
```

### Circuits Configuration

Circuit configurations are managed separately in `circuits.config.json`:
```json
[
  {
    "name": "multiplier",
    "protocol": "groth16",
    "circuit": "multiplier/circuit.circom",
    "input": "multiplier/input.json",
    "wasm": "multiplier/out/circuit.wasm",
    "zkey": "multiplier/out/multiplier.zkey",
    "vkey": "multiplier/out/multiplier.vkey",
    "r1cs": "multiplier/out/multiplier.r1cs",
    "beacon": "0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f"
  }
]
```

### Adding New Circuits

To add a new circuit, run the following Hardhat task:
```bash
npx hardhat newcircuit --name newcircuit
```
This command will automatically generate the necessary configuration and directory structure.

### Deterministic Builds

For development, you can use the `--deterministic` flag to avoid generating new keys every time:
> When recompiling the same circuit using the Groth16 protocol, the plugin typically generates new zkey files with different beacons. For easier inspection during development, use the `--deterministic` flag to apply a non-random, hardcoded entropy value (default: 0x000000), which can be set in the circuit's configuration.

---

**Author:** Karanveer Mittal
