# EZIO

<p>
  <a href="./LICENSE"><img alt="License" src="https://img.shields.io/badge/license-AGPLv3-blue.svg"></a>
  <img alt="Status" src="https://img.shields.io/badge/status-MVP-purple">
</p>

Privacy without Compliance is banned, Compliance without privacy is pointless.


Private fund transfers powered by **zero-knowledge proofs**. Enables confidential payments where the transferred amount and sender–receiver linkage remain hidden, while correctness and one‑time spend are enforced on-chain.

Users maintain two balance types:
- **Public Balance:** Visible on-chain
- **Private Balance:** Hidden via Poseidon hash commitments


<div style="display:flex;flex-wrap:wrap;gap:8px;align-items:flex-start;justify-content:space-between">
  <img src="./assets/frontend_1.jpg" alt="App screenshot 1" style="width:48%;max-width:450px;height:auto;border-radius:6px;box-shadow:0 1px 3px rgba(0,0,0,.08);">
  <img src="./assets/frontend_2.jpg" alt="App screenshot 2" style="width:48%;max-width:450px;height:auto;border-radius:6px;box-shadow:0 1px 3px rgba(0,0,0,.08);">
</div>

## Features
- Built on a modified version of EIP‑7503 flow
- No sender/receiver linkage is preserved
- No linkability through IP or timings
- Transferred amount remains private
- Nullifier prevents double spends

## Why EZIO
- Previous privacy enhancing protocol were either completely untrackable or a little to public but, loki provides a perfect balance between enhanced privacy and governmental control which none could achieve yet
- Enhances the privacy of users by highly increasing the computation which most people can't do
- Government if wants can track a particular address and its transactions by performing high computations and thus retain the control which will avoid us from getting banned

  
## How it works

![Core protocol](./assets/protocol.jpg)

At a glance:

- `noteC` binds Alice’s private balance deduction to the note Bob can later claim
- `proof_A` proves Alice honestly deducted `n` and posted `noteC`
- `proof_B` proves only the intended recipient (holder of `pk_B`) can claim
- A `nullifier` guarantees each note can be used only once, No double spend/transfer.

## Project layout

- `./ezio` – main frontend
- `./contract` – Solidity smart contracts (Hardhat v3)
- `./circuits` – Circom circuits for zk proofs

## Quickstart
Below are the minimal steps to compile and check the Circom circuits. Frontend and contracts have their own standard workflows; see the respective folders for details.


## Frontend (ezio)

The `ezio` app is the primary UI. Typical steps are: install deps, set env vars, and start the dev server.

**Nexus** handles user authentication and identity management, while **PayPal USD (PYUSD)** enables stable, on-chain payments verified through ZK proofs for secure, compliant transactions.
[For more info on how nexus and PayUSD are used](./ezio/README.md)

```bash
cd ezio
npm install -g pnpm
pnpm install
pnpm dev
```
you might need to run `pnpm approve-build` approve all of them

## Contracts
Contracts live in `./contract` (with additional work in `./contract_v2`). Use a modern Hardhat toolchain. See the folders for tasks such as build, test, and deploy.


| Contract | Address | Purpose |
|----------|---------|---------|
| **Main_Contract** | `0xb21AD25eC6d65d92C998c76a22b3f5Dce2F9F7CB` | Core balance management with ZK privacy |
| **Groth16Verifier** | `0x99923435d5774c962dC5c604Ee9970748E9FD0E2` | Verifier for Circuit A (update_balance.circom) |
| **Groth16VerifierB** | `0x777B6C1bB0608621f8d2AAd364890267A4488Ce1` | Verifier for Circuit B (proofB.circom) |
| **Burner_Verifier** | `0x8Da48CfBCFC981c0f4342D8c3e22cd5A5cB41eCE` | Source chain burn with ZK verification |
| **Minter_Verifier** | `0x78CAb97E087b7696eE31e0cdDCA25AcaA568C237` | Destination chain mint with dual ZK verification |

**Owner Address:** `0xFb93a8DcD5edc3FB6Cb34d77C6811835756c99A0`

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Main_Contract                          │
│  • Public/Private Balance Management                        │
│  • Nullifier Tracking (double-spend prevention)            │
│  • Owner: 0xFb93a8DcD5edc3FB6Cb34d77C6811835756c99A0       │
└───────────────┬──────────────────────┬─────────────────────┘
                │                      │
        ┌───────▼─────┐        ┌───────▼─────┐
        │  Burn Path  │        │  Mint Path  │
        └───────┬─────┘        └───────┬─────┘
                │                      │
    ┌───────────▼───────────┐  ┌──────▼──────────────────┐
    │  Burner_Verifier      │  │  Minter_Verifier        │
    │  • Circuit A Proof    │  │  • Circuit A + B Proofs │
    │  • Groth16Verifier    │  │  • Dual Verification    │
    └───────────────────────┘  └─────────────────────────┘
```

## View on Etherscan

- [Main_Contract](https://sepolia.etherscan.io/address/0xb21AD25eC6d65d92C998c76a22b3f5Dce2F9F7CB)
- [Groth16Verifier](https://sepolia.etherscan.io/address/0x99923435d5774c962dC5c604Ee9970748E9FD0E2)
- [Groth16VerifierB](https://sepolia.etherscan.io/address/0x777B6C1bB0608621f8d2AAd364890267A4488Ce1)
- [Burner_Verifier](https://sepolia.etherscan.io/address/0x8Da48CfBCFC981c0f4342D8c3e22cd5A5cB41eCE)
- [Minter_Verifier](https://sepolia.etherscan.io/address/0x78CAb97E087b7696eE31e0cdDCA25AcaA568C237)

## Circuits Overview

Compile circuits:
```bash
cd circuits
make
```

Verify constraints/proofs (where applicable):
```bash
make verify
```
### Circuit A: `update_balance.circom` (Burner/Sender Side)

**Purpose:** Proves correct balance state transitions for burn/mint operations.

**Inputs:**

```circom
signal input pub_balance;           // Current public balance
signal input priv_balance;          // Current private balance
signal input new_priv_balance;      // New private balance after operation
signal input r;                     // Randomness for old commitment
signal input r_new;                 // Randomness for new commitment
signal input secret;                // User secret (nullifier preimage)
```

**Outputs (5 public signals):**

```circom
signal output old_commitment;       // Commit(priv_balance, r)
signal output new_commitment;       // Commit(new_priv_balance, r_new)
signal output curr_pub_balance;     // = pub_balance
signal output new_priv_balance_out; // = new_priv_balance
signal output nullifier;            // Hash(secret)
```

**Constraints:**

- Old commitment = Poseidon(priv_balance, r)
- New commitment = Poseidon(new_priv_balance, r_new)
- Nullifier = Poseidon(secret)
- All values properly constrained

**Used by:** `Groth16Verifier` (deployed on-chain)

---

### Circuit B: `proofB.circom` (Minter/Receiver Side)

**Purpose:** Validates transfer amounts using commitment scheme.

**Inputs:**

```circom
signal input priv_balance;          // Current private balance
signal input new_priv_balance;      // New private balance
signal input r;                     // Randomness for old commitment
signal input r_new;                 // Randomness for new commitment
signal input amount;                // Transfer amount
```

**Outputs (3 public signals):**

```circom
signal output old_commitment;       // Commit(priv_balance, r)
signal output new_commitment;       // Commit(new_priv_balance, r_new)
signal output amount_hash;          // Poseidon(amount)
```

**Constraints:**

- Old commitment = Poseidon(priv_balance, r)
- New commitment = Poseidon(new_priv_balance, r_new)
- Amount hash = Poseidon(amount)
- Balance constraints enforced

**Used by:** `Groth16VerifierB` (deployed on-chain)

---

## Security properties
- Privacy: amount, sender, and receiver are not linkable on‑chain
- Integrity: proofs enforce correct balance updates
- Unlinkability: network‑level correlation reduced (no timing/IP linkage in protocol design)
- One‑time spend: nullifiers prevent note reuse

## Contributing
Issues and PRs are welcome. If you’re proposing a protocol change, please include a short rationale and any security considerations.

**Last Updated:** October 26, 2025  
**Version:** 2.0-FINAL  
**Repository:** https://github.com/IMPERIAL-X7/3zio

## License
MIT © 3zio contributors

