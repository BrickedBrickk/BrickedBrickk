<div align="center">

# BrickedBrick

**systems · GPUs · protocols · browser ports**

I work on low-level software: GPU compute and mining stacks, proof-of-work design,
binary network protocols, and native code pushed into environments it was never
meant to run in.

<br/>

[![OpenCL](https://img.shields.io/badge/OpenCL-ED271D)](https://github.com/BrickedBrickk/Pearl-BC160)
[![WGSL](https://img.shields.io/badge/WGSL-005A9C?logo=webgpu&logoColor=white)](https://github.com/BrickedBrickk/QelloxHashV1)
[![WebRTC](https://img.shields.io/badge/WebRTC-005A9C?logo=webrtc&logoColor=white)](https://github.com/BrickedBrickk/Eagler-Relay-Client)
[![Rust](https://img.shields.io/badge/Rust-DEA584?logo=rust&logoColor=black)](https://github.com/BrickedBrickk?tab=repositories&q=&type=&language=rust)
[![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?logo=typescript&logoColor=white)](https://github.com/BrickedBrickk?tab=repositories&q=&type=&language=typescript)

</div>

---

## About

I spend most of my time under the abstractions — kernels, packet formats, VMs,
hash pipelines, container bytecode, driver targets. The recurring theme is the
same: take something that officially does not support a platform, a vendor, or a
runtime, and make it work there anyway with the evidence to back it up.

That shows up as porting GPU miners onto unusual AMD silicon, writing proof-of-work
algorithms from a frozen spec with CPU/GPU equivalence tests, implementing binary
relay protocols from scratch in the browser, and instrumenting game internals with
mixins. I care about determinism, test vectors, honest status labels, and docs that
an independent engineer can actually implement from.

---

## What I build

| Area | What that means here |
| --- | --- |
| **GPU compute & mining** | OpenCL kernels on RDNA1 (`gfx1011`), BLAKE3 on-device, Stratum clients, CPU reference paths that must match the GPU bit-for-bit |
| **Proof-of-work research** | Custom VM-based PoW, frozen candidate manifests, formal specs, fuzzing, architecture simulators, hardware qualification harnesses |
| **Browser-native networking** | Hand-rolled binary codecs, WebSocket signaling, WebRTC ICE + data channels, dependency-free TypeScript |
| **Modding & game tooling** | Fabric mixins, container-level instrumentation, Gradle/CI release pipelines across Minecraft version migrations |

---

## Featured work

### [Pearl-BC160](https://github.com/BrickedBrickk/Pearl-BC160)

![status](https://img.shields.io/badge/status-experimental-orange)
![license](https://img.shields.io/badge/license-Apache--2.0-D22128?logo=apache&logoColor=white)
![BC-160](https://img.shields.io/badge/AMD%20BC--160-E01E26?logo=amd&logoColor=white)
![PoUW](https://img.shields.io/badge/PoUW-Proof%20of%20Useful%20Work-007ACC?logo=ethereum&logoColor=white)

Open-source **Pearl (PRL) GPU miner for the AMD BC-160** (Navi 12 / RDNA1 / `gfx1011`) —
a headless mining card with no existing Pearl backend.

The interesting part is the stack: PearlHash is proof-of-useful-work built on int8
GEMM with cryptographic noise injection, transcript accumulation, and keyed BLAKE3
finalization. This repo traces that pipeline from upstream source into a faithful
CPU reference, then re-implements the compute half as an OpenCL kernel (in-kernel
BLAKE3 compression included), with Stratum v1 pool plumbing around it and a
`GpuBackend` trait so OpenCL / HIP / null backends stay swappable.

**Status:** experimental. Algorithm, pool code, serialization, and kernel exist with
tests and benches; **the kernel has not yet been run on BC-160 hardware**, so there
are no verified hashrates. First-hardware validation scripts are in `tools/`.

```text
Rust · OpenCL C · BLAKE3 · Stratum v1 · criterion · self-test / verify-gpu CLI
```

---

### [QelloxHashV1](https://github.com/BrickedBrickk/QelloxHashV1)

![status](https://img.shields.io/badge/status-frozen%20candidate-blue)
![license](https://img.shields.io/badge/license-Apache--2.0-D22128?logo=apache&logoColor=white)
![WGSL](https://img.shields.io/badge/WGSL-005A9C?logo=webgpu&logoColor=white)
![tests](https://img.shields.io/badge/test%20vectors-2E7D32)

A **consumer-GPU proof-of-work algorithm** designed from scratch for equal AMD/NVIDIA
behavior and economic resistance to specialized hardware — not a fork of an existing
hash.

Each nonce seeds a deterministic **32-lane × 32-register integer VM** that executes a
unique **112-instruction program** (23 opcodes) over a 48 KiB scratchpad, with
**8 serial dependent reads per pass** specifically to blunt HBM bandwidth advantages,
cross-lane butterfly mixing, and domain-separated SHA-256 finalization. The repo ships
a formal specification, a safe-Rust CPU reference, a matching **wgpu/WGSL** backend,
canonical test vectors, unit + fuzz tests, an architecture simulator for candidate
scoring, and a qualification CLI whose CI pins the frozen manifest by SHA-256.

**Status:** v0.9.0 prerelease — algorithm **frozen** as Candidate A for research and
hardware qualification. **No physical hardware qualification yet**; performance
discussion in the docs is explicitly modeled, not measured. Not active consensus.

```text
Rust · WGSL · wgpu · SHA-256 · fuzzing · architecture simulation · GitHub Actions
```

---

### [Eagler-Relay-Client](https://github.com/BrickedBrickk/Eagler-Relay-Client)

![stars](https://img.shields.io/github/stars/BrickedBrickk/Eagler-Relay-Client?logo=github)
![license](https://img.shields.io/badge/license-AGPL--3.0-007ACC)
![deps](https://img.shields.io/badge/dependency--free-333333)

Independent **browser client for version 1 of the Eaglercraft shared-world relay
protocol** — hosting, joining, relay discovery, and full packet encode/decode with
**zero runtime dependencies**.

The work is the protocol surface: a hand-written binary codec over WebSocket frames,
then a WebRTC signaling state machine (offers/answers, ICE candidate exchange, `lan`
data channels) that lets a page host or join a shared world without shipping the
upstream client. Round-trip tests cover every packet type plus truncated-frame
rejection.

```text
TypeScript · WebSocket · WebRTC/ICE · binary protocols · Node tests · AGPL-3.0
```

---

### [ItemFlow](https://github.com/BrickedBrickk/ItemFlow)

![release](https://img.shields.io/github/v/release/BrickedBrickk/ItemFlow?logo=github&label=release)
![Fabric](https://img.shields.io/badge/Fabric%20Mod-007ACC)
![license](https://img.shields.io/badge/license-MIT-007A5D?logo=open-source-initiative&logoColor=white)

Server-side **Fabric mod** that records container transactions by snapshotting
inventories on open/close boundaries — chests, barrels, shulker boxes, breaks, and
hopper placements — into per-container logs with no commands or config.

Technically it is Mixin instrumentation into block-entity lifecycle methods, kept
building across Minecraft and Java migrations (now Java 25 / MC 26.3) with Gradle,
wrapper validation, and published releases from `v1.0.0` through `v1.0.3`. Fork of
the ChestLogs lineage, actively maintained.

```text
Java · Fabric · Mixin · Gradle · GitHub Actions · release automation
```

---

<details>
<summary><b>Toolbox</b> — languages, runtimes, and tooling used across the repos above</summary>

<br/>

<img src="https://skillicons.dev/icons?i=rust,ts,java,c,linux,git,githubactions,gradle,nodejs" alt="Rust, TypeScript, Java, C, Linux, Git, GitHub Actions, Gradle, Node.js" />

![OpenCL](https://img.shields.io/badge/OpenCL-ED271D)
![WGSL](https://img.shields.io/badge/WGSL-005A9C?logo=webgpu&logoColor=white)
![WebGPU](https://img.shields.io/badge/WebGPU-005A9C?logo=webgpu&logoColor=white)
![wgpu](https://img.shields.io/badge/wgpu-007ACC?logo=webgpu&logoColor=white)
![WebRTC](https://img.shields.io/badge/WebRTC-005A9C?logo=webrtc&logoColor=white)
![WebSocket](https://img.shields.io/badge/WebSocket-010101?logo=socketdotio&logoColor=white)
![BLAKE3](https://img.shields.io/badge/BLAKE3-000000)
![SHA-256](https://img.shields.io/badge/SHA--256-000000)
![Stratum](https://img.shields.io/badge/Stratum%20v1-FF5733?logo=ethereum&logoColor=white)
![gfx1011](https://img.shields.io/badge/gfx1011-RDNA1-E01E26?logo=amd&logoColor=white)
![Mixin](https://img.shields.io/badge/Mixin-007ACC)
![criterion](https://img.shields.io/badge/criterion-DEA584?logo=rust&logoColor=black)
![fuzzing](https://img.shields.io/badge/fuzzing-FF6F00)
![CI](https://img.shields.io/badge/CI-GitHub%20Actions-2088FF?logo=githubactions&logoColor=white)
![Rust 1.81+](https://img.shields.io/badge/Rust%201.81%2B-000000?logo=rust&logoColor=white)
![Node 18+](https://img.shields.io/badge/Node%2018%2B-339933?logo=nodedotjs&logoColor=white)
![Java 25](https://img.shields.io/badge/Java%2025-ED8B00?logo=openjdk&logoColor=white)

</details>

---

## Currently

- **Pearl-BC160** — first BC-160 hardware pass: OpenCL compile on `gfx1011`, CPU↔GPU
  output comparison, then pool/share validation. No results are claimed until that runs.
- **QelloxHashV1** — hardware qualification against the frozen Candidate A manifest;
  independent implementations and measured (not modeled) results welcome per the
  contributing rules.
- **Browser & protocol work** — keeping the relay client clean as a standalone,
  dependency-free library.

**Interested in:** emulation and porting, reverse engineering, systems programming,
GPU compute, AI inference on odd hardware, browser technology, networking, and
performance work — especially anything people assume *cannot* run in a given
environment.

---

## Links

[![GitHub](https://img.shields.io/badge/GitHub-BrickedBrickk-181717?logo=github&logoColor=white)](https://github.com/BrickedBrickk)
[![YouTube](https://img.shields.io/badge/YouTube-@BrickedBrickk-FF0000?logo=youtube&logoColor=white)](https://www.youtube.com/@BrickedBrickk)
