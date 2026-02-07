---
title: "Running Practical Worlds Onchain"
llm_description: "A single monolithic L1 that naively tries to process every transaction on every full validator will always hit a hard ceiling. Practical ultra-high-TPS architectures therefore combine horizontal parallelism (sharding / many sequencers), fast deterministic local execution (compiled & parallel VMs), sublinear consensus and signature primitives (BLS, aggregation), and data availability systems with erasure coding and sampling. Our goal is to present - (a) a reproducible numerical model; (b) a practical architecture that composes known building blocks; and (c) strategies for adding decen­tralized AI acceleration."
---





# Running Practical Worlds Onchain


## Abstract





## 1. Introduction & motivation

A single monolithic L1 that naively tries to process every transaction on every full validator will always hit a hard ceiling. Practical ultra-high-TPS architectures therefore combine **horizontal parallelism (sharding / many sequencers)**, **fast deterministic local execution (compiled & parallel VMs)**, **sublinear consensus and signature primitives (BLS, aggregation)**, and **data availability systems with erasure coding and sampling**. Our goal is to present: (a) a reproducible numerical model; (b) a practical architecture that composes known building blocks; and (c) strategies for adding decen­tralized AI acceleration.

## 2. Building-blocks&#x20;

* **Bullshark / Narwhal**: DAG-based ordering + data-separation that reduces latency and makes DAG BFT practical; useful pattern for high-throughput ordering with short fast paths. [arXiv](https://arxiv.org/pdf/2201.05677)
* **Block-STM** : demonstrates how a pre-ordered block can be executed in parallel at very high throughput by dynamically detecting dependencies (reported 100k–170k-level TPS in experiments). This informs the per-node execution throughput assumptions below. [arXiv](https://arxiv.org/abs/2203.06871)
* **BLS signature aggregation** :  compresses many validator signatures into one constant-size signature; crucial to eliminating O(N) signature overhead in consensus rounds. [IETF](https://www.ietf.org/archive/id/draft-irtf-cfrg-bls-signature-05.html)
* **Data availability & erasure coding** (Celestia+ideas) : shift from “everyone must store everything” to sampling-based availability checks and erasure coding reduces per-node bandwidth/storage pressure. [Celestia Docs](https://docs.celestia.org/learn/how-celestia-works/data-availability-layer)
* **Block propagation techniques** (e.g., multi-layer / Reed-Solomon shreds) : reduce leader upload bottlenecks and distribute block distribution load.

## 3. Model, assumptions & notation (so the math is reproducible)

We construct a numerical model for a target throughput `T_target` = **100,000,000 TPS** (100M TPS). Below are the core variables and the base assumptions used for the worked example.

**Variables / base assumptions**

* `T_target` = target TPS (100,000,000)
* `s_avg` = average compressed transaction size (bytes). Base case: **80 bytes** per tx (achievable with schema-aware + streaming compression for simple payments / short calldata). (We analyze sensitivity later.)
* `S_total = T_target * s_avg` = bytes/sec global.
* `B_global` = total _aggregate_ network traffic required (bits/sec) = `S_total * 8`.
* `shards` = number of independent ordered partitions (sequencer shards / execution domains). We pick `shards = 1000` for the base scenario → so per-shard TPS = `T_target / shards`.
* `t_per_core` = per-core execution capacity (TPS per CPU core) under a highly optimized parallel VM. We use a **conservative** `t_per_core = 5000` TPS/core inspired by Block-STM experimental figures (Block-STM achieved on the order of hundreds of thousands of TPS on multi-core machines; dividing their peak by thread counts gives per-core ballpark). [arXiv](https://arxiv.org/abs/2203.06871)
* `cores_per_machine` = 128 (dense server/edge machine with many cores).
* `E_machine = cores_per_machine * t_per_core` = execution capacity per sequencer/validator machine.
* Network & hardware efficiency assumptions are conservative; final section outlines experimental checks.

## 4. Step-by-step numerical calculation (explicit arithmetic)

We will compute all intermediate values carefully and explicitly.

**4.1 Global data rate**

1. `T_target = 100,000,000` transactions/sec.
2. `s_avg = 80` bytes/tx.
3. Compute bytes/sec: `S_total = T_target * s_avg = 100,000,000 * 80`.
   * `100,000,000 * 80 = 8,000,000,000` bytes/sec.
4. Convert to Gbps:
   * bits/sec = `8,000,000,000 * 8 = 64,000,000,000` bits/sec.
   * `64,000,000,000 bits/sec = 64 Gbps`.

**Result 4.1:** With 80-byte compressed txs, _global_ payload bandwidth required is **64 Gbps**.

***

**4.2 Partition into shards / sequencers**

1. Choose `shards = 1000`.
2. Per-shard TPS: `T_shard = T_target / shards = 100,000,000 / 1000 = 100,000` TPS per shard.
   * (Because `100,000,000 / 1000 = 100,000`.)

**4.3 Execution capacity per machine**

1. `t_per_core = 5,000` TPS/core (assumption from Block-STM scaling baseline). [arXiv](https://arxiv.org/abs/2203.06871)
2. `cores_per_machine = 128`.
3. `E_machine = cores_per_machine * t_per_core = 128 * 5,000`.
   * `128 * 5,000 = 640,000` TPS per machine.

**4.4 Machines needed**

1. `machines_per_shard = T_shard / E_machine = 100,000 / 640,000`.
   * `100,000 / 640,000 = 0.15625` machines per shard (i.e., **one machine could handle \~6 shards** at this efficiency).
2. `total_sequencer_machines = machines_per_shard * shards = 0.15625 * 1000 = 156.25` → **157 machines** (round up).

**Interpretation:** Using these (conservative) execution assumptions, \~**157 sequencer/executor machines** each with 128 cores could cover the execution workload for 100M TPS (with ample CPU headroom).

***

**4.5 Bandwidth per machine**

1. Global bandwidth (from 4.1) = 64 Gbps.
2. Spread across `total_sequencer_machines ≈ 156.25` → per-machine bandwidth:
   * `BW_machine = 64 Gbps / 156.25 = 0.4096 Gbps ≈ 410 Mbps`.
   * (Calculation: `64 / 156.25 = 0.4096`.)

**Result 4.5:** Each sequencer machine needs ≈ **410 Mbps** sustained network capacity to carry its share of payload , modest compared to modern datacenter NICs (10–100 Gbps).\
<br>

**4.6 Consensus & signature cost (high level)**

* If each _block_ (per shard) is aggregated using BLS into **one signature**, the validator-side signature verification cost is **O(1)** per block rather than O(n) per signer, this removes a major per-block per-validator overhead. BLS aggregation and constant-size commit proofs are essential to scaling to thousands of shards without multiplying signature traffic. [IETF](https://www.ietf.org/archive/id/draft-irtf-cfrg-bls-signature-05.html)
* Blocks per second per shard depend on batching strategy. Example: if the shard batches `B_tx = 1,000` tx per block, then `blocks_per_sec_shard = T_shard / B_tx = 100,000 / 1,000 = 100` blocks/sec. Each block requires verifying one aggregate signature (100 verify ops/sec per node per shard). If nodes verify across all shards they subscribe to, verification load becomes block\_rate × shards\_subscribed, but aggregate signatures keep that load tiny and constant.

***

## 5. Sensitivity & reality checks

**Bandwidth sensitivity**:

* If `s_avg` increases to 200 bytes (less compression), global bits/sec = `100,000,000*200*8 = 160 Gbps`. That implies per-machine bandwidth `≈ 1.02 Gbps` with the earlier 157 machines , still achievable in datacenter NICs, but more demanding for globally distributed validators.

**Execution sensitivity**:

* If `t_per_core` drops to 2500 TPS/core (less optimized VM), then `E_machine = 128*2500 = 320k` → machines needed double → \~314 machines , still in practical datacenter scale.

**Latency / propagation**:

* These numbers intentionally assume **data partitioning** so each shard/sequence domain shares responsibility for a fraction of global data. The key global limit remains **propagation & finality**: to achieve subsecond finality across globally distributed parties you must either (a) restrict the validator set per shard / use committees and aggregate commits, (b) use fast DAG ordering (e.g., Bullshark/Narwhal patterns) for low latency, and (c) use erasure coding / sampling for DA to avoid everyone needing full copies. [arXiv](https://arxiv.org/pdf/2201.05677) [Celestia Docs](https://docs.celestia.org/learn/how-celestia-works/data-availability-layer)

## 6. Putting the architecture pieces together (practical blueprint)

To reach **100M+ TPS** in practice, compose the following layers:

1. **Many sequenced execution partitions (shards/sectors/rollups)**
   * Partition the global transaction namespace into _S_ sequenced domains (100s–10k). Each domain has independent sequencers + committees. This linearizes scale.
2. **High-performance local execution engine**
   * Use a VM with: compiled (or JIT/native) smart contract execution, multi-threaded parallel engine (Block-STM style), deterministic scheduling for reproducibility. This maximizes `t_per_core`. [arXiv](https://arxiv.org/abs/2203.06871)
3. **Sublinear consensus & aggregated commits**
   * Use DAG ordering (Narwhal/Bullshark style) or pipelined HotStuff with BLS-based aggregated commits so that the consensus overhead per block is constant wrt validator count. [arXiv](https://arxiv.org/pdf/2201.05677)[I ETF](https://www.ietf.org/archive/id/draft-irtf-cfrg-bls-signature-05.html)
4. **Streaming compression + compact tx formats**
   * Use schema-aware binary formats + sliding window streaming compression to reduce `s_avg` as blocks are time-series, not independent random bytes.
5. **Distributed data availability (erasure coding + sampling)**
   * Use 2D Reed-Solomon or namespaced Merkle trees and data sampling to ensure availability without every node needing full copies; only a subset hold full data, others sample. [Celestia Docs](https://docs.celestia.org/learn/how-celestia-works/data-availability-layer)
6. **Balanced propagation (multi-layer shreds / shards)**
   * Avoid single-leader megablock broadcast. Use multi-layer dissemination (e.g., Turbine-style shreds and Reed-Solomon) to split, distribute and reconstruct blocks efficiently.&#x20;
7. **Hardware acceleration & optimized crypto libs**
   * Use vectorized curve libs (blst, relic) and accelerate heavy tasks (pairings, NTT) on GPUs / FPGAs for ZK proofs and other heavy crypto workloads. [GitHub](https://github.com/supranational/blst) [MIT CSAIL](https://people.csail.mit.edu/devadas/pubs/micro24_nocap.pdf)

## 7. How to add AI acceleration while maintaining decentralization

Many visions place powerful AI tightly coupled with on-chain systems. To add AI acceleration **without centralization** follow a layered approach:

### 7.1 Training: Federated learning + secure aggregation

* Use **federated learning** where model updates are computed locally by many participants and aggregated securely on-chain or by a committee using secure aggregation protocols (Bonawitz et al.). Secure aggregation prevents a central aggregator from seeing individual updates and scales with O(N log N) techniques (Turbo-Aggregate / other improvements). This keeps model training _decentralized_. [Google Research](https://research.google/pubs/practical-secure-aggregation-for-privacy-preserving-machine-learning/) [arXiv](https://arxiv.org/abs/2002.04156)

### 7.2 Inference: decentralized inference marketplaces + verifiable compute

* Real-time high-throughput inference can be offered by many _inference providers_ (GPU nodes) that stake to provide correct service. Use **verifiable computation** primitives: providers return results together with succinct proofs (SNARK/STARK) or attestations from TEEs so that consumers can verify correctness cheaply. Hardware acceleration (GPUs / FPGAs / ASICs) powers the inference, while cryptographic proofs + economic staking provide decentralization incentives. Research on accelerating zk systems shows GPU and hardware pipelines are practical for accelerating prover/verifier tasks. [MIT CSAIL](https://people.csail.mit.edu/devadas/pubs/micro24_nocap.pdf) [Wiley Online Library](https://onlinelibrary.wiley.com/doi/full/10.1002/eng2.12639)

### 7.3 Secure incentive & provenance on-chain

* Record model commitments, training round hashes, and contributor staking on-chain. Use on-chain governance for model updates. Distribute rewards algorithmically for honest contributions. Use secure aggregation to preserve contributor privacy (Bonawitz). [Google Research](https://research.google/pubs/practical-secure-aggregation-for-privacy-preserving-machine-learning/)

### 7.4 Maintain non-centralized control of model weights

* Avoid a single authoritative model owner: store model checkpoints as _content-addressed artifacts_ in the DA layer. Updates occur by a consensus-proven aggregation round. Participants can fork, retrain, or contribute alternative models, the blockchain stores provenance and incentive metadata.



## 8. Security & decentralization trade-offs (short)

* **Committee/Shard sizing**: Smaller per-shard committees improve latency but reduce Byzantine robustness. Use rotating committees and randomized VRF selection to mitigate capture risk.
* **DA sampling vs full replication**: Sampling reduces per-node cost but increases reliance on economic incentives & fraud proofs. Erasure coding + light sampling provides probabilistic guarantees. [Celestia Docs](https://docs.celestia.org/learn/how-celestia-works/data-availability-layer)
* **Hardware & centralized clouds**: Relying on large datacenter-grade machines reduces node count and increases efficiency, but risks concentration, mitigate by geographically dispersing sequencers and having permissionless alternatives with slower performance tiers.

## 9. Experimental roadmap & benchmarks (what to measure)

**Bench suite (per shard & global):**

* `throughput`: TPS achieved under steady state and burst.
* `latency`: 95/99/999th percentiles for commit/finality.
* `bandwidth`: sustained per-node & per-sequencer.
* `CPU / memory`: per core utilization, per-machine heatmaps.
* `DA detection`: probability of data withholding under sampling.
* `failure modes`: network partitions, sequential leader failure, equivocation.

**Example microbenchmarks to replicate the model**

1. Block-STM style execution tests to measure `t_per_core` on your VM and contract mix. [arXiv](https://arxiv.org/abs/2203.06871)
2. BLS aggregation verification per second using blst & pairings to estimate aggregate verification throughput. [GitHub](https://github.com/supranational/blst)
3. Erasure coding / shreds throughput & latency experiments (Turbine style).



## 10. Conclusions & practical takeaways

* **Theoretical possibility**: With careful composition (thousands of sequenced partitions, aggressive streaming compression, BLS aggregation, Block-STM style parallel execution, erasure-coded DA, and compact propagation), **100M TPS is achievable in principle** in datacenter-backed, well-provisioned deployments. The arithmetic above shows that execution & bandwidth are not the hard, impossible part,  they become practical with partitioning and compression.
* **Real-world caveats**: Global decentralization/participation, cross-domain messaging, finality guarantees, and censorship resistance remain the true operational constraints. The bigger obstacles are _synchronization_ and _security trade-offs_ when pushing to these extremes.
* **AI acceleration** can be integrated non-centrally via federated learning, secure aggregation, decentralized inference markets, and verifiable compute, combined with hardware acceleration for crypto and ML workloads.

***

## References

* BullShark: “BullShark: DAG BFT Protocols Made Practical.” A. Spiegelman et al. (Bullshark / Narwhal analyses). [arXiv](https://arxiv.org/pdf/2201.05677)
* Block-STM: “Block-STM: Scaling Blockchain Execution by Turning Ordering Curse to a Performance Blessing.” R. Gelashvili et al. (Aptos / Diem research). [arXiv](https://arxiv.org/abs/2203.06871)
* BLS Signatures: IETF draft and practical references on BLS aggregation for consensus (Boneh-Lynn-Shacham families / BLS12-381). [IETF](https://www.ietf.org/archive/id/draft-irtf-cfrg-bls-signature-05.html) [eth2book.info](https://eth2book.info/latest/part2/building_blocks/signatures/)
* Data Availability & Erasure Coding: Celestia docs and DA literature (namespaced Merkle trees + RS coding). [Celestia Docs](https://docs.celestia.org/learn/how-celestia-works/data-availability-layer)
* Turbine / shreds style propagation: Solana Turbine exposition & Reed-Solomon shreds method. [Solana](https://solana.com/news/turbine---solana-s-block-propagation-protocol-solves-the-scalability-trilemma)
* Practical Secure Aggregation for Federated Learning: Bonawitz et al., Google research (secure aggregation). [Google Research](https://research.google/pubs/practical-secure-aggregation-for-privacy-preserving-machine-learning/)
* Hardware acceleration for ZK and crypto kernels: recent papers showing GPU/FPGA acceleration for SNARKs/NTT/KZG kernels. [MIT CSAIL](https://people.csail.mit.edu/devadas/pubs/micro24_nocap.pdf) [Wiley Online Library](https://onlinelibrary.wiley.com/doi/full/10.1002/eng2.12639)

