# Halo2 Solidity Verifier

> ⚠️ This repo has NOT been audited and is NOT intended for a production environment yet.

Solidity verifier generator for [`halo2`](http://github.com/privacy-scaling-explorations/halo2) proof with KZG polynomial commitment scheme on BN254.

For audited solidity verifier generator and proof aggregation toolkits, please refer to [`snark-verifier`](http://github.com/axiom-crypto/snark-verifier).

## Usage

### Generate verifier and verifying key separately as 2 solidity contracts

```rust
let generator = SolidityGenerator::new(&params, &vk, Bdfg21, num_instances);
let (verifier_solidity, vk_solidity) = generator.render_separately().unwrap();
```

Check [`examples/separately.rs`](./examples/separately.rs) for more details.

> [!NOTE]
> Currently example run only with rust `1.77.0` version due to a `cargo` update ([rust-lang/rust#123285](https://github.com/rust-lang/rust/issues/123285)).
>
> The `rust` toolchain version is specified in [rust-toolchain.toml](./rust-toolchain.toml) file.

Run example with the following command:

```bash
cargo run --all-features --example separately
```

### Generate verifier and verifying key in a single solidity contract

```rust
let generator = SolidityGenerator::new(&params, &vk, Bdfg21, num_instances);
let verifier_solidity = generator.render().unwrap();
```

### Encode proof into calldata to invoke `verifyProof`

```rust
let calldata = encode_calldata(vk_address, &proof, &instances);
```

Note that function selector is already included.

## Test

To run tests, use the following command:

```bash
cargo test --workspace --all-features --all-targets -- --nocapture
```

> [!NOTE]
> Currently tests run only with rust `1.77.0` version due to a `cargo` update ([rust-lang/rust#123285](https://github.com/rust-lang/rust/issues/123285)).
>
> The `rust` toolchain version is specified in [rust-toolchain.toml](./rust-toolchain.toml) file.


## Limitations & Caveats

- It only allows circuit with **less or equal than 1 instance column** and **no rotated query to this instance column**.
- Currently even the `configure` is same, the [selector compression](https://github.com/privacy-scaling-explorations/halo2/blob/7a2165617195d8baa422ca7b2b364cef02380390/halo2_proofs/src/plonk/circuit/compress_selectors.rs#L51) might lead to different configuration when selector assignments are different. To avoid this, please use [`keygen_vk_custom`](https://github.com/privacy-scaling-explorations/halo2/blob/6fc6d7ca018f3899b030618cb18580249b1e7c82/halo2_proofs/src/plonk/keygen.rs#L223) with `compress_selectors: false` to do key generation without selector compression.

## Compatibility

The [`Keccak256Transcript`](./src/transcript.rs#L19) behaves exactly same as the `EvmTranscript` in `snark-verifier`.

## Design Rationale

The current solidity verifier generator within `snark-verifier` faces a couple of issues:

- The generator receives only unoptimized, low-level operations, such as add or mul. As a result, it currently unrolls all assembly codes, making it susceptible to exceeding the contract size limit, even with a moderately sized circuit.
- The existing solution involves complex abstractions and APIs for consumers.

This repository is a ground-up rebuild, addressing these concerns while maintaining a focus on code size and readability. Remarkably, the gas cost is comparable, if not slightly lower, than the one generated by `snark-verifier`.

## Acknowledgement

The template is heavily inspired by Aztec's [`BaseUltraVerifier.sol`](https://github.com/AztecProtocol/barretenberg/blob/4c456a2b196282160fd69bead6a1cea85289af37/sol/src/ultra/BaseUltraVerifier.sol).
