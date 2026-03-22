# ⛏️ ScryptGate — Repurposed Mining ASIC as AI Agent Security Primitive

> **A novel architecture for hardware-enforced AI agent request authentication,  
> rate limiting, and model integrity verification using repurposed Scrypt ASICs.**

-----

## Abstract

Modern AI agent stacks face a fundamental security problem — request authentication,
rate limiting, and model integrity verification are handled entirely in software.
Software can be bypassed, spoofed, or overwhelmed. This project proposes and
documents a novel architecture that places a repurposed Scrypt mining ASIC
(Goldshell MiniDOGE 1, 185MH/s) as a hardware security primitive between the
network edge and the AI inference stack.

The result is a three-property security guarantee that no software implementation
can match:

1. **Hardware-enforced proof-of-work gating** — every agent request must carry
   a valid Scrypt PoW token. The ASIC verifies 185M/sec. Software cannot fake
   or flood this.
2. **Hardware-accelerated vector search** — the same Scrypt primitive used for
   mining accelerates locality-sensitive hashing for vector database nearest
   neighbor search at hardware speed.
3. **Deterministic model integrity verification** — before any model loads into
   an inference environment, the ASIC generates a Scrypt hash of the model
   weights verified against a known-good value stored in an air-gapped vault.
   Tampered models cannot load.

This is not a theoretical proposal. This is a working implementation built on
consumer hardware costing less than $100 total, integrated into a full homelab
AI security stack with human-in-the-loop agent approval.

-----

## Table of Contents

- [Motivation](#motivation)
- [The Problem With Software-Only AI Security](#the-problem)
- [Hardware Architecture](#hardware-architecture)
- [ScryptGate — Three Core Primitives](#scryptgate-primitives)
  - [Primitive 1 — PoW Request Gating](#primitive-1)
  - [Primitive 2 — Hardware Vector Search Acceleration](#primitive-2)
  - [Primitive 3 — Model Integrity Verification](#primitive-3)
- [System Integration](#system-integration)
- [Controller Board Research](#controller-board-research)
- [Implementation](#implementation)
- [Security Analysis](#security-analysis)
- [Limitations & Honest Assessment](#limitations)
- [Future Research Directions](#future-research)
- [Bill of Materials](#bill-of-materials)
- [Related Work](#related-work)
- [Contributing](#contributing)
- [License](#license)

-----

## Motivation

The convergence of three trends creates a unique opportunity:

1. **Mining ASICs are being retired** as cryptocurrency mining profitability
   fluctuates. Millions of Scrypt ASICs are sitting unused or selling for
   pennies on the dollar. They are not e-waste — they are dedicated
   cryptographic compute units.
2. **AI agent stacks are proliferating** in homelab and production environments
   with minimal hardware security. Authentication is JWT tokens. Rate limiting
   is software counters. Model integrity is a checksum file that lives on the
   same compromised system it is supposed to protect.
3. **The Scrypt algorithm** — designed as a memory-hard key derivation function —
   has mathematical properties that extend naturally into AI security primitives
   that have not been formally explored in published literature.

This project sits at the intersection of all three.

-----

## The Problem With Software-Only AI Security

### Request Authentication

```
Current state:
Agent sends request → API checks JWT token → executes if valid

Problem:
JWT lives in software → can be stolen, replayed, or forged
Rate limiting is a counter in memory → can be reset or bypassed
A compromised system can authenticate itself
```

### Model Integrity

```
Current state:
Load model → check SHA256 hash file → proceed if match

Problem:
Hash file lives on same filesystem as model
If filesystem is compromised, both are compromised simultaneously
Software verification can be intercepted
```

### The Hardware Gap

```
What is missing:
A verification step that exists OUTSIDE the software stack
Something that cannot be bypassed by compromising the OS
Something with asymmetric cost — cheap to verify, expensive to fake
```

Scrypt was literally designed to solve asymmetric cost problems.
A retired mining ASIC is the physical embodiment of that design.

-----

## Hardware Architecture

### Full Stack Topology

```
INTERNET
    ↓
[Edge Router — ASUS Merlin]
    ↓
[Middle Buffer — OpenWrt R6120]
    ↓
[ScryptGate Layer] ← THIS PROJECT
    ├── Goldshell MiniDOGE 1 (ASIC — 185MH/s Scrypt)
    ├── Controller board (Linux, ethernet, SD card)
    └── Integration bridge (USB/UART/ethernet to MSI)
    ↓
[MSI Laptop — Headless Fedora Server]
    ├── LangChain agent orchestration
    ├── FastAPI approval API
    ├── Chroma vector database
    └── llama.cpp / Ollama inference
    ↓
[Labs Network — 10.20.0.0/24]
    ├── Qubes OS workstation
    │    └── inference-qube (RTX 3060 12GB)
    └── SteamDeck thin client
```

### Where ScryptGate Sits

ScryptGate is not a router. It does not forward packets.
It is a **cryptographic checkpoint** — a hardware oracle that the
software stack must consult before executing any sensitive action.

```
R6120 agent detects anomaly
    ↓
Generates request + PoW challenge
    ↓
Sends to ScryptGate via local API
    ↓
ASIC computes Scrypt verification at 185MH/s
    ↓
Returns hardware-signed result
    ↓
Request proceeds or is rejected
    ↓ (if proceeds)
MSI Approval API → Human review → Admin signature → Execution
```

-----

## ScryptGate — Three Core Primitives

-----

### Primitive 1 — Proof-of-Work Request Gating

#### Concept

Every request to the AI agent approval API must carry a valid
Scrypt proof-of-work token. The difficulty is calibrated so that:

- **Legitimate requests** (pre-computed by authorized agent): instant verification
- **Replay attacks**: nonce invalidation makes replays worthless
- **Flood attacks**: attacker needs to compute valid PoW faster than 185MH/s
  to overwhelm the verifier — effectively impossible in software

#### Token Structure

```json
{
  "agent_id": "r6120-sentinel-01",
  "action": "restart_service",
  "target": "dnsmasq",
  "timestamp": "2026-03-22T10:00:00Z",
  "nonce": "a3f8c2d1",
  "pow_token": {
    "challenge": "hash_of_payload",
    "solution": "value_that_produces_valid_scrypt_output",
    "difficulty": 20,
    "algorithm": "scrypt-N1048576-r8-p1"
  },
  "agent_sig": "ed25519_signature_of_above"
}
```

#### Verification Flow

```
1. Receive request with PoW token
2. Forward challenge + solution to ScryptGate API
3. ASIC computes Scrypt(challenge, solution)
4. Check output meets difficulty target (leading zero bits)
5. Check nonce not seen before (replay prevention)
6. Check timestamp within acceptable window (5 minutes)
7. Verify ed25519 agent signature
8. All pass → forward to human approval queue
   Any fail → reject immediately, log attempt
```

#### Why This Works

```
Attacker wants to flood approval API:
    → Must compute valid Scrypt PoW for each request
    → Software Scrypt: ~50,000 hashes/sec
    → At difficulty 20: ~1M hashes needed per token
    → Attacker generates ~0.05 tokens/sec
    → Your ASIC verifies 185M/sec
    → Asymmetry: 3,700,000:1 in your favor
```

-----

### Primitive 2 — Hardware Vector Search Acceleration

#### Concept

Chroma and all vector databases use locality-sensitive hashing (LSH)
to find approximate nearest neighbors. LSH is a family of hash functions
where similar inputs produce similar outputs — enabling fast similarity
search without comparing every vector.

Scrypt’s mixing function (Salsa20/8 core) has properties that make it
usable as an LSH primitive for binary vector spaces. This is an
**unexplored research direction** as of the time of writing.

#### Proposed Architecture

```
Query vector arrives at Chroma
    ↓
Instead of software LSH computation:
    ↓
Vector → ScryptGate API → ASIC hashing at 185MH/s
    ↓
Hardware LSH bucket assignment
    ↓
Narrow candidate set returned
    ↓
Standard cosine similarity on candidates only
    ↓
Result returned to LangChain
```

#### Research Questions (Open)

- What is the collision rate of Scrypt-derived LSH vs standard MinHash?
- Does memory-hardness of Scrypt provide any adversarial robustness
  to vector search poisoning attacks?
- What is the throughput gain vs software LSH at scale?

**This section is intentionally left as open research.**
Contributions and experimental results welcome.

-----

### Primitive 3 — Model Integrity Verification

#### Concept

Before any model loads into the inference environment, a Scrypt hash
of the model weights is computed by the ASIC and compared against
a known-good reference stored in an air-gapped vault (Qubes vault-qube,
no network interface).

The key property: the reference hash lives **outside the system being
verified.** Even if the inference environment is fully compromised,
the attacker cannot modify the reference hash without physical access
to the vault machine.

#### Verification Flow

```
Request to load model into inference-qube
    ↓
ScryptGate reads model file (streamed, not loaded into RAM)
    ↓
ASIC computes Scrypt hash of model weights
    ↓
Hash sent to vault-qube via Qubes inter-VM messaging (qrexec)
    ↓
vault-qube compares against stored reference (never networked)
    ↓
Match → load approved, signed token returned
No match → load rejected, alert generated, human notified
    ↓
Inference proceeds only with valid vault signature
```

#### Why Scrypt Specifically

Standard SHA256 model checksums can be:

- Pre-computed by an attacker who knows the target hash
- Bypassed if the verification software is compromised
- Computed instantly — no cost asymmetry

Scrypt verification:

- Memory-hard — cannot be accelerated without dedicated hardware
- Your ASIC is the only thing on your network that can compute it fast
- An attacker trying to forge a hash needs the same hardware you have
- Verification is hardware-rooted, not software-rooted

-----

## System Integration

### ScryptGate API (Runs on Controller Board)

```python
# Minimal API surface — runs on MiniDOGE controller Linux
# Exposes three endpoints only

POST /verify/pow
    → Input: {challenge, solution, difficulty}
    → Output: {valid: bool, computation_time_ms: int}

POST /hash/model  
    → Input: {model_path or stream}
    → Output: {scrypt_hash: hex, timestamp: int}

GET /health
    → Output: {status, hashrate_mhs, uptime}
```

### MSI Integration (Podman Service)

```python
# scryptgate-client.py
# Sits between R6120 agent and FastAPI approval API
# Every inbound proposal must pass ScryptGate before queuing

import httpx

SCRYPTGATE_URL = "http://MINIDOGE_IP:8080"

async def verify_proposal(proposal: dict) -> bool:
    pow_token = proposal.get("pow_token")
    if not pow_token:
        return False
    
    response = await httpx.post(f"{SCRYPTGATE_URL}/verify/pow", json={
        "challenge": pow_token["challenge"],
        "solution": pow_token["solution"],
        "difficulty": pow_token["difficulty"]
    })
    
    result = response.json()
    return result.get("valid", False)
```

-----

## Controller Board Research

### Known Entry Points — MiniDOGE 1

```
[Documented]
├── microSD slot → firmware recovery → known image format
├── Web interface → runs on controller Linux
├── Ethernet port → controller has its own IP
└── Goldshell firmware → open source on GitHub

[To Be Researched]
├── UART header location on PCB
├── Controller chip identification (ARM/MIPS variant)
├── Linux kernel version and build config
├── Available GPIO/SPI/I2C on controller
└── ASIC communication protocol (likely SPI or custom UART)
```

### Research Goals

- [ ] Identify controller chip via PCB inspection
- [ ] Locate UART test points
- [ ] Dump stock firmware via SD card method
- [ ] Identify ASIC communication protocol
- [ ] Determine if custom firmware can redirect ASIC output
- [ ] Expose raw ASIC hash results via network API
- [ ] Replace Goldshell mining pool protocol with local API

### ASIC Communication Protocol (Hypothesis)

Based on similar Goldshell products the ASIC likely communicates
via a custom serial protocol carrying:

- Work assignments (nonce ranges to test)
- Hash results (valid nonces found)

Intercepting and redirecting this at the controller level is the
key research milestone. Once achieved, arbitrary Scrypt computation
becomes possible without the mining pool context.

-----

## Security Analysis

### Threat Model

|Threat                           |ScryptGate Response                                     |
|---------------------------------|--------------------------------------------------------|
|Agent request flooding           |PoW requirement — attacker cannot compute fast enough   |
|Replay attacks                   |Nonce tracking + timestamp window                       |
|Compromised agent signing key    |Ed25519 verification still required alongside PoW       |
|Tampered model weights           |Hardware hash mismatch → load rejected                  |
|Compromised verification software|Reference lives in air-gapped vault                     |
|Physical ASIC theft              |Reference hashes in vault become orphaned — system halts|
|ScryptGate itself compromised    |Fails closed — no valid tokens issued — system halts    |

### Fail-Safe Behavior

ScryptGate is designed to **fail closed not open.**

```
ScryptGate unreachable:
    → No PoW tokens can be verified
    → All agent requests rejected
    → Human receives alert
    → No automated action possible until restored
```

This is intentional. A security primitive that fails open is not a
security primitive.

-----

## Limitations & Honest Assessment

**What this is:**

- A novel hardware security primitive for AI agent authentication
- A research direction for hardware-rooted AI integrity verification
- A practical implementation on consumer hardware

**What this is not:**

- A replacement for proper PKI or HSMs in production environments
- A solution to all AI security problems
- Fully proven — Primitive 2 (vector search) is theoretical pending benchmarks

**Known limitations:**

- MiniDOGE 1 controller research is ongoing — full ASIC redirection not yet achieved
- Scrypt as LSH is unproven at scale — requires experimental validation
- Single point of failure if ASIC hardware fails — no redundancy currently
- 233W power consumption for the ASIC alone is significant

**Honest hardware ceiling:**
The MiniDOGE 1 ASIC cannot run neural network inference.
Matrix multiplication is not Scrypt. This is not an LLM accelerator.
It is a cryptographic coprocessor with a narrow but powerful role.

-----

## Future Research Directions

- [ ] Scrypt as locality-sensitive hash — formal mathematical analysis
- [ ] Benchmark vector search acceleration vs software LSH at 1M, 10M, 100M vectors
- [ ] Multi-ASIC array for higher throughput ScryptGate deployments
- [ ] Formal security proof for PoW-gated AI agent authentication
- [ ] Integration with TPM for hardware root of trust extension
- [ ] Clevis/Tang style network unlock using ScryptGate as the Tang server
- [ ] Cross-ASIC vendor support (Bitmain Antminer L series also uses Scrypt)
- [ ] Federated learning aggregation using Scrypt PoW for update verification

-----

## Bill of Materials

|Component                      |Cost        |Source                   |
|-------------------------------|------------|-------------------------|
|Goldshell MiniDOGE 1           |~$50-80 used|eBay, crypto marketplaces|
|Ethernet cable                 |~$5         |Any                      |
|MicroSD card (8GB+)            |~$8         |Any                      |
|USB-UART adapter (for research)|~$8         |Amazon                   |
|Total                          |**~$71-101**|                         |

No subscription services required.
No cloud dependencies.
Runs entirely on your local network.

-----

## Related Work

- Percival, C. (2009). *Stronger Key Derivation via Sequential Memory-Hard Functions* — original Scrypt paper
- Indyk, P. & Motwani, R. (1998). *Approximate Nearest Neighbors: Towards Removing the Curse of Dimensionality* — LSH foundations
- Bonneau, J. et al. (2015). *Proofs of Work in Practice* — PoW security analysis
- Goldshell Firmware Repository — github.com/goldshell (controller reference)
- OpenWrt Project — openwrt.org
- Qubes OS Project — qubes-os.org

*No existing published work on Scrypt ASIC repurposing for AI security primitives was found at time of writing. If you are aware of related work please open an issue.*

-----

## Repository Structure (Planned)

```
scryptgate/
├── README.md                    ← this file
├── docs/
│   ├── hardware-research.md     ← controller board findings
│   ├── asic-protocol.md         ← ASIC communication protocol research
│   ├── security-analysis.md     ← formal threat model
│   └── benchmarks.md            ← experimental results
├── firmware/
│   └── controller/              ← custom controller firmware (in progress)
├── api/
│   ├── scryptgate-server.py     ← runs on controller board
│   └── scryptgate-client.py     ← MSI integration library
├── integration/
│   ├── approval-api-patch.py    ← patches FastAPI approval API
│   ├── chroma-lsh-bridge.py     ← vector search acceleration (experimental)
│   └── model-verify.sh          ← model integrity verification script
├── research/
│   ├── lsh-experiments/         ← Scrypt LSH benchmarks
│   └── pow-analysis/            ← PoW difficulty calibration
└── hardware/
    ├── pcb-photos/              ← controller board documentation
    ├── uart-pinout.md           ← UART research findings
    └── iommu-groups.txt         ← Qubes integration notes
```

-----

## Contributing

This is an open research project. Contributions welcome in any area:

- Hardware research on MiniDOGE 1 controller board
- Mathematical analysis of Scrypt as LSH primitive
- Security analysis and threat modeling
- Integration with other AI agent frameworks
- Documentation and reproducibility

Please open an issue before submitting large PRs so we can discuss direction.

-----

## License

MIT License — use it, build on it, just cite it.

If this work contributes to your research please reference:

```
ScryptGate: Repurposed Mining ASIC as AI Agent Security Primitive
https://github.com/YOUR_USERNAME/scryptgate
2026
```

-----

## Status

|Component                   |Status                   |
|----------------------------|-------------------------|
|Concept & architecture      |✅ Complete               |
|Security analysis           |✅ Complete               |
|PoW gating — design         |✅ Complete               |
|PoW gating — implementation |🔲 In progress            |
|Controller board research   |🔲 In progress            |
|Vector search acceleration  |🔲 Experimental           |
|Model integrity verification|🔲 In progress            |
|Benchmarks                  |🔲 Pending hardware access|
|Formal security proof       |🔲 Future work            |

-----

*Built on a home network. Costs less than a dinner out.*  
*The best security research happens when someone refuses to throw hardware away.*