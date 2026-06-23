<div align="center">

<img src="https://raw.githubusercontent.com/AxonOS-BCI/AxonOS-BCI/main/assets/axonos-neural-banner.svg" alt="AxonOS — real-time neural operating system for brain–computer interfaces" width="100%"/>

[axonos.org](https://axonos.org) &nbsp;·&nbsp; [Engineering notes](https://medium.com/@AxonOS) &nbsp;·&nbsp; connect@axonos.org

</div>

<br/>

**AxonOS** is a hard real-time operating system for brain–computer interfaces — the layer between the silicon and the application, where neither Linux nor a stock RTOS can hold the jitter and latency a closed neural loop demands. Written in Rust, `#![no_std]`, for ARMv8-M.

The guarantees below are measured on the target, not aspirational. They are encoded the same way they are enforced — in code.

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
    pub const TARGET:       &'static str = "thumbv8m.main-none-eabihf"; // Cortex-M33 + TrustZone-M
    pub const SCHEDULE:     Schedule     = Schedule::EarliestDeadlineFirst;
    pub const WCRT_NS:      u32          = 972_000; // worst-case response, sensor → intent
    pub const JITTER_NS:    u32          = 2_100;   // σ; Linux mainline ≈ 1_325_000 ns  (~630×)
    pub const HEAP_ON_PATH: bool         = false;   // zero-copy DMA into a static slab arena
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
| **Target** | Cortex-M33 + TrustZone-M (ARMv8-M) · `thumbv8m.main-none-eabihf` |
| **Scheduling** | Earliest-Deadline-First, biological deadlines |
| **Jitter** | 2.1 µs σ · 6.5 µs P99.9 — ≈630× tighter than Linux mainline |
| **WCRT** | 972 µs, sensor acquisition → classified intent, dual-core |
| **Front end** | ADS1299 · 8-channel · 24-bit |
| **Discipline** | `#![no_std]` · `#![forbid(unsafe_code)]` on the hot path · no heap on the critical path |

### One stream, every language

The kernel emits a single 32-byte intent record. Every binding decodes it byte-for-byte identically — the wire format *is* the contract.

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

### Repositories

Reference implementations live under [**AxonOS-org**](https://github.com/AxonOS-org): [axonos-sdk](https://github.com/AxonOS-org/axonos-sdk) · [axonos-sdk-swift](https://github.com/AxonOS-org/axonos-sdk-swift).

| Repository | Language | Description |
|:--|:--|:--|
| [**axonos-community-radar**](https://github.com/AxonOS-BCI/axonos-community-radar) | Python | AxonOS community radar for BCI, neurotechnology, real-time Rust, privacy and cognitive i |
| [**axonos-boundary-run-v52**](https://github.com/AxonOS-BCI/axonos-boundary-run-v52) | JavaScript | Playable, zero-telemetry cognitive-privacy game for AxonOS boundary semantics — with a d |
| [**neural-boundary-game**](https://github.com/AxonOS-BCI/neural-boundary-game) | Rust | AxonOS Boundary Console — deterministic Rust/WASM cognitive sovereignty runtime for neur |
| [**axonos-boundary-run-v64**](https://github.com/AxonOS-BCI/axonos-boundary-run-v64) | JavaScript | AxonOS Boundary Run v64 — The Sovereign Signal. Zero-telemetry cognitive privacy simulat |
| [**axonos-boundary-run-v9**](https://github.com/AxonOS-BCI/axonos-boundary-run-v9) | JavaScript | Boundary Run v9.0.1 — AxonOS serious browser game with SHA-256 replay proof and zero tel |
| [**neural-boundary-game-play**](https://github.com/AxonOS-BCI/neural-boundary-game-play) | HTML | AxonOS Education Boundary Run v8.8.4 — click RUN GAME and play instantly |
| [**AG0001**](https://github.com/AxonOS-BCI/AG0001) | — |  |

<sub>Engineering is documented end to end in the [AxonOS notes](https://medium.com/@AxonOS).</sub>

---

<div align="center">
<sub><b>The AxonOS Project</b> &nbsp;·&nbsp; <a href="https://axonos.org">axonos.org</a> &nbsp;·&nbsp; connect@axonos.org &nbsp;·&nbsp; security@axonos.org</sub>
</div>
