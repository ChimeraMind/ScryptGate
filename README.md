# ScryptGate

Modern AI agent stacks face a fundamental security problem — request authentication,
rate limiting, and model integrity verification are handled entirely in software.
Software can be bypassed, spoofed, or overwhelmed. This project proposes and
documents a novel architecture that places a repurposed Scrypt mining ASIC
(Goldshell MiniDOGE 1, 185MH/s) as a hardware security primitive between the
network edge and the AI inference stack.
