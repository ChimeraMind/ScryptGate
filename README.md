# ⛏️ ScryptGate

### Repurposed Mining ASIC as AI Agent Security Primitive

> *A framework for using retired cryptographic hashing hardware as dedicated AI security coprocessors.*

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Status: Active Research](https://img.shields.io/badge/Status-Active%20Research-green.svg)]()
[![First Published: March 22 2026](https://img.shields.io/badge/First%20Published-March%2022%202026-orange.svg)]()
[![Contributions Welcome](https://img.shields.io/badge/Contributions-Welcome-brightgreen.svg)]()

-----

> **Prior Art Notice:** No existing published work on cryptographic mining ASIC  
> repurposing for AI security primitives was found at time of writing.  
> This repository establishes that record. First published: **March 22, 2026.**

-----

## The Idea In One Paragraph

Billions of dollars worth of specialized cryptographic hardware is sitting unused in garages, warehouses, and secondhand markets as mining profitability shifts. These are not e-waste. They are **dedicated cryptographic compute units** — purpose-built silicon running specific hash functions at speeds no CPU or GPU can match per watt. Meanwhile, AI agent stacks running on software-only authentication, software rate limiting, and software model integrity checks are trivially attackable. This project asks a simple question nobody has formally asked before:

**What happens when you stop thinking of a mining ASIC as a miner and start thinking of it as a cryptographic coprocessor for AI security?**

The answer is a new class of hardware-rooted AI primitive that costs under $100, consumes a fraction of a GPU’s power for its specific role, and provides security guarantees that software architecturally cannot provide regardless of how well it is written.

-----

## Table of Contents

- [Why This Matters](#why-this-matters)
- [The Problem With Software-Only AI Security](#the-problem-with-software-only-ai-security)
- [The Hardware Landscape](#the-hardware-landscape)
- [Three Core Primitives](#three-core-primitives)
  - [Primitive 1 — Proof-of-Work Request Gating](#primitive-1--proof-of-work-request-gating)
  - [Primitive 2 — Hardware Vector Search Acceleration](#primitive-2--hardware-vector-search-acceleration)
  - [Primitive 3 — Model Integrity Verification](#primitive-3--model-integrity-verification)
- [Reference Implementation — Goldshell MiniDOGE 1](#reference-implementation--goldshell-minidoge-1)
- [Full Stack Integration](#full-stack-integration)
- [Controller Board Research](#controller-board-research)
- [Security Analysis](#security-analysis)
- [Expanding Beyond Scrypt — The Broader Vision](#expanding-beyond-scrypt--the-broader-vision)
- [What a Hashing-Native AI Could Look Like](#what-a-hashing-native-ai-could-look-like)
- [Limitations & Honest Assessment](#limitations--honest-assessment)
- [Future Research Directions](#future-research-directions)
- [Bill of Materials](#bill-of-materials)
- [Repository Structure](#repository-structure)
- [Related Work](#related-work)
- [Contributing](#contributing)
- [License](#license)
- [Status](#status)

-----

## Why This Matters

This is not about Dogecoin. It is about what happens when the cryptographic properties of a hashing algorithm — designed for one purpose — are examined for utility in a completely different domain.

**Three things are true simultaneously:**

```
1. Retired mining ASICs are abundant, cheap, and powerful
         ↓
2. AI agent stacks have no hardware security layer
         ↓
3. Hash functions are fundamental primitives in AI/ML
   (vector search, nearest neighbors, model fingerprinting,
    randomness, proof systems, federated learning)
```

The intersection of these three facts has not been formally explored. This project starts that exploration.

**The broader implication:** If Scrypt hardware has properties useful for AI security, then SHA-256 hardware, Ethash hardware, Blake3 hardware, and every other algorithm-specific ASIC potentially has its own set of AI-relevant properties waiting to be mapped. There is an entire taxonomy of repurposed cryptographic hardware to explore. This repository is the beginning of that map.

-----

## The Problem With Software-Only AI Security

### Request Authentication

```
Current state:
  Agent sends request → API checks JWT token → executes if valid

The problem:
  JWT lives in software    → can be stolen, replayed, or forged
  Rate limiting is memory  → can be reset or bypassed in a compromised process
  Compromised OS           → can authenticate itself against its own API
```

### Model Integrity

```
Current state:
  Load model → check SHA256 hash file → proceed if match

The problem:
  Hash file lives on the same filesystem as the model
  If the system is compromised, both are compromised simultaneously
  Software verification can be intercepted before the comparison runs
```

### The Fundamental Gap

```
What software cannot provide:
  A verification step that exists OUTSIDE the software stack entirely
  Something a compromised OS kernel cannot intercept or spoof
  Asymmetric cost — trivially cheap to verify, impossibly expensive to fake

Scrypt was designed to solve asymmetric cost problems.
A retired Scrypt ASIC is the physical embodiment of that design principle.
Applied not to mining — but to AI agent security.
```

-----

## The Hardware Landscape

This framework is not limited to one device or one algorithm. The following table maps known mining hardware to its cryptographic algorithm and potential AI security applications. **This is a living document — contributions adding new hardware are explicitly invited.**

|Hardware            |Algorithm        |Hash Rate|Power|AI Security Application                            |Research Status     |
|--------------------|-----------------|---------|-----|---------------------------------------------------|--------------------|
|Goldshell MiniDOGE 1|Scrypt           |185 MH/s |233W |PoW gating, LSH acceleration, model integrity      |🔬 Active — this repo|
|Bitmain Antminer L7 |Scrypt           |9.5 GH/s |3425W|Same as above, massively higher throughput         |🔲 Unexplored        |
|Bitmain Antminer S19|SHA-256          |95 TH/s  |3250W|Merkle tree model provenance, blockchain AI logging|🔲 Unexplored        |
|iPollo V1 Mini      |Ethash           |130 MH/s |240W |DAG memory-hard verification                       |🔲 Unexplored        |
|Goldshell KD-Box    |Kadena (Blake2S) |1.6 TH/s |205W |Ultra-fast generic hashing, BLAKE2 primitives      |🔲 Unexplored        |
|Jasminer X4         |Ethash           |520 MH/s |240W |Memory-hard verification, high efficiency          |🔲 Unexplored        |
|Iceriver KS0        |KHeavyHash       |100 GH/s |65W  |**65W only** — extremely power-efficient primitive |🔲 Unexplored        |
|Goldshell AL-BOX    |Alephium (Blake3)|200 GH/s |220W |Blake3 AI applications, tree hashing               |🔲 Unexplored        |
|FutureBit Apollo LTC|Scrypt           |100 MH/s |15W  |**15W** — near-zero power always-on primitive      |🔲 Unexplored        |
|FutureBit Apollo BTC|SHA-256          |varies   |~5W  |USB-powered, tiny, hackable entry point            |🔲 Unexplored        |


> 📝 **Note on the FutureBit Apollo LTC:** At 15W for a Scrypt ASIC it is arguably the most interesting device in the table for homelab AI security. An always-on hardware security primitive drawing less power than a light bulb. This warrants dedicated research.

> 📝 **Note on KHeavyHash (Kaspa):** The KHeavyHash algorithm combines SHA-3 and Matrix operations. Its unique properties may have direct relevance to neural network weight verification and deserve focused research attention.

-----

## Three Core Primitives

### Primitive 1 — Proof-of-Work Request Gating

#### The Concept

Every request reaching the AI agent approval API must carry a valid proof-of-work token computed using the ASIC’s native algorithm. The difficulty is calibrated so that:

- **Legitimate requests** pre-computed by an authorized agent: instant verification
- **Replay attacks**: nonce invalidation renders replays worthless
- **Flood attacks**: attacker must compute valid PoW faster than the ASIC verifies — asymmetrically impossible in software

#### Token Structure

```json
{
  "agent_id":  "r6120-sentinel-01",
  "action":    "restart_service",
  "target":    "dnsmasq",
  "timestamp": "2026-03-22T10:00:00Z",
  "nonce":     "a3f8c2d1",
  "pow_token": {
    "challenge":  "sha256_of_payload",
    "solution":   "value_producing_valid_hash_output",
    "difficulty": 20,
    "algorithm":  "scrypt-N1048576-r8-p1"
  },
  "agent_sig": "ed25519_signature_of_above"
}
```

#### The Asymmetry That Makes This Work

|Actor                  |Hash Capacity           |Tokens/sec at Diff-20|Outcome                    |
|-----------------------|------------------------|---------------------|---------------------------|
|ScryptGate (MiniDOGE 1)|185 MH/s                |185 verifications/sec|Instant                    |
|Software attacker (CPU)|~50,000 H/s             |~0.05 tokens/sec     |3,700,000:1 disadvantage   |
|GPU attacker           |~10 MH/s est.           |~10 tokens/sec       |Still 18,500:1 disadvantage|
|ASIC attacker          |Needs identical hardware|Equal                |Cost and logistics barrier |

#### Verification Flow

```
1. Receive request with PoW token
2. Forward challenge + solution to ScryptGate API
3. ASIC computes hash(challenge, solution)
4. Check output meets difficulty target (leading zero bits)
5. Check nonce not seen before              → replay prevention
6. Check timestamp within 5-minute window  → freshness
7. Verify ed25519 agent signature           → identity
8. All pass → forward to human approval queue
   Any fail → reject immediately, log attempt, alert admin
```

-----

### Primitive 2 — Hardware Vector Search Acceleration

#### The Concept

Vector databases (Chroma, Weaviate, Pinecone, FAISS) use locality-sensitive hashing (LSH) to find approximate nearest neighbors efficiently. LSH requires hash functions where similar inputs produce similar (or identically bucketed) outputs — enabling sub-linear search without comparing every vector.

Scrypt’s mixing function (based on the Salsa20/8 core) has structural properties that warrant investigation as an LSH primitive for binary vector spaces. **This is an open and unexplored research question.**

#### Proposed Architecture

```
Query vector arrives at Chroma
        ↓
Instead of software LSH:
        ↓
Vector → ScryptGate API → ASIC at 185 MH/s
        ↓
Hardware LSH bucket assignment returned
        ↓
Narrow candidate set → standard cosine similarity on candidates only
        ↓
Result returned to LangChain in microseconds
```

#### Open Research Questions

- What is the collision behavior of Scrypt-derived LSH versus standard MinHash or SimHash?
- Does the memory-hardness property of Scrypt provide adversarial robustness against vector search poisoning attacks?
- What is the throughput gain versus software LSH at 1M, 10M, 100M vectors?
- Does this approach extend naturally to other memory-hard algorithms (Ethash, Argon2)?

> ⚠️ **This section is intentionally left as open research.** Experimental results and mathematical analysis are explicitly invited. This is the primitive with the most unexplored potential.

-----

### Primitive 3 — Model Integrity Verification

#### The Concept

Before any model loads into an inference environment, a hardware hash of the model weights is computed by the ASIC and compared against a known-good reference stored in a physically separate, air-gapped vault system. In the reference implementation this is a Qubes OS vault-qube with no network interface — ever.

The security property: **the reference hash lives outside the system being verified.** Even complete compromise of the inference system cannot allow a tampered model to load, because the verification oracle is physically separate and has no network exposure.

#### Verification Flow

```
Request to load model into inference environment
        ↓
ScryptGate reads model file (streamed — not loaded into RAM)
        ↓
ASIC computes hardware hash of model weights
        ↓
Hash sent to air-gapped vault via secure inter-VM message (qrexec)
        ↓
Vault compares against stored reference
(vault has no network interface — cannot be remotely attacked)
        ↓
MATCH    → load approved, vault returns signed approval token
NO MATCH → load rejected, alert generated, human notified immediately
        ↓
Inference proceeds ONLY with valid vault signature in memory
```

#### Why ASIC Hashing vs Standard SHA256

|Property         |SHA256 Checksum          |ASIC Hash (ScryptGate)                 |
|-----------------|-------------------------|---------------------------------------|
|Compute cost     |Instant — no asymmetry   |Memory-hard — asymmetric cost          |
|Forgery cost     |Pre-computable trivially |Requires identical dedicated hardware  |
|Verification root|Same filesystem as model |Air-gapped vault — physically separate |
|Software bypass  |Intercept comparison code|Hardware oracle — cannot be intercepted|
|Attacker cost    |Near zero                |Needs the same ASIC hardware           |

-----

## Reference Implementation — Goldshell MiniDOGE 1

The reference implementation uses the Goldshell MiniDOGE 1 as the initial research platform. Chosen not because it is the best hardware for this purpose, but because it is widely available secondhand for $50–80, well-documented, and has a known entry point via microSD firmware recovery.

|Spec        |Value                         |
|------------|------------------------------|
|Algorithm   |Scrypt (N=1048576, r=8, p=1)  |
|Hash rate   |185 MH/s                      |
|Power draw  |233W                          |
|Connectivity|Ethernet                      |
|Controller  |Embedded Linux                |
|Firmware    |Open source on GitHub         |
|Entry point |microSD card firmware recovery|

**Important distinction:** The ASIC chips themselves are not reprogrammable. The opportunity is the **controller board** — an embedded Linux computer that manages the ASICs, has Ethernet, and can be accessed via microSD firmware replacement. The research goal is to redirect the ASIC’s hashing output from a mining pool to a local API endpoint.

-----

## Full Stack Integration

### Where ScryptGate Sits

```
INTERNET
    ↓
[Edge Router]              ← ASUS GT-AX6000 Merlin in reference impl
    ↓
[Middle Buffer]            ← Netgear R6120 OpenWrt in reference impl
    ↓
[ScryptGate Layer]         ← THIS PROJECT
    ├── Goldshell MiniDOGE 1 (ASIC)
    ├── Controller board (embedded Linux)
    └── Local API (exposes ASIC compute to AI server)
    ↓
[AI Server]                ← MSI Laptop, Fedora Server, Podman stack
    ├── LangChain orchestration
    ├── FastAPI approval API    (ScryptGate-gated)
    ├── Chroma vector DB        (ScryptGate-accelerated)
    └── Ollama inference        (ScryptGate-verified model load)
    ↓
[Labs Network]
    └── Qubes OS workstation
         └── vault-qube (air-gapped, holds reference hashes)
```

### ScryptGate API Surface

```python
# Minimal API — runs on MiniDOGE controller Linux
# Three endpoints only — surface area is intentionally tiny

POST /verify/pow
    Input:  { challenge: str, solution: str, difficulty: int }
    Output: { valid: bool, computation_time_ms: int }

POST /hash/model
    Input:  { model_path: str } or streaming bytes
    Output: { scrypt_hash: str, timestamp: int }

GET  /health
    Output: { status: str, hashrate_mhs: float, uptime: int }
```

### MSI Integration Client

```python
# scryptgate-client.py
# Every inbound agent proposal must clear ScryptGate before human review queue

import httpx

SCRYPTGATE_URL = "http://MINIDOGE_IP:8080"

async def verify_proposal(proposal: dict) -> bool:
    pow_token = proposal.get("pow_token")
    if not pow_token:
        return False
    try:
        response = await httpx.post(
            f"{SCRYPTGATE_URL}/verify/pow",
            json={
                "challenge":  pow_token["challenge"],
                "solution":   pow_token["solution"],
                "difficulty": pow_token["difficulty"],
            },
            timeout=5.0
        )
        return response.json().get("valid", False)
    except Exception:
        return False  # Fail closed — always


async def verify_model_before_load(model_path: str, vault_reference: str) -> bool:
    response = await httpx.post(
        f"{SCRYPTGATE_URL}/hash/model",
        json={"model_path": model_path},
        timeout=300.0
    )
    hardware_hash = response.json().get("scrypt_hash")
    return hardware_hash == vault_reference
```

-----

## Controller Board Research

### Known Entry Points — MiniDOGE 1

|Entry Point                |Status       |Notes                                   |
|---------------------------|-------------|----------------------------------------|
|microSD slot               |✅ Documented |Firmware recovery — main research path  |
|Web interface              |✅ Documented |Runs on controller Linux, LAN accessible|
|Ethernet port              |✅ Documented |Controller has its own IP               |
|Goldshell firmware source  |✅ Documented |Open source on GitHub                   |
|UART header location       |🔬 In Progress|PCB inspection required                 |
|Controller chip ID         |🔬 In Progress|ARM or MIPS — TBD                       |
|ASIC communication protocol|🔬 In Progress|Likely custom SPI or UART               |
|Raw ASIC output redirection|🔲 Not Started|Key research milestone                  |

### Research Pathway

```
Phase 1 — Non-destructive access
  └── microSD firmware dump and analysis
  └── Identify controller chip via firmware strings

Phase 2 — Controller understanding
  └── UART header location via PCB inspection
  └── Map ASIC ↔ controller bus communication
  └── Identify nonce assignment and result protocol

Phase 3 — Redirection
  └── Intercept ASIC result stream at controller
  └── Redirect from pool protocol to local HTTP API
  └── Expose raw Scrypt compute as a network service

Phase 4 — ScryptGate API
  └── Build three-endpoint API on Phase 3 foundation
  └── Integration testing with AI approval stack
  └── Benchmark and document throughput
```

-----

## Security Analysis

### Threat Model

|Threat                           |ScryptGate Response                                  |
|---------------------------------|-----------------------------------------------------|
|Agent request flooding           |PoW requirement — attacker cannot compute fast enough|
|Replay attacks                   |Nonce ledger + 5-minute timestamp window             |
|Stolen agent signing key         |Ed25519 signature still required alongside PoW       |
|Tampered model weights           |Hardware hash mismatch → load rejected               |
|Compromised verification software|Reference lives in air-gapped vault                  |
|Physical ASIC theft              |Reference hashes orphaned → system halts cleanly     |
|ScryptGate process compromised   |Fails closed → no tokens → all agent actions blocked |
|Vault exfiltration               |No network interface — physical access required      |

### Fail-Safe Behavior

ScryptGate fails **closed, not open.** This is non-negotiable.

```
If ScryptGate is unreachable for any reason:
    → No PoW tokens can be verified
    → All automated agent requests are rejected
    → Human operator receives alert
    → Zero automated action possible until hardware restored
    → System defaults to requiring full manual human operation

A security primitive that fails open is not a security primitive.
```

-----

## Expanding Beyond Scrypt — The Broader Vision

Scrypt is one algorithm. One hardware family. The same conceptual framework applies across every major mining algorithm — and each algorithm has different mathematical properties that may map to different AI and ML primitives.

### Algorithm Property Map

|Algorithm             |Core Property               |Potential AI Application                               |
|----------------------|----------------------------|-------------------------------------------------------|
|**Scrypt**            |Memory-hard sequential      |PoW gating, LSH, model integrity — this repo           |
|**SHA-256**           |Deterministic tree hashing  |Merkle tree model provenance, audit chains             |
|**Ethash / ProgPoW**  |DAG memory-hard             |Large-scale memory-hard verification                   |
|**Blake2S / Blake3**  |Ultra-fast generic hash     |High-throughput model fingerprinting                   |
|**KHeavyHash (Kaspa)**|SHA-3 + Matrix operations   |**Matrix ops in silicon — direct neural net relevance**|
|**RandomX (Monero)**  |CPU-optimized diverse ops   |General compute emulation, instruction diversity       |
|**Autolykos (Ergo)**  |Memory-hard + elliptic curve|Key derivation, cryptographic AI attestation           |
|**Fishhash**          |Memory-hard Ethash variant  |Memory-hard verification, newer hardware               |

### The KHeavyHash Observation

KHeavyHash deserves specific attention. The algorithm used by Kaspa mining hardware combines SHA-3 (Keccak) hashing with **matrix multiplication operations.**

Matrix multiplication is the fundamental operation in neural network inference. Hardware specifically designed to run SHA-3 combined with matrix operations at high efficiency shares silicon primitives with hardware designed to run small neural network layers. The Iceriver KS0 does this at 100 GH/s drawing only 65W.

**The research question:** Can KHeavyHash ASIC hardware be leveraged as a neural network layer accelerator for specific architectures? Even for a narrow class of operations this would be a significant finding.

-----

## What a Hashing-Native AI Could Look Like

This is speculative. It is worth articulating clearly.

Current AI architectures are built on float32/float16/int8 matrix multiplication, attention mechanisms, and learned activation functions running on GPU hardware that costs hundreds to thousands of dollars and draws hundreds of watts.

What if you designed a neural architecture from scratch around operations that run natively and efficiently on hashing silicon?

```
Hypothetical Hash-Native Neural Architecture:

Input
    ↓
Hash-based feature encoding
(fixed projection via hardware hash — no learned embedding needed)
    ↓
LSH-based sparse attention
(hardware LSH finds relevant tokens — no full attention matrix)
    ↓
Hash-accumulator layers
(hash composition as learned transform — runs on ASIC natively)
    ↓
Proof-of-work verified output
(output carries PoW token — verifiable computation provenance)
    ↓
Output with hardware attestation
(cryptographic proof of which hardware produced this output)
```

This is not a claim that such an architecture would outperform transformers on general benchmarks. It is a claim that an architecture designed around hashing primitives would:

- Run on hardware costing $15–500 used
- Draw a fraction of the power of GPU inference
- Produce outputs with **cryptographic provenance** — verifiable proof that specific hardware produced a specific output
- Be physically impossible to fake without the specific hardware in hand

**Whether such a system produces useful intelligence is the open research question.** But AI with hardware-rooted output provenance — where you can cryptographically verify not just what was output but on what hardware it was computed — is a novel property that transformer-based systems running on commodity GPUs fundamentally cannot provide.

That might matter. A lot. In contexts where you need to trust not just the output but the process that produced it.

-----

## Limitations & Honest Assessment

**What this is:**

- A novel hardware security primitive for AI agent authentication
- A research framework for mapping cryptographic hashing hardware to AI/ML applications
- A working reference implementation on consumer hardware under $101
- A first formal articulation of an unexplored problem space

**What this is not:**

- A replacement for proper HSMs or TPMs in enterprise environments
- A proven production system — research is active and ongoing
- An LLM accelerator — matrix multiplication is not Scrypt
- Fully validated — Primitive 2 is theoretical pending benchmarks

**Known limitations:**

- MiniDOGE 1 controller research ongoing — full compute redirection not yet achieved
- Scrypt as LSH is mathematically unproven at scale
- 233W for a security coprocessor is high — FutureBit Apollo at 15W is the better long-term target
- Single point of failure without hardware redundancy

-----

## Future Research Directions

|Direction                                |Priority   |Status     |
|-----------------------------------------|-----------|-----------|
|Scrypt LSH — formal mathematical proof   |High       |Open       |
|Benchmarks at 1M / 10M / 100M vectors    |High       |Open       |
|KHeavyHash matrix operation analysis     |High       |Not Started|
|MiniDOGE 1 controller firmware RE        |Critical   |In Progress|
|ASIC communication protocol docs         |Critical   |In Progress|
|FutureBit Apollo LTC research (15W)      |High       |Not Started|
|SHA-256 ASIC for Merkle provenance       |Medium     |Not Started|
|Blake3 ASIC applications                 |Medium     |Not Started|
|Hash-native neural architecture prototype|Speculative|Long Term  |
|Formal security proof for PoW-gated auth |High       |Not Started|
|Multi-ASIC array throughput scaling      |Medium     |Not Started|
|Cross-platform controller abstraction    |Medium     |Not Started|
|Federated learning PoW verification      |Low        |Not Started|

-----

## Bill of Materials

> Zero ongoing subscriptions. Runs entirely on local hardware.

|Component           |Est. Cost   |Notes                                |
|--------------------|------------|-------------------------------------|
|Goldshell MiniDOGE 1|$50–80      |Used/retired — eBay or crypto markets|
|Ethernet cable      |$5          |Cat5e or better                      |
|MicroSD card (8GB+) |$8          |Firmware access                      |
|USB-UART adapter    |$8          |Controller research                  |
|**Total**           |**~$71–101**|**$0 recurring**                     |

**Alternative starting points by budget:**

|Budget  |Device              |Algorithm |Why                                |
|--------|--------------------|----------|-----------------------------------|
|$15–30  |FutureBit Apollo BTC|SHA-256   |Tiny, USB-powered, hackable        |
|$50–80  |Goldshell MiniDOGE 1|Scrypt    |This repo — documented             |
|$60–100 |Iceriver KS0        |KHeavyHash|65W, matrix ops, interesting       |
|$80–150 |iPollo V1 Mini      |Ethash    |Memory-hard, efficient             |
|$100–200|FutureBit Apollo LTC|Scrypt    |15W only — best efficiency in class|

-----

## Repository Structure

```
scryptgate/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
│
├── docs/
│   ├── hardware-research.md         ← controller board findings per device
│   ├── asic-protocol.md             ← ASIC communication protocol research
│   ├── algorithm-map.md             ← algorithm → AI primitive mapping
│   ├── security-analysis.md         ← formal threat model
│   ├── hash-native-ai.md            ← speculative architecture notes
│   └── benchmarks.md                ← experimental results as they come in
│
├── firmware/
│   └── controller/
│       ├── minidoge-1/              ← MiniDOGE specific work
│       └── README.md
│
├── api/
│   ├── scryptgate-server.py         ← runs on controller board
│   ├── scryptgate-client.py         ← AI server integration library
│   └── tests/
│
├── integration/
│   ├── approval-api-patch.py        ← patches FastAPI approval API
│   ├── chroma-lsh-bridge.py         ← vector search (experimental)
│   ├── model-verify.sh              ← model integrity script
│   └── langchain-plugin/            ← LangChain tool wrapper
│
├── research/
│   ├── lsh-experiments/             ← Scrypt as LSH benchmarks
│   ├── pow-analysis/                ← PoW difficulty calibration
│   ├── kheavyhash-matrix/           ← KHeavyHash matrix op analysis
│   └── hash-native-architecture/    ← speculative AI architecture sketches
│
└── hardware/
    ├── minidoge-1/
    │   ├── pcb-photos/
    │   ├── uart-pinout.md
    │   └── firmware-notes.md
    ├── hardware-table.md            ← full hardware landscape tracking
    └── iommu-notes.md
```

-----

## Related Work

- Percival, C. (2009). *Stronger Key Derivation via Sequential Memory-Hard Functions.* — Original Scrypt paper
- Indyk, P. & Motwani, R. (1998). *Approximate Nearest Neighbors: Towards Removing the Curse of Dimensionality.* — LSH foundations
- Bonneau, J. et al. (2015). *Proofs of Work in Practice.* — PoW security analysis
- Dwork, C. & Naor, M. (1993). *Pricing via Processing, or Combatting Junk Mail.* — Original PoW concept
- Goldshell Firmware Repository — `github.com/goldshell`
- FutureBit Apollo documentation — USB-native mining ASIC reference
- OpenWrt Project — `openwrt.org`
- Qubes OS Project — `qubes-os.org`

> **If you know of work at the intersection of mining hardware repurposing and AI/ML that belongs here — please open an issue. The goal is a complete literature map of this space.**

-----

## Contributing

This is an open research project at an early stage. All contributions are welcome:

- **Hardware research** — any device in the hardware table, not just MiniDOGE 1
- **Mathematics** — formal analysis of Scrypt/Blake3/KHeavyHash as ML primitives
- **Security analysis** — threat modeling, formal proofs, attack research
- **Implementation** — controller firmware, API code, integration libraries
- **Algorithm mapping** — new entries in the hardware and algorithm tables
- **Architecture** — hash-native neural network exploration
- **Documentation** — reproducibility, clarity, translation

Please open an issue before submitting large PRs so direction can be discussed.

If you have physical access to any hardware in the table and want to contribute findings — **that is the most valuable contribution you can make right now.**

-----

## License

MIT License — use it, build on it, cite it.

```
ScryptGate: Repurposed Mining ASIC as AI Agent Security Primitive
First published: March 22, 2026
https://github.com/ChimeraMind/scryptgate
```

-----

## Status

|Component                               |Status                           |
|----------------------------------------|---------------------------------|
|Concept and architecture                |✅ Complete                       |
|Security analysis and threat model      |✅ Complete                       |
|Hardware landscape table                |✅ Initial — contributions invited|
|Algorithm → AI primitive map            |✅ Initial — contributions invited|
|Primitive 1 — PoW gating design         |✅ Complete                       |
|Primitive 1 — PoW implementation        |🔬 In Progress                    |
|Primitive 2 — Vector search acceleration|🔬 Open Research                  |
|Primitive 3 — Model integrity design    |✅ Complete                       |
|Primitive 3 — Implementation            |🔬 In Progress                    |
|Controller board research (MiniDOGE 1)  |🔬 In Progress                    |
|KHeavyHash matrix analysis              |🔲 Not Started                    |
|Hash-native AI architecture             |🔲 Speculative / Long Term        |
|Benchmarks                              |🔲 Pending hardware               |
|Formal security proof                   |🔲 Future work                    |

-----

*Built on a home network.*  
*Costs less than a dinner out.*  
*The best research happens when someone refuses to throw hardware away.*

-----

**First published: March 22, 2026**  
**Prior art established. The ground is claimed. Build on it.**
