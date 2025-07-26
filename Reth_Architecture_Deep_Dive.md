

# **Reth: An Architectural and Performance Analysis of a Next-Generation Ethereum Execution Client**

## **Section 1: Executive Summary & Strategic Positioning**

### **1.1 Introduction to Reth**

Reth (short for Rust Ethereum) is a next-generation Ethereum execution layer (EL) client, distinguished by its implementation in the Rust programming language and a foundational design philosophy centered on modularity, performance, and contributor-friendliness.1 Developed initially by Paradigm and now supported by a broad open-source community, Reth has matured from an experimental project into a production-ready client suitable for mission-critical applications, including institutional staking and high-availability RPC services.2 As a full Ethereum node, it is compatible with all consensus layer (CL) clients that support the Engine API, allowing it to fully synchronize and interact with the Ethereum blockchain and its historical state.1

This report provides an exhaustive technical analysis of Reth, arguing that it represents a strategic architectural evolution in Ethereum client design. It is engineered not merely as an alternative to existing clients but as a foundational platform built to meet the escalating demands of Ethereum's rollup-centric roadmap. By prioritizing extreme performance, unparalleled customizability, and operational stability, Reth is tailored for a diverse set of "power users," including Layer-2 (L2) solutions, high-throughput RPC providers, MEV (Maximal Extractable Value) searchers, and data indexers.1

### **1.2 Critical Clarification: Reth (Client) vs. rETH (Token)**

A point of recurring ambiguity in public discourse necessitates immediate clarification. This report is exclusively concerned with **Reth**, the software client.1 It should not be confused with

**rETH**, which is the liquid staking token issued by the Rocket Pool protocol.7 While the ecosystems of staking and client software are interconnected, the subjects themselves are distinct. The frequent conflation of these terms in online forums and search results underscores a need for precision.7 All subsequent mentions of Reth in this document refer to the Rust Ethereum execution client.

## **Section 2: Comparative Analysis: Reth vs. Geth**

### **2.1 Foundational Philosophies and Language Choice**

The differences between Reth and Geth (Go Ethereum) extend beyond programming language to their core architectural philosophies, which are products of their respective eras and design goals.

**Geth**, written in Go, is the incumbent and most widely used execution client. Its long history and battle-hardened stability have made it the de facto reference implementation for the Ethereum protocol and the canonical client for major ecosystems like the OP Stack.10 Its architecture, having evolved since Ethereum's inception, is comparatively monolithic, making deep customization or component reuse challenging without forking the entire codebase.1

**Reth**, by contrast, is a modern challenger written in Rust. Its design philosophy is explicitly "modular by design," with the stated goal for every component to be a reusable, well-documented, and independently testable library.1 This approach is a deliberate strategy to foster a vibrant developer ecosystem, enabling projects to build highly customized node software with minimal overhead.6 Reth's development acknowledges the foundational work of Geth and other clients like Erigon, but aims to build upon their lessons with a fresh, modular architecture and the performance and safety guarantees of Rust.1

### **2.2 Quantitative Performance Benchmarks**

Empirical data reveals a significant performance gap between Reth and older clients, particularly in high-throughput scenarios.

* **RPC Throughput:** Reth consistently demonstrates superior requests-per-second (RPS) capabilities. Benchmarks from Chainstack show a Reth full node achieving 3,221.67 RPS, nearly double Geth's 1,670 RPS. The advantage is even more pronounced for archive nodes, where Reth handles 9,680.7 RPS.13 For developers and MEV searchers, Reth's  
  Debug & Trace API is a standout, delivering over 564 RPS, more than ten times faster than Erigon's 48.3 RPS.14  
* **Synchronization Speed:** Reth was engineered to be one of the fastest syncing clients. It employs a "staged sync" architecture, pioneered by Erigon, which has been observed to be significantly faster for bootstrapping new nodes from genesis.1 Early reports indicated a full mainnet archive sync could be completed in approximately 50 hours on suitable hardware, a benchmark that established it as a leader in sync performance.15

### **2.3 Database Architecture and The Storage Dilemma**

The most critical architectural divergence between Reth and Geth lies in the database layer, which has profound real-world implications for node operators, especially on high-throughput L2 networks.

Geth utilizes LevelDB as its key-value store, combined with a canonical Merkle-Patricia Trie (MPT) data structure. While robust, this design leads to significant write amplification and inefficient storage of historical state. For large-scale operators, this manifests as two primary operational challenges: unsustainable disk space requirements and database instability. A case study from the Base L2 network revealed that a Geth archive node's disk usage was growing by approximately 560 GB per week, with projections showing it would exceed the practical limits of a 30 TB NVMe drive by mid-2025.10 Furthermore, as write traffic increases, Geth's database is prone to long-running compaction processes, which can take a node offline for over 30 minutes, causing it to fall behind the chain tip and impacting service availability.10

Reth directly addresses these pain points. Inspired by Erigon, it uses the MDBX database engine combined with a more efficient data layout and the staged-sync model.1 This architecture is purpose-built to minimize storage overhead and maintain performance under load. The results are dramatic. In the same Base L2 environment, a Reth archive node was 83.5% smaller than its Geth counterpart (2.74 TB vs. 16.61 TB) and exhibited a weekly disk growth rate that was 91.1% lower (around 50 GB).10 This architectural superiority not only reduces hardware costs but also eliminates the operational instability of Geth's database compactions, a factor that allowed the Base team to avoid multiple outages by running Reth nodes in parallel with Geth.10

The choice of database architecture is therefore not merely a technical detail but a strategic one. Geth's design, while proven, shows signs of strain under the high-throughput conditions of modern L2s. Reth's architecture is fundamentally more scalable in terms of storage, positioning it as a more viable and cost-effective solution for any entity requiring reliable, long-term access to historical chain data, such as L2s, indexers, and analytics platforms.

| Metric | Geth (Archival on Base) | Reth (Archival on Base) | Improvement | Source |
| :---- | :---- | :---- | :---- | :---- |
| **Current Disk Usage** | 16.61 TB | 2.74 TB | 83.5% | 10 |
| **Weekly Disk Growth** | \~560 GB | \~50 GB | 91.1% | 10 |
| **Block Building p50** | 374 ms | 201 ms | 46.3% | 10 |
| **RPC RPS (Full Node)** | 1,670 | 3,221 | 92.9% | 13 |
| **RPC RPS (Archive Node)** | 3,999 (Erigon) | 9,680 | 142.3% (vs Erigon) | 14 |
| **Debug & Trace RPS** | 48.3 (Erigon) | 564 | \>1000% (vs Erigon) | 14 |

## **Section 3: Core Architecture and Rust-Specific Design**

### **3.1 The "Node as a Library" Paradigm**

Reth's most defining architectural principle is its conception as a "node as a library" or software development kit (SDK).1 Every major function of the node—from networking to the EVM—is encapsulated in a distinct, reusable software package (a "crate" in Rust terminology) with a permissive Apache/MIT license.3

This paradigm is realized through the **Node Builder** pattern, a powerful API that allows developers to programmatically construct a node instance, swapping out default components for custom implementations or modifying their behavior.12 This enables deep customization, such as implementing unique consensus rules or adding new RPC endpoints, without needing to fork and maintain a separate version of the entire codebase. This approach stands in stark contrast to the more rigid, monolithic structures of older clients and is a key enabler for the rapid innovation seen in the L2 and app-chain ecosystem.

### **3.2 Key Components Deep-Dive**

Reth's modularity is evident in its clear separation of components, each with a distinct responsibility.

| Component | Primary Responsibility | Key Rust Crates |
| :---- | :---- | :---- |
| **Database** | Efficient, transactional key-value storage of all chain data. | reth\_db, reth\_libmdbx |
| **Networking** | P2P communication, peer discovery, and data synchronization. | reth\_network, reth\_discv4 |
| **EVM** | Transaction execution and state transitions. | reth\_evm, revm |
| **Consensus** | Block validation according to protocol rules (e.g., PoS). | reth\_consensus |
| **RPC** | Exposing node functionality via JSON-RPC over HTTP/WS. | reth\_rpc, reth\_rpc\_api |
| **Transaction Pool** | Management of pending transactions (mempool). | reth\_transaction\_pool |
| **Node Builder** | Orchestration and construction of custom node instances. | reth\_node\_core |

* **Database Layer:** Reth leverages **MDBX**, a high-performance embedded transactional database known for its speed and reliability.1 MDBX provides features like automatic on-the-fly database size adjustment and continuous, zero-overhead compactification, which are critical for managing the large state of a blockchain node.16 Reth abstracts the database backend behind a  
  Database trait, which allows for future flexibility to support other engines like redb.17 To further optimize storage, Reth employs custom, space-efficient  
  **Codecs** (e.g., Compact), which reduce the on-disk size of data types by eliminating redundant information like leading zeros in large numbers.17  
* **Networking Stack:** The P2P stack is engineered as a sophisticated hierarchy of asynchronous Rust futures and event streams, designed for high concurrency.5 At the top sits the  
  NetworkManager, a state machine that orchestrates the entire stack. It drives a Swarm component responsible for managing peer discovery, session lifecycles, and message passing.5 This design follows an actor-like pattern, where a clonable, thread-safe  
  NetworkHandle is shared across the node, allowing other components (like the synchronizer) to safely interact with the networking layer.5  
* **Consensus and Execution:** Following the post-Merge Ethereum architecture, Reth cleanly separates its components. The Consensus component is responsible for validating blocks according to the network's rules, while the EVM component, powered by the Revm engine, is responsible for the critical task of executing transactions and applying state transitions.12

### **3.3 The Rust Advantage: Performance, Safety, and Concurrency**

The choice of Rust is not a superficial implementation detail; it is a foundational pillar that enables Reth's ambitious architectural goals.

* **Performance:** Rust provides low-level control over memory without the overhead of a garbage collector, which is essential for performance-critical systems software.21 Idiomatic Rust code naturally encourages high-performance patterns that are central to Reth's speed, such as zero-copy operations (passing data by reference to avoid costly duplication), designing cache-friendly data structures, and utilizing smart allocation patterns to minimize pressure on the system's memory allocator.21  
* **Safety and Reliability:** Rust's ownership model and compile-time borrow checker are its most celebrated features, as they eliminate entire classes of memory-related bugs, such as null pointer dereferences, buffer overflows, and data races.22 For a consensus-critical application like a blockchain client, this is paramount. It provides a high degree of assurance that the software is free from subtle memory corruption bugs that could lead to database corruption, node crashes, or even catastrophic consensus failures.15  
* **Concurrency:** The complexity of a modern blockchain node requires a robust concurrency model. Reth's architecture, with its many interacting asynchronous tasks, is a testament to this. Rust's "fearless concurrency" provides language-level guarantees that make it possible to build such complex, multi-threaded systems with a much higher degree of safety and confidence than in languages where concurrency is more error-prone.5 The compiler itself acts as a safeguard against common concurrency pitfalls, allowing developers to focus on the logic of the system rather than the intricacies of thread synchronization. This direct link between the language's features and the client's architecture underpins Reth's ability to be both highly performant and robust.

## **Section 4: The Execution Engine: A Closer Look at Revm**

At the heart of Reth's execution capabilities lies Revm, a standalone, high-performance Ethereum Virtual Machine (EVM) implementation also written in Rust. Revm is not merely a component of Reth; it is a powerhouse in its own right, widely adopted across the Ethereum ecosystem by tools like Foundry, major block builders, L2s, and even zkVM projects.23

### **4.1 Revm's Design and Performance Characteristics**

Revm was created with three guiding principles: speed, simplicity, and stability.24 It features a highly optimized execution loop and instruction handling, making it one of the fastest EVM implementations available.24 This performance is a primary driver of Reth's overall speed, particularly in execution-heavy workloads like processing large blocks or running complex simulations.10 While Revm is fundamentally an interpreter, its design does not preclude future performance enhancements. Research into alternative execution strategies like Just-In-Time (JIT) or Ahead-Of-Time (AOT) compilation using frameworks like MLIR has shown potential for 3-6x throughput improvements over the baseline Revm in specific benchmarks, indicating a clear path for future optimization.26

### **4.2 The Revm Framework API: A Gateway to Multi-Chain Support**

Crucially, Revm is architected not just as a static executor but as a flexible **framework** for building custom EVM variants.23 This is a strategic design choice that directly enables Reth's multi-chain ambitions. The framework API allows developers to extend the EVM's logic, add or modify precompiles, and support different state transition functions.

This capability is best demonstrated by op-revm, a specialized version of Revm tailored for the OP Stack.23 Reth leverages this extensibility to seamlessly support OP Stack chains like Base and Optimism.10 The ability to support new EVM-based chains is not a matter of a complex, bespoke integration but rather of implementing a new configuration within the Revm framework. This architectural foresight makes Reth's multi-chain support a feature with low maintenance overhead and high adaptability, positioning it as an ideal client for the rapidly expanding L2 ecosystem.

### **4.3 Advanced Tracing and Inspection**

Revm comes equipped with powerful, built-in capabilities for transaction inspection and tracing, which are exposed through Reth's API.23 The

revm-trace library provides a high-performance, multi-threaded environment for simulating and analyzing transactions in detail.30 These features are indispensable for developers using tools like Foundry for smart contract testing and debugging. They are also critical for advanced users like MEV searchers, who rely on deep transaction analysis to identify opportunities. By integrating Revm, Reth provides these powerful tools to its users out of the box, solidifying its position as a client built for expert use cases.

## **Section 5: Network Protocols and Advanced Features**

### **5.1 P2P Layer and Block Propagation**

Reth implements the standard devp2p protocol suite that forms the backbone of Ethereum's peer-to-peer network.32 This includes:

* **RLPx Transport Protocol:** A transport layer running over TCP that provides encrypted and authenticated communication sessions between nodes.18  
* **Peer Discovery:** Reth uses the Kademlia-based Discovery v4 protocol (discv4) over UDP to find and connect to other nodes in the network.5 Support for the newer  
  discv5 is also being progressively enabled to align with the consensus layer's networking stack.5  
* **Wire Protocol:** Block and transaction data are exchanged using the eth sub-protocol, which defines messages for synchronization (GetBlockHeaders, BlockBodies) and transaction gossip (NewPooledTransactionHashes, Transactions).5

While the protocols are standard, Reth's implementation is modern. It is built as a hierarchy of concurrent, event-driven state machines managed by the tokio asynchronous runtime, a design that leverages Rust's safety features to handle the inherent complexity of P2P networking robustly.5 It is worth noting that while Reth uses the standard gossip mechanism for block propagation, active research in the broader Ethereum community is exploring more advanced techniques like Random Linear Network Coding (RLNC) to dramatically reduce propagation latency and bandwidth consumption, though these are not yet implemented in any major client.35

### **5.2 RPC and Transport Layer**

Reth provides a comprehensive JSON-RPC server that allows users and applications to interact with the node. This service runs over standard TCP, supporting HTTP on the default port 8545 and WebSockets (for real-time subscriptions) on port 8546\.19

#### **5.2.1 QUIC Transport Analysis**

Based on a thorough review of the available documentation and code repositories, there is **no evidence that Reth currently supports the QUIC transport protocol for its RPC services**, nor are there public plans for its immediate implementation.38

QUIC is a modern transport protocol that runs over UDP and offers several advantages over TCP for latency-sensitive applications like RPC. Its key benefits include a faster connection handshake (requiring only one round-trip, or zero for subsequent connections) and the elimination of head-of-line blocking, where a single lost packet can delay all subsequent data streams.41 For high-frequency users such as MEV searchers or trading bots, these latency improvements could be significant. While the Rust ecosystem contains libraries like

quic-rpc that could facilitate such an implementation, this remains a potential future optimization path for Reth rather than a current feature.38

### **5.3 Linux Kernel Optimizations**

#### **5.3.1 XDP (eXpress Data Path) Analysis**

Similarly, there is **no evidence that Reth has implemented any specific optimizations for or integrations with the Linux XDP (eXpress Data Path) feature**.40

XDP is a high-performance data path within the Linux kernel that allows eBPF programs to process network packets at the earliest possible point—directly within the network interface card (NIC) driver.43 This enables line-rate packet filtering and redirection, bypassing most of the kernel's slower networking stack. Its primary use cases include high-speed load balancing and highly effective DDoS mitigation, as malicious traffic can be dropped before it consumes any CPU resources on the host machine.45

For a high-throughput node client like Reth, especially one run by public-facing infrastructure providers, an XDP-based defense layer could provide a powerful mechanism to harden against network-level denial-of-service attacks. Like QUIC, this represents a notable area for future performance and security engineering rather than an existing feature.

## **Section 6: Achieving High Performance: Optimization Strategies**

Reth's development is characterized by an aggressive pursuit of performance, with a roadmap that encompasses both maximizing the capabilities of a single node (vertical scaling) and designing for a distributed future (horizontal scaling). The explicit goal is to "break the gigagas barrier" and achieve execution speeds in excess of 1 gigagas per second.6

### **6.1 Vertical Scaling: Maximizing Single-Node Performance**

* **Parallel EVM Execution:** A significant portion of a node's processing time during block execution is spent inside the EVM interpreter, often accounting for around 50% of the total time.46 To address this bottleneck, Reth is actively developing a parallel execution engine. The strategy involves using an optimistic execution model based on Block-STM for live synchronization and pre-calculating an optimal execution schedule for historical syncs.6 The potential for this optimization is substantial; related research projects like  
  pevm have demonstrated up to a 22x speedup in raw EVM execution using a parallel, Rust-based executor.47  
* **State Root Calculation:** The second major performance bottleneck is the calculation of the world state root after executing the transactions in a block. This process can consume over 75% of the total time required to seal a block.46 Reth's architecture already provides an advantage by decoupling transaction execution from state root calculation. Further optimizations are being actively explored, including using faster hash functions like Blake3 (instead of Keccak256), implementing wider Tries to reduce I/O amplification, and, for L2s, potentially diverging from L1 behavior by calculating the state root less frequently or in a "trailing" fashion, lagging a few blocks behind execution.46  
* **Memory and Allocation Patterns:** At a lower level, Reth's performance is built on a foundation of idiomatic Rust optimization techniques.21 These include a disciplined approach to memory management, such as using zero-copy operations to avoid redundant data duplication, designing data structures for optimal CPU cache alignment, and employing smart allocation strategies like pre-allocation and object pooling to minimize expensive interactions with the operating system's memory allocator. The staged-sync model also naturally facilitates batching database writes, which can be tuned via configuration parameters to balance memory usage and disk I/O for optimal performance on specific hardware.48

### **6.2 Horizontal Scaling Vision: "Cloud-native Reth"**

The Reth team recognizes that vertical scaling, while crucial, has ultimate physical limits. The long-term vision is to transcend the single-machine paradigm by evolving Reth from a monolithic node application into a decomposable, **cloud-native service stack**.46

This forward-looking architecture would break the node into a set of distinct, independently scalable services—for example, a P2P gateway service, a transaction execution service, and a state root calculation service. This model, common in modern serverless databases like NeonDB and CockroachDB, would allow a Reth deployment to scale horizontally across multiple machines, using cloud infrastructure to handle workloads that far exceed the capacity of any single server.46 This vision is the ultimate expression of Reth's modular philosophy, positioning it not just as a better node client but as a foundational, web-scale infrastructure for a future where blockchain interaction is ubiquitous.

## **Section 7: Ecosystem Adoption and Impact**

Since its launch, Reth has seen rapid and significant adoption across the Ethereum ecosystem, validating its design goals of performance and modularity. It has become a trusted component for institutional infrastructure providers and a foundational building block for a new generation of L2s and rollups.

### **7.1 Infrastructure Providers and Validators**

Reth is used in production by some of the industry's leading infrastructure companies, who choose it to solve real-world challenges of scale, cost, and reliability.4

* **Coinbase Staking:** Trusts and utilizes Reth for its institutional-grade, production staking infrastructure, where uptime and reliability are paramount.4  
* **Alchemy:** Employs Reth to power its high-throughput RPC services, benefiting from its superior performance and low latency under load.49  
* **Chainstack:** Offers Reth as a premium client option for dedicated nodes across Ethereum, BNB Smart Chain, Base, and Optimism, publicly citing its significant performance advantages in RPC throughput.14  
* **Conduit:** A Rollup-as-a-Service (RaaS) provider, uses Reth to meet the high-throughput and fast block time requirements of its clients.49

The public endorsement from the Base team, which detailed how running Reth in parallel with Geth prevented multiple service outages, serves as a powerful testament to its operational stability and architectural superiority in demanding environments.10

### **7.2 Foundation for L2s and Rollups**

Reth's modular SDK has proven to be a catalyst for L2 and rollup development. It allows teams to build custom, high-performance clients with a fraction of the engineering effort that would be required to fork and maintain a monolithic codebase.4

| Project Name | Backed By | Chain Type | Description | Approx. LoC |
| :---- | :---- | :---- | :---- | :---- |
| **Base Node** | Coinbase | L2 Rollup | Coinbase's L2 scaling solution node implementation. | \~3,000 |
| **BSC Reth** | Binance | L1 | BNB Smart Chain execution client with custom consensus. | \~6,000 |
| **Gnosis Reth** | Gnosis | L1 | Gnosis Chain's xDai-compatible execution client. | \~5,000 |
| **Bera Reth** | Berachain | L1 | Berachain's high-performance EVM node with custom features. | \~1,000 |
| **rbuilder** | N/A (Open Source) | MEV Tooling | An open-source block builder built on Reth. | N/A |
| **Ress** | Paradigm | Research | An experimental stateless Ethereum client. | \<4,000 |

This table, with data sourced from 4, illustrates the power of the Reth SDK. Projects are able to achieve deep customization and implement chain-specific logic with a relatively small and maintainable amount of code.

### **7.3 Emerging and Bleeding-Edge Use Cases**

Reth's flexible architecture has also positioned it at the forefront of protocol research and emerging technologies.

* **Stateless Ethereum:** The **Ress** ("Reth Stateless") client, a fully validating stateless node with a disk footprint of only 14 GB, was built on top of Reth in under 4,000 lines of code.51 This demonstrates that Reth's SDK is robust and flexible enough to serve as a platform for cutting-edge research into Ethereum's long-term scaling solutions.  
* **Zero-Knowledge (ZK) Proofs:** Reth is explicitly designed to be "ZK & Stateless Ready".49 Its performance and modularity make it an ideal component for ZK-centric applications. For example,  
  **SP1**, a high-performance zkVM for proving arbitrary Rust programs, highlights how Reth's ecosystem enables scalable ZK applications for Ethereum.49  
* **High-Performance Indexing:** The performance benefits of Reth's database architecture are not limited to the node itself. The reth-indexer project, which reads data directly from a Reth node's database, demonstrated indexing performance over 70 times faster than a comparable solution using The Graph's hosted service, showcasing the power of direct data access.53

## **Section 8: Conclusion and Forward Outlook**

### **8.1 Synthesis**

Reth has successfully transitioned from a promising alpha project to a formidable, production-grade Ethereum execution client. It has validated its core design principles by delivering verifiable, order-of-magnitude improvements in performance, storage efficiency, and operational stability compared to incumbent clients. Its defining characteristics are its hyper-performant Rust-based implementation and a deeply modular, SDK-first architecture. This combination has enabled it to gain the trust of major infrastructure providers and become the go-to foundation for a new generation of L2s and rollups.

### **8.2 Strategic Importance**

The emergence and adoption of Reth are strategically significant for the Ethereum ecosystem for two primary reasons:

1. **Enhancing Client Diversity:** Reth provides a robust and high-performance alternative to Geth, which has historically held a dominant market share. This diversity is critical for the overall health and resilience of the Ethereum network, mitigating the risks associated with a potential bug in a single, dominant client.1  
2. **Enabling the Rollup-Centric Roadmap:** Ethereum's future is inextricably linked to the success of its L2 ecosystem. These rollups require node software that is not only performant but also highly customizable and easy to maintain. Reth's SDK-centric design directly addresses this need, providing the ideal toolkit for building the diverse, specialized, and high-throughput chains that will scale Ethereum.6

### **8.3 Recommendations for Adopters**

Based on this analysis, different ecosystem participants should consider Reth for the following reasons:

* **L2/Rollup Developers:** Reth should be the default consideration for building a new EVM-based chain. The flexibility, low maintenance overhead, and performance of its SDK have been proven by projects like Base, BSC, and Gnosis, offering a significantly faster path to market than forking a monolithic client.  
* **RPC Providers and Indexers:** For any service where RPC throughput, latency, and data access speed are critical business metrics, Reth presents a compelling case. Its dramatic performance advantages can translate directly into lower operational costs, better service reliability, and a superior end-user experience.  
* **Institutional Stakers:** For validators, where uptime and stability are non-negotiable, Reth has demonstrated its production-readiness. Its efficient resource utilization and resistance to issues like database corruption under load make it a reliable choice for securing significant value.

### **8.4 Future Trajectory**

Reth's roadmap remains ambitious, with a clear focus on pushing the boundaries of blockchain performance toward "web scale".6 Key areas of future development that will shape the next phase of its evolution include:

* **Advanced Execution Optimizations:** The integration of a Parallel EVM and exploration of JIT/AOT compilation are the next major steps in enhancing execution speed.  
* **State Growth Solutions:** The AlphaNet testbed will serve as a proving ground for novel solutions to the state bloat problem, such as state expiry and rent, which are critical for Ethereum's long-term sustainability.  
* **The Cloud-Native Vision:** The most forward-looking aspect of the roadmap is the plan to decompose the node into a horizontally scalable, cloud-native service stack. This represents a fundamental paradigm shift in how blockchain infrastructure is designed and deployed, positioning Reth to meet the demands of a future where on-chain activity is orders of magnitude greater than it is today.

#### **Works cited**

1. Introducing Reth \- Paradigm, accessed July 9, 2025, [https://www.paradigm.xyz/2022/12/reth](https://www.paradigm.xyz/2022/12/reth)  
2. Reth, accessed July 9, 2025, [https://reth.rs/intro.html](https://reth.rs/intro.html)  
3. paradigmxyz/reth: Modular, contributor-friendly and blazing-fast implementation of the Ethereum protocol, in Rust \- GitHub, accessed July 9, 2025, [https://github.com/paradigmxyz/reth](https://github.com/paradigmxyz/reth)  
4. Reth, accessed July 9, 2025, [https://reth.rs/](https://reth.rs/)  
5. Diving into the Reth p2p stack | Fiber Network, accessed July 9, 2025, [https://fiber.chainbound.io/blog/reth-p2p/](https://fiber.chainbound.io/blog/reth-p2p/)  
6. Reth AlphaNet \- Paradigm, accessed July 9, 2025, [https://www.paradigm.xyz/2024/04/reth-alphanet](https://www.paradigm.xyz/2024/04/reth-alphanet)  
7. Video: Understanding rETH \- rETH \- Rocket Pool rETH Integration \- Blockchain and Smart Contract Development Courses \- Cyfrin Updraft, accessed July 9, 2025, [https://updraft.cyfrin.io/courses/rocket-pool-reth-integration/understanding-reth/reth](https://updraft.cyfrin.io/courses/rocket-pool-reth-integration/understanding-reth/reth)  
8. rETH \- Cryptocurrencies \- IQ.wiki, accessed July 9, 2025, [https://iq.wiki/wiki/reth](https://iq.wiki/wiki/reth)  
9. Understanding rETH \- Value Of rETH \- Rocket Pool rETH Integration \- Video, accessed July 9, 2025, [https://updraft.cyfrin.io/courses/rocket-pool-reth-integration/understanding-reth/value-of-reth](https://updraft.cyfrin.io/courses/rocket-pool-reth-integration/understanding-reth/value-of-reth)  
10. Scaling Base With Reth \- Base Engineering Blog, accessed July 9, 2025, [https://blog.base.dev/scaling-base-with-reth](https://blog.base.dev/scaling-base-with-reth)  
11. Ethereum Clients: Erigon vs Geth \- A Detailed Comparison | Guides \- GoldRush, accessed July 9, 2025, [https://goldrush.dev/guides/erigon-vs-geth-unravelling-the-dynamics-of-ethereum-clients/](https://goldrush.dev/guides/erigon-vs-geth-unravelling-the-dynamics-of-ethereum-clients/)  
12. Reth for Developers, accessed July 9, 2025, [https://reth.rs/sdk/overview](https://reth.rs/sdk/overview)  
13. chainstack.com, accessed July 9, 2025, [https://chainstack.com/chainstack-introduces-reth-support-for-dedicated-nodes/\#:\~:text=Reth%20performance%20benchmarks,Reth%20Archive%20node%3A%209%2C680.7%20RPS](https://chainstack.com/chainstack-introduces-reth-support-for-dedicated-nodes/#:~:text=Reth%20performance%20benchmarks,Reth%20Archive%20node%3A%209%2C680.7%20RPS)  
14. Chainstack introduces early access Reth support for Dedicated nodes, accessed July 9, 2025, [https://chainstack.com/chainstack-introduces-reth-support-for-dedicated-nodes/](https://chainstack.com/chainstack-introduces-reth-support-for-dedicated-nodes/)  
15. Inside Paradigm's plan to make Ethereum more resilient \- The Block, accessed July 9, 2025, [https://www.theblock.co/post/235941/inside-paradigms-plan-to-make-ethereum-more-resilient](https://www.theblock.co/post/235941/inside-paradigms-plan-to-make-ethereum-more-resilient)  
16. erthink/libmdbx: A potentia Ad Actum Fullfast transactional key-value memory-mapped B-Tree storage engine without WAL Surpasses the legendary LMDB in terms of reliability, features and performance \- GitHub, accessed July 9, 2025, [https://github.com/erthink/libmdbx](https://github.com/erthink/libmdbx)  
17. reth/docs/design/database.md at main \- GitHub, accessed July 9, 2025, [https://github.com/paradigmxyz/reth/blob/main/docs/design/database.md](https://github.com/paradigmxyz/reth/blob/main/docs/design/database.md)  
18. Modifying reth to build the fastest transaction network on BSC and Polygon | Merkle.io, accessed July 9, 2025, [https://www.merkle.io/blog/modifying-reth-to-build-the-fastest-transaction-network-on-bsc-and-polygon](https://www.merkle.io/blog/modifying-reth-to-build-the-fastest-transaction-network-on-bsc-and-polygon)  
19. Node Components \- Reth, accessed July 9, 2025, [https://reth.rs/sdk/node-components/](https://reth.rs/sdk/node-components/)  
20. Running Reth on Ethereum Mainnet or testnets, accessed July 9, 2025, [https://reth.rs/run/mainnet.html](https://reth.rs/run/mainnet.html)  
21. Unofficial Guide to Rust Optimization Techniques | by Yong kang ..., accessed July 9, 2025, [https://extremelysunnyyk.medium.com/unofficial-guide-to-rust-optimization-techniques-ec3bd54c5bc0](https://extremelysunnyyk.medium.com/unofficial-guide-to-rust-optimization-techniques-ec3bd54c5bc0)  
22. Questioning Rust, Understanding Rust, Falling in Love with Rust: It's Not Too Late to Start with Rust Now | by TinTinLand \- Medium, accessed July 9, 2025, [https://medium.com/tintinland/questioning-rust-understanding-rust-falling-in-love-with-rust-its-not-too-late-to-start-with-f66db5c95511](https://medium.com/tintinland/questioning-rust-understanding-rust-falling-in-love-with-rust-its-not-too-late-to-start-with-f66db5c95511)  
23. bluealloy/revm: Rust implementation of the Ethereum ... \- GitHub, accessed July 9, 2025, [https://github.com/bluealloy/revm](https://github.com/bluealloy/revm)  
24. revm \- Rustfinity, accessed July 9, 2025, [https://www.rustfinity.com/open-source/revm](https://www.rustfinity.com/open-source/revm)  
25. revm \- crates.io: Rust Package Registry, accessed July 9, 2025, [https://crates.io/crates/revm/3.3.0](https://crates.io/crates/revm/3.3.0)  
26. EVM performance boosts with MLIR \- LambdaClass Blog, accessed July 9, 2025, [https://blog.lambdaclass.com/evm-performance-boosts-with-mlir/](https://blog.lambdaclass.com/evm-performance-boosts-with-mlir/)  
27. reth\_evm \- Rust, accessed July 9, 2025, [https://reth.rs/docs/reth\_evm/index.html](https://reth.rs/docs/reth_evm/index.html)  
28. Running Reth on OP Stack chains, accessed July 9, 2025, [https://reth.rs/run/opstack/](https://reth.rs/run/opstack/)  
29. How to Discover long-tail MEV Strategies using Revm \- Paweł Urbanek, accessed July 9, 2025, [https://pawelurbanek.com/long-tail-mev-revm](https://pawelurbanek.com/long-tail-mev-revm)  
30. REVM-Trace \- Lib.rs, accessed July 9, 2025, [https://lib.rs/crates/revm-trace](https://lib.rs/crates/revm-trace)  
31. revm\_trace \- Rust \- Docs.rs, accessed July 9, 2025, [https://docs.rs/revm-trace](https://docs.rs/revm-trace)  
32. Unveiling Ethereum's P2P Network: The Role of Chain and Client Diversity \- arXiv, accessed July 9, 2025, [https://arxiv.org/html/2501.16236v1](https://arxiv.org/html/2501.16236v1)  
33. Unveiling Ethereum's P2P Network: The Role of Chain and Client Diversity \- ResearchGate, accessed July 9, 2025, [https://www.researchgate.net/publication/388422900\_Unveiling\_Ethereum's\_P2P\_Network\_The\_Role\_of\_Chain\_and\_Client\_Diversity](https://www.researchgate.net/publication/388422900_Unveiling_Ethereum's_P2P_Network_The_Role_of_Chain_and_Client_Diversity)  
34. reth p2p, accessed July 9, 2025, [https://reth.rs/cli/reth/p2p](https://reth.rs/cli/reth/p2p)  
35. Faster block/blob propagation in Ethereum \- Networking, accessed July 9, 2025, [https://ethresear.ch/t/faster-block-blob-propagation-in-ethereum/21370](https://ethresear.ch/t/faster-block-blob-propagation-in-ethereum/21370)  
36. reth\_rpc \- Rust, accessed July 9, 2025, [https://reth.rs/docs/reth\_rpc/index.html](https://reth.rs/docs/reth_rpc/index.html)  
37. Ports \- Reth, accessed July 9, 2025, [https://reth.rs/run/ports.html](https://reth.rs/run/ports.html)  
38. quic-rpc \- crates.io: Rust Package Registry, accessed July 9, 2025, [https://crates.io/crates/quic-rpc](https://crates.io/crates/quic-rpc)  
39. Support arbitrary RPC modules in the CLI · Issue \#17222 · paradigmxyz/reth \- GitHub, accessed July 9, 2025, [https://github.com/paradigmxyz/reth/issues/17222](https://github.com/paradigmxyz/reth/issues/17222)  
40. Reth, accessed July 9, 2025, [https://reth.rs](https://reth.rs)  
41. A Comprehensive guide to HTTP/3 and QUIC \- Sergey Drozdov, accessed July 9, 2025, [https://sd.blackball.lv/articles/read/19126-a-comprehensive-guide-to-http3-and-quic](https://sd.blackball.lv/articles/read/19126-a-comprehensive-guide-to-http3-and-quic)  
42. Introducing QUIC and HTTP/3 \- DevCentral \- F5, accessed July 9, 2025, [https://community.f5.com/kb/technicalarticles/introducing-quic-and-http3/290674](https://community.f5.com/kb/technicalarticles/introducing-quic-and-http3/290674)  
43. A gentle introduction to XDP \- Datadog, accessed July 9, 2025, [https://www.datadoghq.com/blog/xdp-intro/](https://www.datadoghq.com/blog/xdp-intro/)  
44. bpf-developer-tutorial/src/21-xdp/README.md at main \- GitHub, accessed July 9, 2025, [https://github.com/eunomia-bpf/bpf-developer-tutorial/blob/main/src/21-xdp/README.md](https://github.com/eunomia-bpf/bpf-developer-tutorial/blob/main/src/21-xdp/README.md)  
45. eBPF XDP: The Basics and a Quick Tutorial | Tigera \- Creator of Calico, accessed July 9, 2025, [https://www.tigera.io/learn/guides/ebpf/ebpf-xdp/](https://www.tigera.io/learn/guides/ebpf/ebpf-xdp/)  
46. Reth's path to 1 gigagas per second, and beyond \- Gate.com, accessed July 9, 2025, [https://www.gate.com/learn/articles/reths-path-to-1-gigagas-per-second-and-beyond/2802](https://www.gate.com/learn/articles/reths-path-to-1-gigagas-per-second-and-beyond/2802)  
47. risechain/pevm: Blazingly fast Parallel EVM \- GitHub, accessed July 9, 2025, [https://github.com/risechain/pevm](https://github.com/risechain/pevm)  
48. Configuring Reth, accessed July 9, 2025, [https://reth.rs/run/config.html](https://reth.rs/run/config.html)  
49. Why Reth?, accessed July 9, 2025, [https://reth.rs/introduction/why-reth/](https://reth.rs/introduction/why-reth/)  
50. Reth — the most powerful dedicated node client \- Chainstack, accessed July 9, 2025, [https://chainstack.com/reth/](https://chainstack.com/reth/)  
51. Ress: Scaling Ethereum with Stateless Reth Nodes \- Paradigm.xyz, accessed July 9, 2025, [https://www.paradigm.xyz/2025/03/stateless-reth-nodes](https://www.paradigm.xyz/2025/03/stateless-reth-nodes)  
52. paradigmxyz/ress: The implementation of Stateless Ethereum client based on Reth \- GitHub, accessed July 9, 2025, [https://github.com/paradigmxyz/ress](https://github.com/paradigmxyz/ress)  
53. reth-indexer reads directly from the reth db and indexes the data into traditional and alternative databases / datastores (postgres, GCP bigquery, etc) all decoded with a simple config file and no extra setup alongside exposing a API ready to query the data. \- GitHub, accessed July 9, 2025, [https://github.com/joshstevens19/reth-indexer](https://github.com/joshstevens19/reth-indexer)  
54. Paradigm releases 'Ethereum for Rust' to help ensure network stability \- Cointelegraph, accessed July 9, 2025, [https://cointelegraph.com/news/paradigm-releases-ethereum-for-rust-to-help-ensure-network-stability](https://cointelegraph.com/news/paradigm-releases-ethereum-for-rust-to-help-ensure-network-stability)