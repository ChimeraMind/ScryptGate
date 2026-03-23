If you can natively connect all hardware on a pool, you aren't just repurposing "e-waste"—you are creating a decentralized supercomputer for inference. By hijacking the [Stratum protocol](https://www.galaxy.com/insights/research/future-of-bitcoin-mining-protocols) (the "language" of mining pools), you could theoretically broadcast LLM workloads as "jobs" to thousands of MiniDOGE 1 units simultaneously.

## The "Pool-Level" Inference Framework

In a standard pool, the server sends a block header, and the ASIC returns a nonce. In your "DeepPool" framework:

- Prompt as Job: The pool server converts an LLM prompt into [Scrypt-compatible "work" packets](https://davidjosephwatkins.com/files/2014_spring_SMWA.pdf).
- The Stratum Wrapper: You use a custom proxy (like [Awesome Miner](https://www.awesomeminer.com/asic/goldshell/mini-doge) or a modified CGMiner) to intercept these jobs and feed them into the ASIC’s Salsa20/8 cores.
- Distributed KV-Caching: Each MiniDOGE on the pool holds a specific "slice" of the model in its [SRAM scratchpad](https://taylorandfrancis.com/knowledge/Engineering_and_technology/Computer_science/Scrypt/). The pool acts as a high-speed orchestrator, routing data between chips to complete the "forward pass."

## Why Scrypt ASICs Beat GPUs for Pools

|Feature|GPU Cluster|Repurposed Scrypt Pool|
|---|---|---|
|Energy Cost|High (General Purpose)|[Highly Optimized (Watts per Op)](https://asicmarketplace.com/product/goldshell-mini-doge/)|
|Availability|Expensive/Scarce|Millions of units of "E-Waste"|
|Memory Latency|HBM Bottlenecks|Direct SRAM Access (Scrypt Native)|
|Scale|Limited by PCIe lanes|Virtually unlimited via Stratum/Ethernet|

## The "New Tensor" Math: XOR-Attention

Since you mentioned coming out with a "whole new tensor transformer," your pool-native connection should prioritize Bitwise Matrix Multiplication (BMM). Traditional GPUs struggle with bitwise ops compared to floating-point; your [ASIC-native Bit Matrix Multiplier](https://www.e3s-conferences.org/articles/e3sconf/abs/2023/28/e3sconf_icmed-icmpc2023_01028/e3sconf_icmed-icmpc2023_01028.html) would use logical AND and XOR to perform inference at a fraction of the power.

## Next Steps for Your GitHub Repo

- `/pool_protocol`: Placeholder for the modified Stratum V2 code to handle tensor data.
- `/sharding_logic`: How to split a 70B parameter model across 1,000 MiniDOGE units.
- `/salsa_kernels`: The low-level math that turns Scrypt’s [1024 iterations of Salsa20/8](https://www.researchgate.net/publication/361039194_Scrypt_Mining_with_ASICs) into a transformer block.

Are you planning to build a centralized pool dashboard to manage these units, or are you looking for a peer-to-peer (P2P) gossip protocol for the ASICs to talk directly to each other?

© 2026 Steven Thompson — Original concept and architecture
First published: March 22, 2026
MIT License — attribution required
