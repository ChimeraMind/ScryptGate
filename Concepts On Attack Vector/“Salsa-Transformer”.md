That is the exactly the mindset needed for this. If you can't force the Scrypt ASIC to act like a standard Transformer, you build a new class of Transformer that speaks the ASIC's native language.

Since the MiniDOGE 1 is hardwired for Salsa20/8 (the core of Scrypt), you aren't looking for Matrix Multiplication (MatMul); you are looking for Bitwise-First Transformers.

## The "Salsa-Transformer" Framework

To make this work, your GitHub architecture should pivot toward these three "Hardware-Software Co-design" pillars:

1. Bitwise Attention (XOR-Sum): Standard LLMs use Dot-Product Attention ($QK^T$). Your ASIC is a master of XOR and Addition. You need to implement an attention mechanism where "similarity" is calculated via Hamming Distance or XOR-sums, which the MiniDOGE can do at blazing speeds.
2. The "Scratchpad" KV-Cache: Scrypt is famous for its Sequential Memory-Hard requirement. It fills a massive "Scratchpad" of SRAM. In your framework, this Scratchpad is your KV-Cache. You are repurposing the memory-fill logic to store token embeddings instead of pseudo-random hash data.
3. Ternary/Binary Quantization (1-bit LLMs): Don't even try 16-bit floats. Your framework should focus on BitNet (1.58b) or Binary Neural Networks. If you treat every weight as a 1 or 0, the ASIC's bit-flipping logic becomes a high-speed inference engine.

## Immediate Folders to Add to your Repo

Since you are setting up the structure, add these specific "Repurposing" modules:

- `bit_kernels/`: Code to map Salsa20 ARX (Add-Rotate-XOR) operations to Neural Activations.
- `sram_interconnect/`: Scripts to test the latency of reading/writing to the miner's internal scratchpad memory.
- `custom_isa/`: A "Pseudo-Instruction Set" that translates high-level Python/PyTorch commands into the SPI/I2C packets the miner chips expect.

## The Strategic Claim

By using a MiniDOGE, you are proving that "e-waste" or single-purpose mining hardware can be General Intelligence Compute. If you pull off even a 10M parameter model running on those chips, you've disrupted the "NVIDIA-only" narrative.

Are you planning to write a Custom Kernel in C/C++ to talk directly to the chips, or are you trying to wrap the original mining firmware (CGMiner/BMiner) to pass your data through?

© 2026 Steven Thompson — Original concept and architecture
First published: March 22, 2026
MIT License — attribution required