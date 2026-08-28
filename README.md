<div align="center">

<img src="https://raw.githubusercontent.com/AxonOS-BCI/AxonOS-BCI/main/assets/axonos-neural-banner.svg" alt="AxonOS — real-time neural operating system for brain–computer interfaces" width="100%"/>

</div>

<br/>

**AxonOS is a hard real-time operating system for brain–computer interfaces.** Rust, `#![no_std]`, ARM Cortex-M. It occupies the layer between the electrode and the application, where neither Linux nor a stock RTOS holds the jitter a closed neural loop requires.

<div align="center">

[axonos.org](https://axonos.org) &nbsp;·&nbsp; [AxonOS-org](https://github.com/AxonOS-org) &nbsp;·&nbsp; [Engineering notes](https://medium.com/@AxonOS) &nbsp;·&nbsp; [Preprint](https://doi.org/10.5281/zenodo.20552007) &nbsp;·&nbsp; connect@axonos.org

</div>

<br/>

## Every figure is falsifiable, and says how

None of the numbers on this page are measurements. They are analytical bounds, proven with Kani and derived from datasheet cycle counts. On-hardware validation is pre-registered in [`axonos-validation`](https://github.com/AxonOS-org/axonos-validation). No measured figure is claimed until its raw trace lands there.

One file governs all of them. [`CLAIMS.md`](https://github.com/AxonOS-org/axonos-standard/blob/main/CLAIMS.md) records, for every published number, its evidence level, the artefact it is re-derived from, and the finding that would falsify it. Evidence levels are defined in [`VALIDATION.md`](https://github.com/AxonOS-org/axonos-standard/blob/main/VALIDATION.md): **L1** formally proven, **L2** measured on reference hardware, **L3** independently validated.

| | |
|:--|:--|
| **Current target** | Cortex-M4F, STM32F407, `thumbv7em-none-eabihf` |
| **Next target** | Cortex-M33 with TrustZone-M, STM32H573 |
| **Scheduling** | Earliest-Deadline-First, on biological deadlines |
| **WCRT** | ≤ 972 µs end to end, inside a 4 ms deadline. **L1**, Liu–Layland. |
| **Jitter** | 2.1 µs σ, 6.5 µs P99.9. Derived. On-hardware trace pending. |
| **Front end** | ADS1299, 8 channel, 24-bit |
| **Verification** | `#![forbid(unsafe_code)]` outside two documented, Kani-verified operations. 30 Kani harnesses. |
| **Discipline** | `#![no_std]`. No heap on the critical path. Zero-copy DMA into a static slab arena. |

Consent is enforced at Layer 2, below the coupling engine, so no higher cognitive layer can override a withdrawal. `Withdrawn` is terminal.

<br/>

## Ninety seconds

Reading this page is the slow way to judge it.

```sh
git clone https://github.com/AxonOS-org/axonos-stack && cd axonos-stack
cargo run --locked --bin session -- --seed 7 --frames 3000 | diff - reference/session-7.txt
```

Silence means the whole chain reproduced byte for byte on your machine: electrode, privacy vault, the right to act.

The session is not a happy path. An electrode lifts at 4.8 s, and the transcript records the system withdrawing the right to actuate 96 ms later while continuing to record. The moments a device stops trusting itself are the ones a clinician needs afterwards. The last line is an accounting identity, `delivered + lost = produced`. If it ever fails, one of the three components is lying about what it saw.

The kernel emits one 32-byte intent record. Every binding decodes it identically, and the wire format is the contract, cross-checked across Rust, Python, C, JavaScript and Java in [`axonos-conformance`](https://github.com/AxonOS-org/axonos-conformance).

```rust
let mut stream = IntentStream::connect(&manifest)?;   // ABI handshake, then data flows
while let Some(obs) = stream.poll() {
    if let IntentKind::Direction(dir) = obs.kind() {
        cursor.step(dir, obs.confidence_raw());       // u16 Q0.16, exact, never a float
    }
}
```

<br/>

## A claim that was wrong

```sh
gh api repos/AxonOS-BCI/axonos-community-radar/commits \
  --jq '.[0:10][] | "\(.commit.verification.verified)  \(.author.login)"'
```

This command is printed here because it recently returned `false`. The scanner published through a personal token, and GitHub signs API-created commits only for app identities, so the public map carried an unsigned history for six weeks. The fix removed the token's write access entirely: the scanner hands its payload over as a release asset, which creates no commit, and this repository's own workflow commits it under the identity that is signed.

The command stays in this README. A claim that was wrong once is worth leaving checkable.

<br/>

## The engineering substrate

[**AxonOS-org**](https://github.com/AxonOS-org) holds the kernel, the specification and everything the kernel contracts with. Every quantitative claim across these repositories is re-derived from an artefact and catalogued in `CLAIMS.md`.

| Layer | Repository | |
|:--|:--|:--|
| Kernel | [`axonos-kernel`](https://github.com/AxonOS-org/axonos-kernel) | EDF scheduling core, no heap on the critical path |
| Signal | [`axonos-signal-pipeline`](https://github.com/AxonOS-org/axonos-signal-pipeline) | ADS1299 to intent, zero-copy DMA |
| Consent | [`axonos-consent`](https://github.com/AxonOS-org/axonos-consent) | Layer-2 enforcement, `Withdrawn` is terminal |
| Protocol | [`axonos-protocol`](https://github.com/AxonOS-org/axonos-protocol) | The 32-byte intent record, mesh wire format |
| Swarm | [`axonos-swarm`](https://github.com/AxonOS-org/axonos-swarm) | Multi-device coordination over the mesh |
| Silicon | [`axonos-hal`](https://github.com/AxonOS-org/axonos-hal) | A timing budget that refuses a configuration it cannot meet |
| Privacy | [`axonos-vault`](https://github.com/AxonOS-org/axonos-vault) | Raw samples unreachable by construction |
| Posture | [`axonos-supervisor`](https://github.com/AxonOS-org/axonos-supervisor) | The right to act is withdrawn on signal quality. The right to record never is. |
| Integration | [`axonos-stack`](https://github.com/AxonOS-org/axonos-stack) | One session from a seed, transcript diffed in CI |
| Scoring | [`axonos-brs`](https://github.com/AxonOS-org/axonos-brs) | Integer relevance score a browser can recompute |
| Standard | [`axonos-standard`](https://github.com/AxonOS-org/axonos-standard) | Evidence levels, `CLAIMS.md`, `VALIDATION.md`, conformance tiers |
| Conformance | [`axonos-conformance`](https://github.com/AxonOS-org/axonos-conformance) | Byte-identical decoding across five languages |
| Validation | [`axonos-validation`](https://github.com/AxonOS-org/axonos-validation) | Pre-registered on-hardware traces |
| SDKs | [`axonos-sdk`](https://github.com/AxonOS-org/axonos-sdk) · [`python`](https://github.com/AxonOS-org/axonos-sdk-python) · [`swift`](https://github.com/AxonOS-org/axonos-sdk-swift) | Capability-gated client bindings |
| Governance | [`axonos-rfcs`](https://github.com/AxonOS-org/axonos-rfcs) | How the architecture is decided, in the open |

<br/>

## This account

AxonOS-org holds the law and the running core. This account holds what faces outward.

**[axonos-community-radar](https://github.com/AxonOS-BCI/axonos-community-radar)** is a scored map of the open BCI, neurotech and real-time-Rust field, rescanned from public GitHub every three hours. Every scan commits its own data, so the history is a git log rather than a claim. AxonOS appears on the map on the same terms as everyone else: same formula, no bonus for being ours.
[Live map](https://axonos-bci.github.io/axonos-community-radar/) · [Capability specification](https://axonos-bci.github.io/axonos-community-radar/premium.html) · [API](https://axonos-bci.github.io/axonos-community-radar/data/api.json)

**[neural-boundary-game](https://github.com/AxonOS-BCI/neural-boundary-game)** is a deterministic Rust and WASM model of the sovereignty architecture: consent, least-privilege scopes, the sealed vault, StimGuard. Playable in a browser, replayable byte for byte. It is the demo embedded on axonos.org.

**[axonos-boundary-run-v64](https://github.com/AxonOS-BCI/axonos-boundary-run-v64)** is a zero-telemetry cognitive-boundary simulator in pure JavaScript, with a SHA-256 replay proof re-checked in CI in both JavaScript and Python.

<sub>Earlier iterations (`axonos-boundary-run-v9`, `-v52`, `neural-boundary-game-play`) are kept as historical references and are being archived in favour of the two above.</sub>

<br/>

---

<div align="center">

Free and open. No paywalls, no ads, no tracking, no tokens.

Voluntary support: [support page](https://axonos-bci.github.io/axonos-community-radar/support.html) &nbsp;·&nbsp; `DMwHAhqVNWf7dyEznukxCufNS5rjuP5MTp` &nbsp;·&nbsp; [verify on-chain](https://dogechain.info/address/DMwHAhqVNWf7dyEznukxCufNS5rjuP5MTp)

<sub>Contributions are voluntary. They are not purchases or investments and carry no product entitlement. Commercial licensing is a separate written agreement, fiat by default: connect@axonos.org. Terms: <a href="https://github.com/AxonOS-BCI/neural-boundary-game/blob/main/CRYPTO_PAYMENT_TERMS.md">CRYPTO_PAYMENT_TERMS.md</a></sub>

<br/>

<sub><b>The AxonOS Project</b> &nbsp;·&nbsp; <a href="https://axonos.org">axonos.org</a> &nbsp;·&nbsp; connect@axonos.org &nbsp;·&nbsp; security@axonos.org</sub>

</div>
