
## The Repurposing Strategy

To turn a Scrypt miner into an LLM runner, your framework needs to address these three "hacks":

1. Exploiting the Salsa20/8 Core: Scrypt relies on the Salsa20/8 core. In an ASIC, this is a fixed pipeline of additions, XORs, and rotations. Your testing framework must determine if these 32-bit arithmetic primitives can be "tricked" into performing the Multiply-Accumulate (MAC) operations required for LLM matrix math.
2. SRAM Mapping: The MiniDOGE has dedicated high-speed internal memory to store the "Scratchpad" for Scrypt. Your GitHub repo should include a Memory Map test to see if you can load Quantized Weights (INT4 or GGUF) directly into that SRAM instead of the Scrypt data.
3. The Controller Bypass: Most miners use a simple control board (often running a stripped-down Linux). You'll need to test if your Custom Firmware can pass tensor data to the ASIC chips via the SPI or I2C bus without the mining software intercepting it.

## Specific Framework Folders to Add

- `/firmware_mod`: For the Bitstream or Microcode overrides to repurpose the Salsa20 cores.
- `/kernel_tests`: To test GEMM (General Matrix Multiplication) emulation using XOR/ADD logic.
- `/quantization_maps`: Specifically for Ternary or 2-bit weights, as the MiniDOGE won't have the precision for FP16.
- `/bridge_interface`: The code that allows a host PC (via USB/Ethernet) to send "prompts" to the Miner chips.

## The Big Challenge

Scrypt ASICs are fixed-function. Unlike a GPU, they don't have a "General Purpose" instruction set. You aren't just writing software; you are performing Instruction Set Emulation on hardwired logic.

© 2026 Steven Thompson — Original concept and architecture
First published: March 22, 2026
MIT License — attribution required