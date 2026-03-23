The research phase is complete. I have verified the hardware specifications of the MiniDOGE 1, confirmed the feasibility of using libp2p for embedded P2P networking, and identified layer sharding as the primary strategy for distributed LLM inference on these devices. [1, 2, 3, 4, 5, 6] 

A key technical insight discovered is that while the Salsa20/8 cores are hard-wired for the Scrypt algorithm, custom firmware can potentially leverage these blocks for line-rate encryption by managing the "Block Mix" iterations. [7, 8] 

Your commitment to repurposing e-waste for decentralized compute is a massive move. Given the technical complexity of the Salsa20/8 driver, would you like to explore the firmware reverse-engineering steps first, or should we look into the specific C++ libp2p configuration needed for the M01K board?

  

[1] [https://arxiv.org](https://arxiv.org/html/2505.18164v1)

[2] [https://www.newegg.com](https://www.newegg.com/goldshell-mini-doge-white/p/0X6-0983-00004)

[3] [https://www.zeusbtc.com](https://www.zeusbtc.com/Asic-Miner/Asic-Miner-Details.asp?ID=1488)

[4] [https://www.reddit.com](https://www.reddit.com/r/ipfs/comments/hwa084/sneak_peek_a_libp2p_like_protocol_written_in_c/)

[5] [https://github.com](https://github.com/Pier-Two/c-libp2p#:~:text=c%2Dlibp2p%20is%20an%20implementation%20of%20the%20libp2p,building%20blocks%20needed%20for%20peer%2Dto%2Dpeer%20networking%20applications.)

[6] [https://docs.vllm.ai](https://docs.vllm.ai/projects/ascend/en/main/user_guide/feature_guide/layer_sharding.html)

[7] [https://www.researchgate.net](https://www.researchgate.net/publication/361039194_Scrypt_Mining_with_ASICs)

[8] [https://medium.com](https://medium.com/@andreas.mai/how-i-hacked-into-goldshell-crypto-miners-df774574fb47)

© 2026 Steven Thompson — Original concept and architecture
First published: March 22, 2026
MIT License — attribution required