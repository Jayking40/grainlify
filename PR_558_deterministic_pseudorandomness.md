Closes #558

## Changes

- **`contracts/grainlify-core/src/pseudo_randomness.rs`**
  - Added a deterministic pseudo-randomness helper module for verifiable on-chain selection.
  - Implemented seed derivation from:
    - domain tag
    - context bytes
    - external seed (`BytesN<32>`)
  - Implemented candidate selection by per-candidate hash scoring (instead of simple modulo), reducing index/order bias.
  - Documented security trade-offs and adversarial examples (seed grinding, timing bias, candidate stuffing).

- **`contracts/grainlify-core/src/lib.rs`**
  - Exported shared helper via `pub mod pseudo_randomness`.

- **`contracts/bounty_escrow/contracts/escrow/src/lib.rs`**
  - Integrated deterministic selection into claim-ticket flow with:
    - `derive_claim_ticket_winner_index(...)`
    - `derive_claim_ticket_winner(...)`
    - `issue_claim_ticket_deterministic(...)`
  - Added deterministic selection context construction using contract/bounty/amount/expiry/timestamp/ticket-counter material.
  - Added new error variant for invalid selection input.

- **`contracts/bounty_escrow/contracts/escrow/src/events.rs`**
  - Added `DeterministicSelectionDerived` event for verifiable auditability of:
    - selected index
    - candidate count
    - selected beneficiary
    - seed hash
    - winner score

- **`contracts/bounty_escrow/contracts/escrow/src/test_deterministic_randomness.rs`**
  - Added deterministic randomness tests:
    - same inputs => same winner
    - order-independent winner selection for the same candidate set
    - deterministic ticket issuance matches derived winner

## Testing

- `contracts/grainlify-core`
  - `cargo test --lib` ✅

- `contracts/bounty_escrow/contracts/escrow`
  - `cargo fmt --check --all` ✅
  - `cargo test test_deterministic_randomness -- --nocapture` ✅
  - `cargo test --lib` ✅
  - `cargo test --lib invariant_checker_ci` ✅

## Notes

- This is deterministic pseudo-randomness, not true randomness.
- Verifiability is prioritized: any observer can recompute the selected winner from the published inputs.
- Manipulation resistance depends on seed/context discipline; consumers should prefer commit-reveal style external seeds and bounded submission windows where applicable.
