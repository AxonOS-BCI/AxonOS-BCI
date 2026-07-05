<div align="center">

<img src="https://raw.githubusercontent.com/AxonOS-BCI/AxonOS-BCI/main/assets/axonos-neural-banner.svg" alt="AxonOS — real-time neural operating system for brain–computer interfaces" width="100%"/>

[axonos.org](https://axonos.org) &nbsp;·&nbsp; [Engineering notes](https://medium.com/@AxonOS) &nbsp;·&nbsp; connect@axonos.org

<br/>

[![Kernel](https://img.shields.io/badge/kernel-Rust%20%C2%B7%20no__std-000?logo=rust&logoColor=white)](https://github.com/AxonOS-org/axonos-kernel)
[![Verification](https://img.shields.io/badge/verification-Kani%20BMC%20%C2%B7%2030%20harnesses-34d399)](https://github.com/AxonOS-org/axonos-standard/blob/main/VALIDATION.md)
[![Claims](https://img.shields.io/badge/every%20figure-governed%20by%20CLAIMS.md-0a4a8f)](https://github.com/AxonOS-org/axonos-standard/blob/main/CLAIMS.md)
[![Preprint](https://img.shields.io/badge/preprint-Zenodo-1682D4)](https://doi.org/10.5281/zenodo.20552007)
[![Live radar](https://img.shields.io/badge/live-BCI%20ecosystem%20radar-2dd4ff)](https://axonos-bci.github.io/axonos-community-radar/)

</div>

<br/>

> **This is the founder, demo, and community surface for AxonOS.**
> The canonical engineering source of truth is the [**AxonOS-org**](https://github.com/AxonOS-org) organisation.
> Every quantitative performance and validation claim is governed by one file —
> [`axonos-standard/CLAIMS.md`](https://github.com/AxonOS-org/axonos-standard/blob/main/CLAIMS.md) —
> which records, for each figure, its evidence level, the artefact it is re-derived from, and the finding that would falsify it.

<br/>

**AxonOS** is a hard real-time operating system for brain–computer interfaces — the layer between the silicon and the application, where neither Linux nor a stock RTOS can hold the jitter and latency a closed neural loop demands. Written in Rust, `#![no_std]`, for ARM Cortex-M.

The figures below are **analytical bounds, proven with Kani and derived from datasheet cycle counts** — not measurements. On-hardware validation is pre-registered and publication-pending in [`axonos-validation`](https://github.com/AxonOS-org/axonos-validation); no measured number is claimed until its raw trace is published. They are encoded the same way they are enforced — in code.

```rust
#![no_std]
#![forbid(unsafe_code)]

//! AxonOS — a deterministic, hard real-time OS for brain–computer interfaces.
//! The layer between silicon and intent.

/// Deadlines are biological, not arbitrary.
pub enum Schedule {
    EarliestDeadlineFirst,
}

/// Consent is enforced at Layer 2 (Connection) — below the coupling engine —
/// so no higher cognitive layer can override a withdrawal. `Withdrawn` is terminal.
pub enum Consent {
    Granted,
    Suspended,
    Withdrawn,
}

/// The real-time contract, held on every cycle of the dual-core pipeline.
pub struct Kernel;

impl Kernel {
    // Current reference bring-up; the firmware crate boots here.
    pub const TARGET_CURRENT: &'static str = "thumbv7em-none-eabihf"; // Cortex-M4F · STM32F407
    // Documented next target for the secure Cognitive Hypervisor.
    pub const TARGET_NEXT:    &'static str = "thumbv8m.main-none-eabihf"; // Cortex-M33 + TrustZone-M · STM32H573
    pub const SCHEDULE:       Schedule     = Schedule::EarliestDeadlineFirst;
    pub const WCRT_NS:        u32          = 972_000; // L1 analytical bound (Liu–Layland EDF), inside a 4 ms deadline
    pub const JITTER_NS:      u32          = 2_100;   // σ, derived; on-hardware L2 trace publication-pending
    pub const HEAP_ON_PATH:   bool         = false;   // zero-copy DMA into a static slab arena
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
| **Current target** | Cortex-M4F · STM32F407 · `thumbv7em-none-eabihf` (reference firmware boots here) |
| **Next target** | Cortex-M33 + TrustZone-M (ARMv8-M) · STM32H573 · `thumbv8m.main-none-eabihf` |
| **Scheduling** | Earliest-Deadline-First, biological deadlines |
| **WCRT** | ≤ 972 µs end-to-end, inside a 4 ms deadline — **L1 analytical bound** (Liu–Layland EDF) |
| **Jitter** | 2.1 µs σ · 6.5 µs P99.9 — **derived**; on-hardware trace publication-pending |
| **Front end** | ADS1299 · 8-channel · 24-bit |
| **Verification** | `#![forbid(unsafe_code)]` outside two documented, Kani-verified `unsafe` operations · 30 Kani BMC harnesses |
| **Discipline** | `#![no_std]` · no heap on the critical path · zero-copy DMA |

<sub>Evidence levels (**L1** formally proven · **L2** measured on reference hardware · **L3** independently validated) and the falsifiability of each figure are defined in [`axonos-standard/VALIDATION.md`](https://github.com/AxonOS-org/axonos-standard/blob/main/VALIDATION.md) and catalogued in [`CLAIMS.md`](https://github.com/AxonOS-org/axonos-standard/blob/main/CLAIMS.md). The peer-readable derivation is the [Zenodo preprint](https://doi.org/10.5281/zenodo.20552007) — analytical, falsifiable, **no measurement claims**.</sub>

### One stream, every language

The kernel emits a single 32-byte intent record. Every binding decodes it byte-for-byte identically — the wire format *is* the contract, cross-checked across Rust, Python, C, JavaScript, and Java in [`axonos-conformance`](https://github.com/AxonOS-org/axonos-conformance).

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

The kernel and everything it contracts with. Every quantitative claim across these repositories is re-derived from an artefact and catalogued in [`CLAIMS.md`](https://github.com/AxonOS-org/axonos-standard/blob/main/CLAIMS.md); nothing here is asserted without a falsifiable source.

| Layer | Repository | What it holds |
|:--|:--|:--|
| **Kernel** | [`axonos-kernel`](https://github.com/AxonOS-org/axonos-kernel) | Hard real-time scheduling core · EDF · `#![no_std]` · zero heap on the critical path |
| **Signal** | [`axonos-signal-pipeline`](https://github.com/AxonOS-org/axonos-signal-pipeline) | Deterministic ADS1299 → intent DSP pipeline · zero-copy DMA into a static slab arena |
| **Consent** | [`axonos-consent`](https://github.com/AxonOS-org/axonos-consent) | Layer-2 consent enforcement · `Withdrawn` is terminal · below the coupling engine |
| **Protocol** | [`axonos-protocol`](https://github.com/AxonOS-org/axonos-protocol) | BCI-to-BCI mesh wire format · the 32-byte intent record · reference implementation |
| **Swarm** | [`axonos-swarm`](https://github.com/AxonOS-org/axonos-swarm) | Multi-device coordination over the mesh protocol |
| **Standard** | [`axonos-standard`](https://github.com/AxonOS-org/axonos-standard) | The specification, evidence levels (L1/L2/L3), `CLAIMS.md`, `VALIDATION.md`, conformance tiers |
| **Conformance** | [`axonos-conformance`](https://github.com/AxonOS-org/axonos-conformance) | The wire format *is* the contract — decoded byte-for-byte identically across Rust, Python, C, JS, Java |
| **Validation** | [`axonos-validation`](https://github.com/AxonOS-org/axonos-validation) | Pre-registered on-hardware validation · no measured figure is claimed until its raw trace lands here |
| **SDKs** | [`axonos-sdk`](https://github.com/AxonOS-org/axonos-sdk) · [`·python`](https://github.com/AxonOS-org/axonos-sdk-python) · [`·swift`](https://github.com/AxonOS-org/axonos-sdk-swift) | Capability-gated client bindings — one intent stream, every language |
| **Governance** | [`axonos-rfcs`](https://github.com/AxonOS-org/axonos-rfcs) | Design proposals and the record of how the architecture is decided |

<sub>**Evidence levels** — <b>L1</b> formally proven (Kani / analytical bound) · <b>L2</b> measured on reference hardware · <b>L3</b> independently validated. The mapping from every published figure to its evidence level and its falsifier is the single source of truth in [`CLAIMS.md`](https://github.com/AxonOS-org/axonos-standard/blob/main/CLAIMS.md).</sub>

### This account — founder, demos & community



| Repository | Activity | What it is |
|:--|:--|:--|
| [**neural-boundary-game**](https://github.com/AxonOS-BCI/neural-boundary-game) | ![last](https://img.shields.io/github/last-commit/AxonOS-BCI/neural-boundary-game?label=&style=flat-square&color=34d399) | **Canonical interactive demo.** Deterministic Rust/WASM model of the AxonOS sovereignty architecture — consent, least-privilege scopes, sealed privacy vault, StimGuard — playable in-browser, byte-for-byte replayable. The demo embedded on axonos.org. |
| [**axonos-boundary-run-v64**](https://github.com/AxonOS-BCI/axonos-boundary-run-v64) | ![last](https://img.shields.io/github/last-commit/AxonOS-BCI/axonos-boundary-run-v64?label=&style=flat-square&color=34d399) | Latest pure-JS browser game — *The Sovereign Signal*. Zero-telemetry cognitive-boundary simulator with a deterministic, independently verifiable SHA-256 replay proof, re-checked in CI in both JavaScript and Python. |

<br/>

<table>
<tr><td width="60" align="center" valign="top">

### 🛰️

</td><td valign="top">

**[axonos-community-radar](https://github.com/AxonOS-BCI/axonos-community-radar)** &nbsp; [![live](https://img.shields.io/badge/live-axonos--bci.github.io-2dd4ff?style=flat-square)](https://axonos-bci.github.io/axonos-community-radar/) &nbsp; ![last](https://img.shields.io/github/last-commit/AxonOS-BCI/axonos-community-radar?label=&style=flat-square&color=34d399)

A **living map of the open BCI / neurotech / real-time-Rust ecosystem** — refreshed from GitHub every three hours, relevance-filtered by evidence tier, zero runtime dependencies. It now surfaces the AxonOS ecosystem itself: an *"AxonOS ecosystem"* panel mapping the maintaining org, the people building each repository, and the shared-maintainer links between them — every fact live from the public GitHub API, none fabricated.

</td></tr>
</table>

<sub>Earlier game iterations — `axonos-boundary-run-v9`, `axonos-boundary-run-v52`, `neural-boundary-game-play` — are kept as historical references and are being archived in favour of the two demos above. Engineering is documented end to end in the [AxonOS notes](https://medium.com/@AxonOS).</sub>

---

<div align="center">
<sub><b>The AxonOS Project</b> &nbsp;·&nbsp; <a href="https://axonos.org">axonos.org</a> &nbsp;·&nbsp; connect@axonos.org &nbsp;·&nbsp; security@axonos.org</sub>
</div>
