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

| Repository | Purpose |
|:--|:--|
| [**axonos-sdk**](https://github.com/AxonOS-org/axonos-sdk) | Reference SDK for the typed intent stream — Rust |
| [**axonos-sdk-swift**](https://github.com/AxonOS-org/axonos-sdk-swift) | Pure-Swift binding decoding the identical wire format |
| **kernel · pipeline · consent** | Hard real-time core and protocol-level consent — Rust, `#![no_std]` |

<sub>Engineering is documented end to end in the [AxonOS notes](https://medium.com/@AxonOS).</sub>

---

<div align="center">
<sub><b>The AxonOS Project</b> &nbsp;·&nbsp; <a href="https://axonos.org">axonos.org</a> &nbsp;·&nbsp; connect@axonos.org &nbsp;·&nbsp; security@axonos.org</sub>
</div>
