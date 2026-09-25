<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://github.com/AxonOS-BCI/AxonOS-BCI/raw/main/assets/founder-dark.svg">
  <img alt="Denis Yermakou — building the deterministic layer for brain–computer interfaces. Founder, AxonOS. Principal, DY Research." src="https://github.com/AxonOS-BCI/AxonOS-BCI/raw/main/assets/founder-light.svg" width="100%">
</picture>

**[axonos.org](https://axonos.org)** · **[AxonOS-org](https://github.com/AxonOS-org)** · **[dy-wcet](https://github.com/DYResearch/dy-wcet)** · **[Radar](https://axonos-bci.github.io/axonos-community-radar/)** · **[DY Research](https://dyresearch.github.io)** · **[Engineering notes](https://medium.com/@AxonOS)**

[![Kernel](https://img.shields.io/badge/kernel-Rust%20%C2%B7%20no__std-1f8fae?style=flat-square&labelColor=0d1117&logo=rust&logoColor=white)](https://github.com/AxonOS-org/axonos-kernel)
[![Kani](https://img.shields.io/badge/Kani-30%20kernel%20harnesses%20in%20CI-2ea043?style=flat-square&labelColor=0d1117)](https://github.com/AxonOS-org/axonos-standard/blob/main/VALIDATION.md)
[![Claims](https://img.shields.io/badge/every%20figure-CLAIMS.md-1f8fae?style=flat-square&labelColor=0d1117)](https://github.com/AxonOS-org/axonos-standard/blob/main/CLAIMS.md)
[![dy-wcet](https://img.shields.io/badge/dy--wcet-8%20Kani%20proofs%20in%20CI-2ea043?style=flat-square&labelColor=0d1117)](https://github.com/DYResearch/dy-wcet)
[![Preprint](https://img.shields.io/badge/preprint-Zenodo-1f8fae?style=flat-square&labelColor=0d1117)](https://doi.org/10.5281/zenodo.20552007)
[![Ecosystem pulse](https://img.shields.io/endpoint?url=https%3A%2F%2Faxonos-bci.github.io%2Faxonos-community-radar%2Fdata%2Fbadge-ecosystem.json&style=flat-square&labelColor=0d1117)](https://axonos-bci.github.io/axonos-community-radar/)

</div>

> **This is the founder surface of AxonOS: the live map of the field, and the demos.**
> The canonical engineering source is the [**AxonOS-org**](https://github.com/AxonOS-org) organisation.
> Every quantitative claim is governed by one file —
> [`CLAIMS.md`](https://github.com/AxonOS-org/axonos-standard/blob/main/CLAIMS.md) —
> which records each figure's evidence level, the artefact it is re-derived from, and the finding that would falsify it.

**AxonOS** is a hard real-time operating system for brain–computer interfaces: the
layer between the silicon and the application, where neither Linux nor a stock
RTOS can hold the jitter and latency a closed neural loop demands. Rust,
`#![no_std]`, ARM Cortex-M. Its contract is written the way it is enforced — in code.

```rust
#![no_std]
#![forbid(unsafe_code)] // in every kernel crate except axonos-spsc, where it is confined

//! AxonOS — a deterministic, hard real-time OS for brain–computer interfaces.
//! The layer between silicon and intent.

/// Deadlines are biological, not arbitrary.
pub enum Schedule {
    EarliestDeadlineFirst,
}

/// Consent is enforced at Layer 2 (Connection), below the coupling engine,
/// so no layer above it can override a withdrawal. `Withdrawn` is terminal.
pub enum Consent {
    Granted,
    Suspended,
    Withdrawn,
}

/// The real-time contract. Every figure below is a row in CLAIMS.md.
pub struct Kernel;

impl Kernel {
    pub const TARGET_CURRENT: &'static str = "thumbv7em-none-eabihf";    // Cortex-M4F · STM32F407 · boots here
    pub const TARGET_NEXT:    &'static str = "thumbv8m.main-none-eabihf"; // Cortex-M33 + TrustZone-M · planned
    pub const SCHEDULE:       Schedule     = Schedule::EarliestDeadlineFirst;
    pub const WCRT_BOUND_NS:  u32          = 1_000_000; // C-1   · L1 · proven upper bound, end to end
    pub const WCRT_SEEN_NS:   u32          =   972_000; // C-1·L2 · worst observed, 12 h soak · trace pending
    pub const JITTER_NS:      u32          =     2_100; // C-2   · L2 · σ · trace pending
    pub const HEAP_ON_PATH:   bool         = false;     // zero-copy DMA into a static slab arena
}

/// The kernel exposes exactly one typed, capability-gated event stream.
pub trait IntentStream {
    fn poll(&mut self) -> Option<Observation>;
    fn consent(&self) -> Consent;
}
```

### What the kernel holds

| | |
|:--|:--|
| **Current target** | Cortex-M4F · STM32F407 · `thumbv7em-none-eabihf` — the reference firmware boots here |
| **Next target** | Cortex-M33 + TrustZone-M · STM32H573 · `thumbv8m.main-none-eabihf` — planned |
| **Scheduling** | Earliest-Deadline-First, against biological deadlines |
| **Response time** | ≤ 1,000 µs end to end, proven — **C-1 · L1** · 972 µs the worst observed over a 12 h soak — **C-1·L2**, trace pending |
| **Jitter** | 2.1 µs σ — **C-2 · L2**, trace pending |
| **Consent** | Withdrawal proven to terminate in the correct state — **C-4·L1**, from `Granted` · ≤ 1,648 cycles — analytical |
| **Front end** | ADS1299 · 8 channels · 24-bit |
| **Memory safety** | `unsafe` confined to `axonos-spsc`; `#![forbid(unsafe_code)]` everywhere else in the kernel |
| **Verification** | 30 Kani harnesses in the kernel, re-run in CI |
| **Discipline** | `#![no_std]` · no heap on the critical path · zero-copy DMA |

<sub>Evidence levels — **L1** formally proven · **L2** measured on reference hardware · **L3** independently reproduced · **analytical** derived by hand from a reference — are defined in [`VALIDATION.md`](https://github.com/AxonOS-org/axonos-standard/blob/main/VALIDATION.md) and catalogued in [`CLAIMS.md`](https://github.com/AxonOS-org/axonos-standard/blob/main/CLAIMS.md). No measured figure is claimed until its raw trace lands in [`axonos-validation`](https://github.com/AxonOS-org/axonos-validation). L3 is claimed for nothing. The peer-readable derivation is the [Zenodo preprint](https://doi.org/10.5281/zenodo.20552007): analytical, falsifiable, no measurement claims.</sub>

### In focus · dy-wcet

**Timing analysis that refuses rather than rounds.** Worst-case response time in
integer arithmetic, and a named refusal wherever a bound cannot be justified — the
AxonOS timing work as a standalone crate, at [DY Research](https://dyresearch.github.io).

**0** dependencies, no floating point · **8** Kani proofs, every one closing in CI ·
**100** tests, fifteen derived by hand · **6** named refusals

**[Try it live →](https://dyresearch.github.io/wcet/)** · [Source](https://github.com/DYResearch/dy-wcet) · [The method](https://gist.github.com/AxonOS-BCI/3bef2ff217a3ece45e4958fac2c16c4a)

### One stream, every language

The kernel emits a single 32-byte intent record, and the wire format *is* the
contract: the reference codec and the Rust SDK decode it identically, checked on
every push, and the C header is checked against it by the compiler — in
[`axonos-conformance`](https://github.com/AxonOS-org/axonos-conformance), alongside
bindings for Java, JavaScript and assembly.

```rust
use axonos_sdk::{Capability, IntentKind, IntentStream, Manifest};

let manifest = Manifest::builder()
    .app_id("org.axonos.cursor")?
    .capability(Capability::Navigation)   // kernel caps delivery at 50 Hz
    .max_rate_hz(50)
    .build()?;

let mut stream = IntentStream::connect(&manifest)?; // ABI handshake, then data flows
while let Some(obs) = stream.poll() {
    if let IntentKind::Direction(dir) = obs.kind() {
        cursor.step(dir, obs.confidence_raw());       // u16 Q0.16 — exact, never a float
    }
}
```

### The engineering substrate — [AxonOS-org](https://github.com/AxonOS-org)

| Layer | Repository | What it holds |
|:--|:--|:--|
| **Kernel** | [`axonos-kernel`](https://github.com/AxonOS-org/axonos-kernel) | The hard real-time core · EDF · `#![no_std]` · no heap on the critical path |
| **Signal** | [`axonos-signal-pipeline`](https://github.com/AxonOS-org/axonos-signal-pipeline) | ADS1299 to intent, deterministically · zero-copy DMA into a static slab arena |
| **Consent** | [`axonos-consent`](https://github.com/AxonOS-org/axonos-consent) | Layer-2 enforcement below the coupling engine · `Withdrawn` is terminal |
| **Protocol** | [`axonos-protocol`](https://github.com/AxonOS-org/axonos-protocol) | The wire format and the 32-byte intent record |
| **Swarm** | [`axonos-swarm`](https://github.com/AxonOS-org/axonos-swarm) | Multi-device coordination over the protocol |
| **Silicon** | [`axonos-hal`](https://github.com/AxonOS-org/axonos-hal) | A timing budget that *refuses* a configuration whose deadline the measured chain cannot meet |
| **Privacy** | [`axonos-vault`](https://github.com/AxonOS-org/axonos-vault) | Raw samples unreachable by construction; only bounded, purpose-bound reductions leave, each recorded |
| **Posture** | [`axonos-supervisor`](https://github.com/AxonOS-org/axonos-supervisor) | The right to *act* is withdrawn on signal quality; the right to *record* never is |
| **Integration** | [`axonos-stack`](https://github.com/AxonOS-org/axonos-stack) | The layers running as one session from a seed, diffed byte for byte in CI |
| **Scoring** | [`axonos-brs`](https://github.com/AxonOS-org/axonos-brs) | The score behind the Radar, in integer arithmetic a browser can recompute |
| **Standard** | [`axonos-standard`](https://github.com/AxonOS-org/axonos-standard) | The specification, the evidence levels, `CLAIMS.md`, `VALIDATION.md` |
| **Conformance** | [`axonos-conformance`](https://github.com/AxonOS-org/axonos-conformance) | Test vectors and bindings for the wire format |
| **Validation** | [`axonos-validation`](https://github.com/AxonOS-org/axonos-validation) | Pre-registered on-hardware validation and its traces |
| **SDKs** | [`axonos-sdk`](https://github.com/AxonOS-org/axonos-sdk) · [`python`](https://github.com/AxonOS-org/axonos-sdk-python) · [`swift`](https://github.com/AxonOS-org/axonos-sdk-swift) | Capability-gated client bindings |
| **Governance** | [`axonos-rfcs`](https://github.com/AxonOS-org/axonos-rfcs) | Design proposals, and the record of how the architecture is decided |

### Doubt it in ninety seconds

The fastest way to judge any of this is not to read it.

```sh
# the whole chain from a seed: electrode -> privacy vault -> the right to act
git clone https://github.com/AxonOS-org/axonos-stack && cd axonos-stack
cargo run --locked --bin session -- --seed 7 --frames 3000 | diff - reference/session-7.txt
```

Silence means it reproduced byte for byte on your machine. The session is not a
happy path: an electrode lifts at 4.8 s and the transcript records the system
withdrawing the right to actuate 96 ms later, while continuing to record —
because the moments a device stops trusting itself are the ones a clinician
needs afterwards. Its last line is an accounting identity, *delivered + lost =
produced*, and if it ever fails, one of the three components is lying about what
it saw.

Then check who wrote the automated commits on the live map, and whether GitHub
signed them:

```sh
gh api repos/AxonOS-BCI/axonos-community-radar/commits \
  --jq '.[0:10][] | "\(.commit.verification.verified)  \(.author.login)"'
```

That one is here because it once read `false`. The scanner published through a
personal token, and GitHub signs API-created commits only for app identities, so
the public map's history was unsigned for six weeks. The fix was to stop the
token writing at all: the scanner now hands its payload over as a release asset,
which creates no commit, and the repository's own workflow commits it under the
identity that *is* signed. The command stays because a claim that was wrong once
is worth leaving checkable.

### This account

The founder half of one project: [AxonOS-org](https://github.com/AxonOS-org#the-ecosystem)
holds the specification and the running core; this account holds the live map of
the field and the playable demos.

| Repository | What it is |
|:--|:--|
| [**axonos-community-radar**](https://github.com/AxonOS-BCI/axonos-community-radar) · [live](https://axonos-bci.github.io/axonos-community-radar/) | A living map of open neurotech: refreshed from GitHub every three hours, scored by evidence tier, with zero runtime dependencies |
| [**neural-boundary-game**](https://github.com/AxonOS-BCI/neural-boundary-game) | The canonical demo: a deterministic Rust/WASM model of the AxonOS sovereignty architecture — consent, least-privilege scopes, sealed privacy vault, StimGuard — replayable byte for byte, and embedded on axonos.org |
| [**axonos-boundary-run-v64**](https://github.com/AxonOS-BCI/axonos-boundary-run-v64) | *The Sovereign Signal*: a zero-telemetry browser game whose SHA-256 replay proof is re-checked in CI, in JavaScript and in Python |

<sub>Earlier iterations — `axonos-boundary-run-v9`, `axonos-boundary-run-v52`, `neural-boundary-game-play` — are kept as historical references.</sub>

### Fund the work

AxonOS is open, and it is funded by the work it makes possible.

**Commission an engagement.** [DY Research](https://dyresearch.github.io) carries out
independent technical due diligence — Snapshot, Focused Audit, Due Diligence — at a
fixed price from $5,000, and its revenue funds AxonOS.
**[Engagements and scope →](https://dyresearch.github.io/#engagements)**

**Contribute voluntarily**, in Dogecoin, to `DMwHAhqVNWf7dyEznukxCufNS5rjuP5MTp` —
[support page](https://axonos-bci.github.io/axonos-community-radar/support.html) ·
[verify on-chain](https://dogechain.info/address/DMwHAhqVNWf7dyEznukxCufNS5rjuP5MTp).

<sub>Contributions are voluntary: not purchases, not investments, and no product entitlement. Commercial licensing is a separate written agreement, in fiat by default: connect@axonos.org. Terms: [CRYPTO_PAYMENT_TERMS.md](https://github.com/AxonOS-BCI/neural-boundary-game/blob/main/CRYPTO_PAYMENT_TERMS.md).</sub>

---

<div align="center">

© The AxonOS Project / Denis Yermakou

[axonos.org](https://axonos.org) · [connect@axonos.org](mailto:connect@axonos.org) · [security@axonos.org](mailto:security@axonos.org) · [LinkedIn](https://www.linkedin.com/in/axonos)

</div>
