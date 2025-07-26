

# **An In-Depth Technical Analysis of the Linera Protocol: Architecture, Real-Time Capabilities, and Ecosystem Trajectory**

## **Executive Summary**

Linera is a Layer-1 blockchain protocol engineered to address the fundamental limitations of scalability and latency that constrain existing Web3 infrastructures. Founded by a former Meta researcher with deep expertise in distributed systems and cryptographic protocols, Linera introduces a novel architectural paradigm centered on **microchains**. This model deviates from traditional monolithic and sharded blockchains by providing each user with their own lightweight, parallel chain, all operating under the shared security of a unified, elastic validator set. The protocol's stated purpose is to deliver Web2-level performance and responsiveness, enabling a new class of real-time, hyper-interactive decentralized applications.

The core of Linera's innovation lies in its ability to achieve horizontal scalability by adding more microchains, theoretically offering unlimited transaction throughput without the typical trade-offs in security or decentralization seen in other models. Its claim to be a "real-time" blockchain is substantiated by a multi-faceted technical approach that combines sub-second transaction finality with a client-side infrastructure designed for perceived speed, including push notifications and instant pre-confirmation of transactions.

The protocol's technology stack is built on a WebAssembly (Wasm) virtual machine, with an initial SDK targeting Rust developers, and a clear roadmap for future EVM compatibility to attract the broader Solidity community. For its data layer, Linera has forged a strategic partnership with Walrus, a decentralized storage protocol, adopting a modular approach to manage on-chain state and large data blobs. Similarly, integrations with projects like Atoma Network for verifiable AI compute and DeCharge for DePIN data processing highlight a go-to-market strategy focused on high-value, infrastructure-level partnerships.

Currently in a multi-phase testnet, Linera's ecosystem is nascent, with most dApps being early-stage experiments emerging from developer programs. The project is backed by $12 million in seed funding from top-tier venture capital firms, including a16z crypto and Borderless Capital. Its detailed public roadmap outlines a methodical progression towards a mainnet launch, prioritizing core protocol stability before introducing broader compatibility and user-facing features. This report provides an exhaustive technical analysis of Linera's architecture, its unique mechanics, API structure, ecosystem trajectory, and competitive positioning, offering a definitive assessment of its potential to carve a significant niche in the high-performance blockchain landscape.

## **Section 1: Foundational Principles and Strategic Positioning**

This section establishes the "why" behind Linera, grounding its technical architecture in the founder's academic background and a clear critique of existing blockchain scaling paradigms.

### **1.1 Introduction: Defining Linera in the L1 Landscape**

Linera is a Layer-1 (L1) blockchain infrastructure designed from first principles to provide predictable performance, security, and low-latency responsiveness for demanding Web3 applications.1 It is specifically optimized for use cases that involve a large number of active users engaging in time-sensitive transactions in parallel.3 The project officially markets itself as "The Real-Time Blockchain," a claim predicated on its architectural ability to achieve extremely low latency by organizing transactions into small, parallel chains that are synchronized in real time across both network validators and user wallets.4

Founded in 2021, the project has garnered significant institutional interest, securing a total of $12 million across two seed funding rounds. The first round in June 2022 raised $6 million and was led by Andreessen Horowitz (a16z), and a subsequent $6 million round in August 2023 was led by Borderless Capital.5 This backing from prominent venture capital firms underscores the market's interest in novel approaches to blockchain scalability.

### **1.2 The Founder's Pedigree: From Meta's Novi to Linera**

The credibility and technical direction of the Linera project are deeply rooted in the background of its founder, Mathieu Baudet. Baudet is a former infrastructure engineer and researcher at Meta, where he contributed to the Novi digital wallet and the Libra (later Diem) blockchain project.5 He holds a PhD in Computer Science with a specialization in cryptographic protocols, Byzantine Fault Tolerant (BFT) consensus, and formal verification, lending substantial academic and practical weight to the project's design.9

Critically, the Linera protocol is not merely an abstract idea but a direct technological evolution of academic research Baudet co-authored during his tenure at Meta's Novi Research. The protocol's architecture and communication model are a generalization of an academic protocol named **FastPay**, which was published in 2020 at the prestigious ACM Advances in Financial Technologies conference (AFT'20).5 This provides a clear and verifiable intellectual lineage from a peer-reviewed academic paper focused on low-latency, account-based payments to the full-scale implementation of the Linera blockchain. This foundation suggests a design philosophy that is principled and research-driven, as opposed to one that is purely market-driven or iterative. To fully comprehend Linera's architecture, one must understand the core arguments of the FastPay protocol, from which it inherits its foundational DNA.

### **1.3 The Problem Statement: A Critique of Existing Scaling Solutions**

The Linera whitepaper explicitly frames its purpose as solving the problem of **blockspace scarcity**.1 In traditional monolithic blockchains, the finite space within blocks and the fixed rate of block production create a bottleneck during periods of high demand. This competition for limited resources inevitably leads to network congestion, volatile and high transaction fees, and unpredictable delays, rendering such architectures unsuitable for applications that require guaranteed real-time performance.8

The project's foundational documents present a systematic critique of the primary alternative scaling solutions, arguing that each possesses inherent limitations that Linera is designed to overcome 12:

* **Single Chain Optimization:** While techniques like parallel execution of independent transactions within a single block can increase raw throughput, these systems are still fundamentally constrained by the data propagation delay between validators and the need for sequential ordering of the global state. They cannot offer guaranteed latency or predictable fees for any single user, as performance remains dependent on the aggregate activity of all other users.  
* **Blockchain Sharding:** This approach, which divides a blockchain's state into a fixed number of parallel chains (shards), introduces significant challenges. It often creates security trade-offs, as an attacker can focus resources on compromising the weakest or least-secured shard. Furthermore, rebalancing user accounts across shards is a complex and communication-intensive operation. Most critically, as the number of shards increases, the volume of cross-shard messages grows, and the high latency of this communication can ultimately negate the performance benefits of adding more chains.  
* **Layer-2 Rollups:** Rollups solve scalability by executing transactions off-chain and posting compressed data to a Layer-1 for security. However, this model introduces significant latency in achieving finality. Optimistic rollups require a challenge period that can last for days to resolve potential disputes. Validity rollups (ZK-rollups), while faster, must batch a large number of L2 transactions to amortize the high cost of generating and verifying a validity proof on the L1. This settlement delay makes both types of rollups ill-suited for applications that demand sub-second user-to-user interaction and finality.

This pointed critique reveals a deliberate strategic positioning. Linera is not aiming to be a universal replacement for Ethereum or its L2 ecosystem. Instead, it is targeting a specific, and arguably underserved, market niche: applications such as real-time on-chain gaming, high-frequency DeFi auctions, and responsive oracles, where the multi-minute or multi-day finality of rollups is an architectural non-starter.4 This positions Linera as a potential specialized complement to the broader blockchain landscape, rather than a direct, all-purpose competitor.

## **Section 2: The Microchain Architecture: A Deep Dive into Linera's Core Innovation**

This section deconstructs the central concept of microchains, explaining how they function and enable Linera's unique approach to scalability.

### **2.1 The Concept of Microchains: Parallel Chains, Shared Security**

The foundational innovation of the Linera protocol is the **microchain**.4 The core concept is to operate a virtually unlimited number of parallel, lightweight blockchains within a single, unified set of validators.6 This architecture represents a fundamental departure from traditional multi-chain or sharded protocols, where each constituent chain typically requires its own distinct validator set, making the creation of new chains and inter-chain communication prohibitively expensive and slow.16

In the Linera model, all microchains are equally secured by the collective stake of the entire validator pool. A microchain itself is a standard, cryptographically-linked chain of blocks. The key distinction lies in the separation of roles: the act of proposing a new block is decoupled from the act of validating it. For many microchains, block production is delegated directly to the users themselves, allowing an immense number of chains to operate in parallel without congesting the network.15

### **2.2 Types of Microchains and Their Roles**

The Linera protocol defines three distinct types of microchains, each engineered for a specific function within the ecosystem, providing developers with a flexible toolkit for building scalable applications 15:

* **User Chains (Single-user Chains):** This is the protocol's most significant and novel feature. By default, every user wallet is provided with its own dedicated microchain. This chain functions as a form of personal, on-chain blockspace where the user has exclusive rights to propose new blocks. Users interact with the Linera network primarily by creating and signing blocks on their own chain to manage their assets, update their personal state, and initiate communications with other chains or applications.16 This user-centric model fundamentally redefines the relationship between an individual and the blockchain, transforming the user from a passive requester on a shared global ledger to an active owner and manager of their own personal ledger. This shift has profound implications for user agency, data sovereignty, and the design of applications that require users to predictably manage their own state history.  
* **Multi-user Chains (Permissioned Chains):** These are ephemeral or semi-permanent microchains shared among a small, explicitly defined group of users. They are designed to manage stateful interactions that require coordination between multiple parties. Canonical use cases include temporary environments for executing atomic swaps, running private on-chain game instances, or managing multi-party auctions.15  
* **Public Chains:** These microchains operate in a manner more akin to traditional blockchains. Block production is managed by the validators themselves, who typically use a leader election protocol to successively propose and finalize new blocks. Public chains are generally intended to host system-level applications or dApps that require a shared, canonical state accessible to all network participants.15

### **2.3 Horizontal Scalability and Elastic Validators**

Linera's approach to scalability is fundamentally horizontal. The system's capacity increases not by making the blocks of a single chain larger or faster, but by adding more parallel microchains to accommodate more users and applications.8 Because the protocol places no theoretical limit on the number of microchains that can coexist, it claims no theoretical ceiling on its aggregate transactions per second (TPS).4

This model is made viable by the concept of **elastic validators**. Linera validators are designed to function like modern, scalable web services rather than monolithic nodes.1 They are internally sharded and can elastically adjust their computational and network capacity by adding or removing internal processing units, or "workers," in response to network demand.16 This design effectively offloads the burden of scalability from the application developer to the validator operators. These operators are economically incentivized to maintain high performance and can leverage existing Web2 cloud infrastructure (e.g., AWS, Google Cloud) to dynamically scale their services up or down.11

This economic and operational model is analogous to Web2 cloud services and presents a critical trade-off. On one hand, it is likely to attract large, professionalized node operators (such as Everstake, which is participating in the testnet 11) who possess deep expertise in DevOps and scalable cloud architecture. On the other hand, the high operational requirements of running an "elastically scalable" service could create a significant barrier to entry for smaller, community-run validators. This dynamic could potentially lead to a degree of centralization around a few highly competent, large-scale operators, a factor that must be considered in the protocol's long-term decentralization goals.

The following table provides a structured comparison of Linera's architectural approach against other major blockchain scaling paradigms.

| Feature | Monolithic L1 (e.g., Ethereum) | Sharded L1 (e.g., ETH 2.0) | L2 Rollup | Linera (Microchains) |
| :---- | :---- | :---- | :---- | :---- |
| **Scalability Method** | Vertical (bigger blocks) | State Sharding | Off-chain Execution & On-chain Settlement | **Horizontal (more chains)** |
| **State Execution** | Sequential | Parallel (per shard) | Sequential (on L2) | **Parallel (per microchain)** |
| **Security Model** | Global Validator Set | Shard-Specific Validators | L1 Security Guarantees | **Global Validator Set (Shared)** |
| **Inter-Chain Communication** | N/A | High Cost/Latency | High Latency (L1 bridge) | **Low Cost/Latency (internal network)** |
| **User Latency Guarantee** | No (competes for blockspace) | No (competes for shard blockspace) | No (L1 settlement delay) | **Yes (for user's own chain)** |

Table 1: Architectural Comparison of Scaling Solutions, based on data from.12

## **Section 3: Protocol Mechanics and Real-Time Performance**

This section examines the technical underpinnings of Linera's performance claims, focusing on consensus, finality, and the features that enable "real-time" interactions.

### **3.1 Consensus and Finality**

Linera's security and agreement mechanism is built upon a **Delegated Proof-of-Stake (DPoS)** model.1 In this system, token holders will delegate their stake to a set of active validators who are responsible for running the network infrastructure and finalizing blocks. The protocol will be secured through a combination of economic incentives for honest behavior and penalties for malpractice, with plans for a robust, community-driven auditing protocol in the future.2

A cornerstone of Linera's performance is its rapid finality. The protocol is engineered to achieve a **finality time of under 0.5 seconds** for the majority of blocks.16 This metric is critical, as it represents the point at which a transaction is irreversibly confirmed on the blockchain, and it directly supports the protocol's viability for real-time applications.

Furthermore, for every block that is finalized, Linera generates a **certificate of execution**.16 This is a cryptographic proof, signed by a quorum of validators, that attests to the block's validity and the result of its execution. These certificates are a key feature for interoperability, as they can be efficiently verified by external systems, including other blockchains and light clients, without needing to trustlessly replay the entire chain's history.4

### **3.2 Deconstructing "Real-Time" Capabilities**

The "real-time" designation used by Linera is not merely a marketing term but a descriptor for a suite of specific, architecturally-integrated features designed to optimize for end-to-end, user-perceived latency. This holistic view of performance distinguishes Linera from protocols that focus solely on maximizing raw on-chain throughput (TPS). The key components include:

* **Ultra-Low Latency:** The architecture is explicitly designed to minimize user-to-user latency. The goal is for the total time elapsed—from one user creating a block on their microchain to a second user being notified, fetching the new data, and updating their client-side UI—to be consistently **below one second**.13  
* **Real-Time Push Notifications:** The protocol includes native infrastructure for validators to send **real-time push notifications** directly to web clients.16 This is a crucial feature that eliminates the need for applications to engage in inefficient, periodic polling of the blockchain to check for state updates, enabling a truly reactive user experience.  
* **Instant Pre-confirmation:** Linera wallets are designed to be "next-generation smart wallets" that act as active nodes in the protocol.4 They can synchronize the state of relevant microchains and execute the Wasm VM locally. This allows for the  
  **instant pre-confirmation** of a user's own transactions within their wallet's interface, providing immediate feedback long before the transaction is finalized by the validator network. This creates the feel of a Web2 application, where user actions appear to be confirmed instantly.4

### **3.3 Cross-Chain Communication and Composability**

Linera's multi-chain design necessitates a clear and efficient communication model. The protocol makes a sharp distinction between interactions within a single microchain and interactions between different microchains, creating a specific programming paradigm that developers must adopt 1:

* **Asynchronous Inter-Chain Communication:** Communication *across* different microchains is handled via **asynchronous messages**.1 When an application on one chain needs to interact with another, it dispatches a message. This message is routed through the validators' efficient internal network as a remote procedure call (RPC) and placed into the designated inbox of the target chain. The owner of the target chain can then choose to process this message in a subsequent block.17 This asynchronous model is essential for maintaining the parallel nature of the system, as it prevents one chain from having to wait on another.  
* **Synchronous Intra-Chain Composability:** In stark contrast, all interactions *within* the confines of a single microchain support **full synchronous composability**.16 Applications on the same microchain can call each other directly and atomically, similar to how smart contracts interact on a traditional, single-threaded blockchain like Ethereum. This allows for complex, multi-step operations to be executed with guaranteed atomicity on a user's own chain.

This dual communication model encourages a specific and highly scalable development pattern: developers are incentivized to place the majority of an application's logic and state on individual user chains to leverage powerful synchronous operations, while using the more complex asynchronous messaging model sparingly for coordination between users.15 The success of this model will depend significantly on the quality of the Linera SDK and developer education in promoting this new paradigm.

## **Section 4: The Linera Technology Stack**

This section details the key components of Linera's underlying technology, from data storage to the execution environment.

### **4.1 Data Availability and Storage: The Walrus Integration**

Addressing the long-term challenge of blockchain state bloat is a critical design consideration for any scalable L1. Rather than building a monolithic storage solution, Linera has adopted a modular strategy through a strategic partnership with **Walrus**, a decentralized storage protocol.19 This integration is designed to be symbiotic and handle different aspects of data management:

1. **Linera uses Walrus as a Storage Layer:** The Linera protocol will utilize Walrus as a primary storage layer for its own infrastructure needs. This is particularly important for handling large, unstructured data files, or "blobs," such as compiled application bytecode and extensive user data.18 The official roadmap for Testnet 3 explicitly includes "Block indexing and Walrus archives," indicating a deep integration for historical data management.18 This allows the core Linera protocol to remain lean and focused on high-speed transaction execution, while offloading the burden of large-scale, persistent storage to a specialized layer.  
2. **Walrus uses Linera for Verification:** Conversely, the Walrus protocol will leverage Linera as a high-speed verification infrastructure. When large data blobs are stored and managed on Walrus, Linera can be used to rapidly verify their integrity and process associated metadata, enhancing both the speed and security of the storage network.19

This partnership reflects a sophisticated, modular design philosophy. It recognizes that data execution and data availability are distinct problems that can be solved more effectively by specialized, interoperating layers. This pragmatic approach allows Linera to avoid the performance degradation associated with on-chain state bloat while still providing developers with a robust solution for data-intensive applications.

### **4.2 Execution Environment: Wasm First, EVM Next**

The choice of execution environment is a pivotal decision that balances performance against developer accessibility. Linera has adopted a phased strategy that prioritizes performance in its initial stages while building a bridge to the largest existing developer ecosystem for long-term growth.

* **Primary Environment (Wasm):** The initial and primary execution environment for Linera smart contracts is a **WebAssembly (Wasm) virtual machine**.1 Wasm is a modern, high-performance binary instruction format that offers significant speed advantages and language flexibility over legacy virtual machines. In line with this, the first official Software Development Kit (SDK) provided by Linera is for the  
  **Rust** programming language, which compiles efficiently to Wasm and is favored for its safety and performance in systems programming.2  
* **Future Compatibility (EVM):** Recognizing the immense network effects of the Ethereum ecosystem, Linera has placed **Ethereum Virtual Machine (EVM) compatibility** firmly on its development roadmap.6 This will eventually allow developers to write smart contracts in  
  **Solidity**, the most widely used smart contract language, and deploy them on Linera. According to the public roadmap, experimental EVM support is slated for Testnet 3, with the goal of achieving stable support in Testnet 4\.18

This "Wasm first, EVM later" strategy is a calculated trade-off. It allows the core protocol to be built and optimized around a modern, performant VM without the constraints of the EVM. Once the novel microchain architecture is proven to be stable and efficient, the project will then focus on adding EVM support to lower the barrier to entry for the vast pool of existing Web3 developers and facilitate the migration of liquidity and applications.

### **4.3 The Linera SDK and Developer Tooling**

The growth of any blockchain platform is contingent on its ability to attract and empower developers. Linera is actively fostering a developer community through the release of its SDK and a series of engagement initiatives. The initial Linera SDK is available on GitHub and provides the necessary tools and libraries for Rust developers to begin building applications for the Wasm VM.3

To support this, the project has launched several key programs:

* **Linera Developer School:** A multi-week educational program designed to teach developers Linera's unique multi-chain programming paradigm.5  
* **Developer Workshops and Hackathons:** The team hosts weekly live developer workshops and sponsors challenges, such as the "Games on Microchains Challenge," which offer financial prizes and community recognition to incentivize building on the testnet.3  
* **Comprehensive Documentation:** The official developer portal (linera.dev) serves as the central hub for technical documentation, split into three main parts: a high-level protocol overview, a detailed guide for application developers using the Rust SDK, and instructions for node operators who wish to run Linera validators.23

## **Section 5: Developer and Client-Side Interfaces: API Analysis**

This section clarifies Linera's API structure, addressing the user's specific questions about RPC/REST and real-time APIs, and correcting the prevalent confusion with the "Linea" project.

### **5.1 Clarification: Linera vs. Linea API Landscape**

A crucial point of clarification is the distinction between **Linera** and **Linea**. A significant portion of publicly available information, including several sources used for this report, conflates the two projects due to their similar names.24

**Linea is an EVM-equivalent ZK-rollup (Layer 2\) developed by ConsenSys.** It offers a standard Ethereum JSON-RPC API for interaction.27

**The Linea L2 and its API are entirely separate from and unrelated to the Linera L1 protocol.** All analysis of Linea's API is therefore irrelevant to understanding Linera's developer interfaces and has been excluded from this report.

### **5.2 The Primary Interface: A GraphQL-centric Approach**

Linera has made a strategic decision to forgo a traditional, sprawling JSON-RPC API as its primary developer interface. Instead, it has adopted **GraphQL** as the main query language for interacting with the system.30 This choice aligns with modern Web2 development practices and prioritizes developer experience and efficiency by allowing clients to request precisely the data they need in a single query, preventing the over-fetching common with REST or JSON-RPC APIs.

When the Linera client is run in service mode (linera service), it exposes a GraphQL server. This service also hosts a **GraphiQL IDE**, a powerful in-browser tool that enables developers to interactively explore the API's schema, write queries, and view results in real-time, significantly lowering the barrier to entry for understanding and debugging applications.30

The GraphQL API is structured to provide access to two distinct domains:

1. **System API:** This provides mutations and queries for performing system-level operations, such as managing chains and querying protocol-wide state.  
2. **Application API:** For every application deployed on a microchain, the node service exposes a unique GraphQL endpoint at the path /chains/\<chain-id\>/applications/\<application-id\>. This allows a frontend application to directly and securely query the state of a specific application instance on a specific chain, which is a powerful feature for building modular and multi-chain dApps.30

### **5.3 The Underlying linera-rpc Crate**

While Linera does not expose a public JSON-RPC API for dApp development, its internal architecture does rely on RPC. The project's source code on GitHub includes a Rust crate named linera-rpc.20 The purpose of this crate is to define the low-level data types and schemas for the RPC messages that facilitate all internal communication within the protocol. This includes interactions between clients and validator proxies, as well as the cross-chain messages sent between microchains within the validator network.20 This crate represents the internal messaging protocol, not a developer-facing API endpoint.

### **5.4 The "Real-Time API": A Composite Framework**

Linera does not offer a single, monolithic "real-time API." Instead, its real-time capabilities are an emergent property of its architecture, which developers can leverage through a combination of features:

* **Low-Latency Protocol:** The sub-second finality of the core protocol is the fundamental building block for any real-time interaction.16  
* **Push Notifications:** The native support for validators to push updates directly to web clients is a key component that enables reactive applications without constant polling.16  
* **Event Streams:** The most significant development for a formal real-time API is the planned introduction of **"event streams"** in Testnet 3\.18 This feature is slated to deprecate the existing, less formal pub/sub channels. The move to a formal event stream model indicates a maturation of the API toward a standard, robust streaming paradigm. This will provide developers with a structured way to subscribe to specific on-chain events (e.g., "a new bid was placed in auction X," "asset Y was transferred on chain Z") in a granular and efficient manner. This event stream functionality will be the concrete implementation of the real-time, streaming API that is essential for building the highly responsive applications Linera aims to support.

## **Section 6: Ecosystem Analysis: dApps and Strategic Integrations**

This section evaluates the current state of Linera's ecosystem, distinguishing between nascent, community-built dApps and more significant, strategic infrastructure partnerships that showcase the protocol's intended use cases.

### **6.1 Nascent dApp Ecosystem**

As a pre-mainnet protocol, the Linera dApp ecosystem is in its earliest stages of development. The projects that currently exist are primarily the output of community engagement initiatives like developer schools and hackathons, and should be viewed as experiments and proofs-of-concept rather than production-ready applications with significant user traction.

Publicly cited projects include:

* **ResPeer:** A peer-to-peer content publishing platform designed to run on Linera.32 The project won first place in the inaugural Linera Developer School, demonstrating a complex application with a content feed, a native credit system for incentives, and an asset marketplace.5 ResPeer was also recognized with a POAP for winning "Best AI App" at a subsequent hackathon.33 However, an examination of its primary GitHub repository reveals that it was archived in March 2024, indicating that active development by the original owner has ceased.  
* **CheCko:** Described in dApp directories as "Another browser wallet for Linera blockchain by ResPeer".32 This suggests it may be a component of the ResPeer project rather than a standalone wallet. There is minimal public information available, and its development status and user base are unknown and presumed to be negligible at this stage.  
* **Linera Meme:** Listed as a "fair-launch Meme token platform built on Linera".32 This appears to be a community or testnet project. There is a lack of concrete details, and searches for this token often lead to similarly named but unrelated meme tokens on other blockchains, such as Solana.34

The current state of these dApps indicates that Linera's ecosystem is still in a phase of experimentation. The true traction at this stage is not measured by the user count of these applications, but by the number of developers participating in the testnet and building with the SDK, which serves as a leading indicator of future ecosystem health.

### **6.2 Real-World Use Cases via Strategic Partnerships**

A more telling indicator of Linera's potential and strategic direction comes from its formal partnerships with other infrastructure projects. These integrations are not simple dApps but deep, protocol-level collaborations designed to leverage Linera's unique capabilities for high-value, business-to-business (B2B) use cases. This suggests a deliberate go-to-market strategy focused on proving the technology through foundational, infrastructure-level pillars.

Key strategic partnerships include:

* **DeCharge (DePIN):** DeCharge is a decentralized physical infrastructure network (DePIN) for electric vehicle (EV) charging, which is primarily built on Solana. DeCharge plans to integrate with Linera to handle its real-time data processing needs. It will use Linera's microchains to scalably store and process vast amounts of real-time data generated by its charging stations and mobile users, such as charging times, energy consumption, and electricity provenance. Linera's ability to process this data in real time and then post the results to other ecosystems like Solana is a critical feature for enabling user incentives and network optimization at a global scale.35  
* **Atoma Network (AI):** Atoma is a decentralized network for AI inference. It is partnering with Linera to bring private and verifiable AI compute to the Linera ecosystem. Linera's low-latency and high-throughput architecture is ideally suited to function as a settlement and request layer for AI compute, handling a high volume of inference requests without creating bottlenecks. The integration will be facilitated by Wasm-compatible SDKs, allowing Linera applications to seamlessly access Atoma's AI models.36  
* **Walrus (Storage):** As detailed previously, the partnership with Walrus provides a specialized, decentralized storage layer for the Linera ecosystem. This allows Linera to manage large data blobs efficiently, which is essential for storing application bytecode and supporting data-intensive applications.19

These B2B partnerships allow Linera to demonstrate its value proposition on real-world, high-demand use cases, which can then serve as a powerful proof point to attract a wider range of developers and applications to its ecosystem.

### **6.3 Community and Developer Engagement**

Despite its early stage, Linera has cultivated a substantial online presence, reporting over 350,000 followers on X (formerly Twitter) and 320,000 members on Discord.22 The project team actively engages this community through a variety of programs designed to stimulate participation and encourage development on the platform:

* **Drops Campaign:** A points-based program launched with Testnet 2 ("Babbage") that rewards users for social media activities like tweeting about Linera or engaging with official posts. The project has been explicit that these points are for engagement purposes only, have no monetary value, and do not represent a promise of a future token airdrop.22  
* **Developer Challenges:** The team regularly hosts hackathons and challenges, such as the "Games on Microchains Challenge," which provide XP, community recognition, and potential grants to developers who build on the testnet.3  
* **Community Events:** Linera hosts a "Weekly Community Hour" on Discord to discuss project developments and interact with the community. Participation is often incentivized through the issuance of POAPs (Proof of Attendance Protocol tokens), which serve as digital collectibles commemorating events and milestones.22

## **Section 7: Project Trajectory: Roadmap, Funding, and Team**

This section outlines the future development path of Linera, supported by its financial backing and the composition of its team.

### **7.1 Multi-Phase Development Roadmap**

Linera has laid out a clear, methodical, and multi-phase public roadmap that details the progression from its initial devnet to a full mainnet launch. This roadmap demonstrates an infrastructure-first development process, where core protocol features are built and stabilized before more user-facing components and broad compatibility layers are introduced. This disciplined engineering approach prioritizes the long-term stability and security of the novel microchain architecture.

The following table consolidates the key features and goals for each phase of the roadmap, based on information from the project's official documentation.

| Phase | Key Goals | SDK Features | Core Protocol Features | Infrastructure Features |
| :---- | :---- | :---- | :---- | :---- |
| **Testnet 1 "Archimedes"***(Launched Nov 2024\)* | Foundational Features & Decentralization | Rust SDK v0.13+, Web demos, Blob storage for user data | Multi-user chains, Blob storage for bytecode, Initial fee support | Onboarding of 20+ external validators, Fixed number of workers |
| **Testnet 2 "Babbage"***(Launched Apr 2025\)* | Web Client & Oracles | Official Web client framework, Native oracles (HTTP queries), POW public chains | Scalable reconfigurations, Bridge-friendly EVM-compatible headers | Improved hotfix process, Offline resizing of workers |
| **Testnet 3***(Future)* | User Experience & EVM | Browser extension & WalletConnect, **Event streams**, Experimental EVM support | Scalable client (partial execution), Execution cache, Protocol upgradability | High-TPS configuration, Block explorer, Block indexing & Walrus archives |
| **Testnet 4***(Future)* | Governance & Security | Stable EVM support, Transaction scripts, Application upgradability | **Governance chain**, Final tokenomics & fees, Storage durability | Network performance measurement, Validator incentives, **Security audits** |
| **Mainnet & Beyond***(Future)* | Full Deployment & Elasticity | Account abstraction, Light clients for other languages (e.g., Sui Move) | Permissionless auditing protocol, Performance improvements | Dynamic shard assignment & elasticity, Native bridges |

Table 2: Linera's Detailed Development Roadmap, based on data from.6

### **7.2 Financial Backing and Governance Model**

The Linera project is well-capitalized to pursue its ambitious roadmap, having secured **$12 million** in seed funding from a consortium of top-tier venture capital firms. The lead investors, **a16z crypto** and **Borderless Capital**, are joined by other notable names such as Tribe Capital, GSR, Flow Traders, and DFG.5 This strong financial backing provides a significant runway for research and development.

The long-term governance of the protocol is planned to be decentralized. The underlying consensus mechanism is **Delegated Proof-of-Stake (DPoS)**.1 The formal introduction of an on-chain

**governance chain** and the finalization of the protocol's **tokenomics** and fee structure are key milestones scheduled for Testnet 4\.18 At present, Linera does not have a native token, and all testnet activities and points campaigns are explicitly non-monetary and not a guarantee of future token rewards.22

### **7.3 Team Analysis**

Linera is led by its founder, Mathieu Baudet, whose extensive technical background in distributed systems at Meta provides the project with a strong, credible vision.5 However, a point of potential concern noted in industry analyses is the relatively small size of the Linera team. As of early 2023, the team consisted of approximately 8 members, a stark contrast to the much larger teams of its primary competitors, Aptos (64 members) and Sui (93 members) at similar stages.39

While a smaller team can be more agile and maintain a highly focused vision during the intensive research and core development phases, this size could become a constraint as the project moves toward mainnet. The demands of broad ecosystem support, business development, developer relations, and marketing may require significant team expansion. The project's ability to scale its organization effectively will be a critical factor in its long-term success. The current small size can be seen as a feature for the deep research phase but a potential bug for the future growth phase.

## **Section 8: Competitive Landscape and Market Positioning**

This section provides a comparative analysis of Linera against its most direct competitors, highlighting the key architectural and strategic differentiators.

### **8.1 Linera vs. Other High-Performance, Parallel Execution L1s**

Linera enters a competitive landscape of L1 blockchains that aim to solve the scalability trilemma, particularly those that leverage parallel execution and have roots in the ex-Meta/Diem ecosystem.40 Its primary competitors are:

* **Solana:** The pioneer in bringing parallel execution to a mainstream audience. Solana uses a combination of **Proof-of-Stake (PoS)** and a unique **Proof-of-History (PoH)** timing mechanism to order transactions, along with a parallel processing engine called **Sealevel**. It boasts a large, mature ecosystem with high Total Value Locked (TVL), but its history has been marked by several network outages that have raised questions about its reliability.42  
* **Aptos:** A direct descendant of the Diem project, Aptos uses the **Move** programming language and a parallel execution engine named **Block-STM**. This engine optimistically executes transactions in parallel and re-executes them only when conflicts are detected. Architecturally, it follows a more traditional linear blockchain model. It has a larger team and a more developed ecosystem than Linera.39  
* **Sui:** Also founded by ex-Meta developers and using the **Move** language, Sui takes a different architectural path. It is built on a **Directed Acyclic Graph (DAG)** and employs an object-centric data model. This allows independent transactions (e.g., a simple asset transfer) to be finalized by validators without needing to be ordered in a global sequence, resulting in extremely low latency for simple operations.45

### **8.2 Key Architectural Differentiators**

While all these protocols aim for high performance, Linera's architecture is fundamentally distinct. The most critical technical distinction is that Linera's architecture is designed to **isolate user activity**, whereas its competitors' architectures are designed to **optimize the processing of co-mingled user activity**. An intensive operation on one Linera user's chain does not impact the performance or fees on another's, making the protocol architecturally immune to the "noisy neighbor" problem that can plague shared-state blockchains during events like popular NFT mints.

Other key differentiators include:

* **User Agency:** Linera's model of user-owned chains and user-driven block production is unique. In Solana, Aptos, and Sui, users are passive submitters of transactions to a global mempool for processing by validators.  
* **Scalability Philosophy:** Linera achieves horizontal scalability by adding more chains. Competitors primarily focus on vertically scaling a single (albeit parallelized) state machine.  
* **Intellectual Lineage:** While often grouped with Aptos and Sui as "Diem-descendants," this is a simplification. Aptos is a direct successor to the Diem codebase. Sui is a major evolution of the Move language and Diem concepts. Linera, however, is based on a different academic protocol (FastPay) and does not use Move (initially opting for Rust/Wasm), representing a distinct intellectual branch from the same Meta research tree.

The following table summarizes the key technical and strategic differences between these high-performance L1s.

| Feature | Linera | Solana | Aptos | Sui |
| :---- | :---- | :---- | :---- | :---- |
| **Core Architecture** | Multi-chain (Microchains) | Single Chain (with Sealevel) | Single Chain (with Block-STM) | DAG (Checkpoints) |
| **Consensus** | DPoS | PoS \+ Proof of History (PoH) | AptosBFT (PoS) | Mysticeti (DAG-based PoS) |
| **Primary Language/VM** | Rust / Wasm (EVM planned) | Rust / Solana VM (Move VM available) | Move / Move VM | Move / Move VM |
| **Claimed Finality** | \< 0.5s | \~2.5s (optimistic) | \~0.9s | \~0.5s |
| **Key Differentiator** | User-owned chains & elastic validators | Proof of History for ordering | Optimistic parallel execution | Object-centric model & DAG consensus |
| **Ecosystem Maturity** | Nascent / Pre-Mainnet | Mature & High TVL | Launched & Growing | Launched & Growing |
| **Primary VC Backers** | a16z, Borderless Capital | Multicoin Capital, Jump Crypto | a16z, Jump Crypto, Binance Labs | a16z, FTX Ventures, Jump Crypto |

Table 3: Competitive Analysis: Linera vs. High-Performance L1s, based on data from.42

## **Section 9: Concluding Analysis and Expert Assessment**

This final section synthesizes the preceding analysis to provide a balanced, forward-looking assessment of the Linera protocol.

### **9.1 Synthesis of Findings: Strengths and Weaknesses**

The analysis reveals a project with a unique profile, characterized by significant strengths in its design philosophy and considerable challenges in its market position.

**Strengths:**

* **Radical, Research-Driven Innovation:** The microchain architecture is a genuinely novel approach to blockchain scalability, grounded in peer-reviewed academic research, which sets it apart from more iterative designs.  
* **Superior Architectural Isolation:** The protocol's design theoretically offers unparalleled performance isolation, predictable latency, and guaranteed blockspace for individual users, directly addressing the "noisy neighbor" problem.  
* **Holistic "Real-Time" Focus:** The protocol's design extends beyond on-chain metrics to include client-side UX enhancements like push notifications and pre-confirmation, demonstrating a sophisticated understanding of what makes an application feel responsive.  
* **Strong Founder Pedigree and Top-Tier Backing:** The project is led by a recognized expert in the field and is well-capitalized by premier venture firms, providing both technical credibility and a substantial financial runway.

**Weaknesses:**

* **Nascent and Unproven Ecosystem:** As a pre-mainnet protocol, Linera currently lacks production-ready dApps and has yet to demonstrate significant user or developer traction beyond its community programs.  
* **Smaller Team and Methodical Pace:** The team is significantly smaller than its main competitors, and its deliberate, infrastructure-first roadmap means it may reach market maturity later than its rivals.  
* **Developer Learning Curve:** The asynchronous, multi-chain programming model is a new paradigm that will require a significant educational effort to onboard developers accustomed to globally synchronous environments like the EVM.  
* **Potential DPoS Centralization Vectors:** The DPoS consensus model, combined with the high operational requirements for running elastic validators, could favor a small number of large, professionalized operators, posing a potential risk to long-term decentralization.

### **9.2 Opportunities and Threats**

Linera's unique architecture positions it for specific opportunities while also exposing it to considerable market threats.

**Opportunities:**

* **Dominating a Niche:** Linera has a clear opportunity to become the dominant platform for ultra-low-latency applications—such as on-chain gaming, real-time oracles, and high-frequency DeFi—that are fundamentally ill-suited for the high-latency finality of L2 rollups or the unpredictable performance of congested monolithic L1s.  
* **Becoming Critical B2B Infrastructure:** The protocol could thrive by serving as a high-performance infrastructure layer for other protocols, as demonstrated by its partnerships with DeCharge (DePIN) and Atoma (AI). This B2B focus could provide a sustainable path to adoption.  
* **Attracting Web2 Developers:** Its modern, GraphQL-centric API and focus on a seamless developer experience could attract a new wave of developers from the Web2 world who are less entrenched in the EVM ecosystem.

**Threats:**

* **First-Mover Disadvantage:** Established competitors like Solana, Aptos, and Sui are already building network effects and could capture the majority of the market for high-performance dApps before Linera achieves mainnet maturity.  
* **Execution Risk:** The ambitious, multi-phase roadmap and small team size create significant execution risk. Any major delays or security vulnerabilities could cause the project to lose critical momentum.  
* **Adoption Inertia:** The network effect of the EVM is immense. Convincing a critical mass of developers and users to adopt a new, non-EVM-first platform is a monumental challenge that many technologically superior projects have failed to overcome.

### **9.3 Final Assessment and Future Outlook**

Linera stands out as one of the most intellectually compelling and architecturally ambitious projects in the current Layer-1 landscape. Its core thesis—that true scalability can be achieved by empowering individual users with their own personal, parallel blockchains—represents a fundamental paradigm shift.

However, its success is far from guaranteed and hinges on the team's ability to navigate a challenging path to market. The project's future trajectory will likely be determined by three critical factors:

1. **Flawless Technical Execution:** The team must deliver on its ambitious roadmap, particularly the key features of Testnet 3 (browser wallet, event streams, experimental EVM), without significant delays or security compromises.  
2. **Effective Developer Onboarding:** Success will depend on creating an SDK, documentation, and educational resources that make the novel multi-chain programming model not just understandable, but attractive and productive for developers.  
3. **Securing Foundational Partnerships:** The B2B strategy is sound, but it requires landing two or three flagship, high-volume infrastructure partners who can publicly showcase the protocol's unique capabilities at scale upon mainnet launch, thereby validating the entire model.

In conclusion, Linera is a high-risk, high-reward venture betting on a new vision for blockchain architecture. It should not be viewed as a direct "Solana killer" or "Ethereum killer." It is a specialized protocol, purpose-built for a specific class of applications that do not exist today because the requisite infrastructure has been missing. If Linera succeeds, it will be by enabling this new generation of real-time, hyper-interactive decentralized applications. Its progress, particularly around the maturation of its SDK, the growth of its developer community, and the deepening of its strategic partnerships, warrants close observation from technical analysts and sophisticated investors.

#### **Works cited**

1. Linera Whitepaper v2 | PDF | Computing \- Scribd, accessed June 17, 2025, [https://www.scribd.com/document/681428900/Linera-Whitepaper-v2](https://www.scribd.com/document/681428900/Linera-Whitepaper-v2)  
2. Whitepaper — Linera | The Real-Time Blockchain, accessed June 17, 2025, [https://linera.io/whitepaper](https://linera.io/whitepaper)  
3. Developers — Linera | The Real-Time Blockchain, accessed June 17, 2025, [https://linera.io/developers](https://linera.io/developers)  
4. Linera | The Real-Time Blockchain, accessed June 17, 2025, [https://linera.io/](https://linera.io/)  
5. About — Linera | The Real-Time Blockchain, accessed June 17, 2025, [https://linera.io/about](https://linera.io/about)  
6. Potential Linera Airdrop » How to be eligible?, accessed June 17, 2025, [https://airdrops.io/linera/](https://airdrops.io/linera/)  
7. Linera Funding Rounds, Token Sale Review & Tokenomics Analysis | CryptoRank.io, accessed June 17, 2025, [https://cryptorank.io/ico/linera](https://cryptorank.io/ico/linera)  
8. Linera Blockchain Raises Additional $6M for its Microchain Design Evolving from Meta's Novi Research | Financial IT, accessed June 17, 2025, [https://financialit.net/news/fundraising-news/linera-blockchain-raises-additional-6m-its-microchain-design-evolving-metas](https://financialit.net/news/fundraising-news/linera-blockchain-raises-additional-6m-its-microchain-design-evolving-metas)  
9. Mathieu Baudet, writer | CoinMarketCap, accessed June 17, 2025, [https://coinmarketcap.com/academy/author/mathieu-baudet](https://coinmarketcap.com/academy/author/mathieu-baudet)  
10. Mathieu Baudet \- Founder at Linera \- Apple Podcasts, accessed June 17, 2025, [https://podcasts.apple.com/us/podcast/mathieu-baudet-founder-at-linera/id1559587482?i=1000698274641](https://podcasts.apple.com/us/podcast/mathieu-baudet-founder-at-linera/id1559587482?i=1000698274641)  
11. Linera: a Brand New Approach to the Old Scalability Problem \- Everstake, accessed June 17, 2025, [https://everstake.one/blog/linera-a-brand-new-approach-to-the-old-scalability-problem](https://everstake.one/blog/linera-a-brand-new-approach-to-the-old-scalability-problem)  
12. A quick look at the Linera whitepaper: Microchains, elastic validators, and multi-chain programming | 链捕手ChainCatcher on Binance Square, accessed June 17, 2025, [https://www.binance.com/en/square/post/127623](https://www.binance.com/en/square/post/127623)  
13. Linera Provides User Latency Under a Second \#crypto \#web3 \#blockchain \- YouTube, accessed June 17, 2025, [https://www.youtube.com/shorts/dA4ZIzNnbU8](https://www.youtube.com/shorts/dA4ZIzNnbU8)  
14. Real-Time Data Services on Linera, accessed June 17, 2025, [https://linera.io/use-cases/real-time-analytics-data-feeds-oracles](https://linera.io/use-cases/real-time-analytics-data-feeds-oracles)  
15. Microchains in Linera, accessed June 17, 2025, [https://linera.io/news/microchains-in-linera](https://linera.io/news/microchains-in-linera)  
16. Overview \- linera.dev, accessed June 17, 2025, [https://linera.dev/protocol/overview.html](https://linera.dev/protocol/overview.html)  
17. Linera \- Organizations | IQ.wiki, accessed June 17, 2025, [https://iq.wiki/wiki/linera](https://iq.wiki/wiki/linera)  
18. Roadmap \- linera.dev, accessed June 17, 2025, [https://linera.dev/protocol/roadmap.html](https://linera.dev/protocol/roadmap.html)  
19. L1 Blockchain Linera Selects Walrus as Its Storage Solution, accessed June 17, 2025, [https://www.walrus.xyz/blog/linera-microchain-layer-1-storage](https://www.walrus.xyz/blog/linera-microchain-layer-1-storage)  
20. linera-protocol/README.md at main \- GitHub, accessed June 17, 2025, [https://github.com/linera-io/linera-protocol/blob/main/README.md](https://github.com/linera-io/linera-protocol/blob/main/README.md)  
21. Introducing the Linera Developer School, accessed June 17, 2025, [https://linera.io/news/introducing-the-linera-developer-school](https://linera.io/news/introducing-the-linera-developer-school)  
22. Community — Linera | The Real-Time Blockchain, accessed June 17, 2025, [https://linera.io/community](https://linera.io/community)  
23. linera.dev, accessed June 17, 2025, [https://linera.dev/](https://linera.dev/)  
24. Linea: The home network for the world, accessed June 17, 2025, [https://linea.build/](https://linea.build/)  
25. Web3 RPC LINEA nodes API \- GetBlock.io, accessed June 17, 2025, [https://getblock.io/nodes/linea/](https://getblock.io/nodes/linea/)  
26. Full Linea RPC APIs Support in Web3j \- Linux Foundation Decentralized Trust, accessed June 17, 2025, [https://www.lfdecentralizedtrust.org/blog/full-linea-rpc-apis-support-in-web3j](https://www.lfdecentralizedtrust.org/blog/full-linea-rpc-apis-support-in-web3j)  
27. Use the Linea API, accessed June 17, 2025, [https://docs.linea.build/api/reference](https://docs.linea.build/api/reference)  
28. Linea RPC Endpoint & Chain Settings \- 1RPC, accessed June 17, 2025, [https://www.1rpc.io/ecosystem/linea](https://www.1rpc.io/ecosystem/linea)  
29. QuickNode Linea RPC Overview, accessed June 17, 2025, [https://www.quicknode.com/docs/linea](https://www.quicknode.com/docs/linea)  
30. Node Service \- linera.dev, accessed June 17, 2025, [https://linera.dev/developers/core\_concepts/node\_service.html](https://linera.dev/developers/core_concepts/node_service.html)  
31. Main repository for the Linera protocol \- GitHub, accessed June 17, 2025, [https://github.com/linera-io/linera-protocol](https://github.com/linera-io/linera-protocol)  
32. Linera Ecosystem: Explore 3 Projects & Dapps, accessed June 17, 2025, [https://thedapplist.com/chain/linera](https://thedapplist.com/chain/linera)  
33. Linera | Next Top Blockchain Startup Hackathon — ResPeer App 1st Place AI App, accessed June 17, 2025, [https://collections.poap.xyz/organizations/955/drops/177809](https://collections.poap.xyz/organizations/955/drops/177809)  
34. Liner (meme) Price Chart \- Buy and Sell on Phantom, accessed June 17, 2025, [https://phantom.com/tokens/solana/EhRMpY1FgwjJ7SUxKAnhAbUz6AYvGHPiosMXt35zpump](https://phantom.com/tokens/solana/EhRMpY1FgwjJ7SUxKAnhAbUz6AYvGHPiosMXt35zpump)  
35. Linera x Decharge, accessed June 17, 2025, [https://linera.io/news/linera-x-decharge](https://linera.io/news/linera-x-decharge)  
36. Linera x Atoma: Under the Hood, accessed June 17, 2025, [https://linera.io/news/linera-x-atoma-under-the-hood](https://linera.io/news/linera-x-atoma-under-the-hood)  
37. More on Drops: Earn Testnet Points \- Linera, accessed June 17, 2025, [https://linera.io/news/introducing-drops-points-program](https://linera.io/news/introducing-drops-points-program)  
38. Scaling Community Growth with POAPs \- Linera, accessed June 17, 2025, [https://linera.io/news/scaling-community-growth-with-poaps](https://linera.io/news/scaling-community-growth-with-poaps)  
39. MOVE Series 'Third Brother': A Comprehensive Analysis of the New Public Chain Linera., accessed June 17, 2025, [https://www.theblockbeats.info/en/news/34708](https://www.theblockbeats.info/en/news/34708)  
40. The New Wave of Layer 1 Smart Contract Platforms: Aptos, Sui and Linera, accessed June 17, 2025, [https://research.sideshift.ai/aptos-sui-and-linera/](https://research.sideshift.ai/aptos-sui-and-linera/)  
41. Understanding Parallel Execution in Blockchain \- Suipiens, accessed June 17, 2025, [https://suipiens.com/blog/understanding-parallel-execution-in-blockchain/](https://suipiens.com/blog/understanding-parallel-execution-in-blockchain/)  
42. All Chains DeFi TVL \- DefiLlama, accessed June 17, 2025, [https://defillama.com/chains](https://defillama.com/chains)  
43. Solana vs Base: which is the best blockchain? \- SwissBorg Academy, accessed June 17, 2025, [https://academy.swissborg.com/en/learn/solana-vs-base](https://academy.swissborg.com/en/learn/solana-vs-base)  
44. Is my understanding correct of Ethereum vs Solana in terms of security? \- Reddit, accessed June 17, 2025, [https://www.reddit.com/r/ethereum/comments/1humt5c/is\_my\_understanding\_correct\_of\_ethereum\_vs\_solana/](https://www.reddit.com/r/ethereum/comments/1humt5c/is_my_understanding_correct_of_ethereum_vs_solana/)  
45. Sui VS Aptos: Which Is The BETTER Layer 1? \- YouTube, accessed June 17, 2025, [https://m.youtube.com/watch?v=ZP-kQBotUrs\&pp=ygUGI2FwdG9h](https://m.youtube.com/watch?v=ZP-kQBotUrs&pp=ygUGI2FwdG9h)  
46. Aptos vs. Sui vs. Movement: Move Blockchains Compared | Dwflabs on Gate Post, accessed June 17, 2025, [https://www.gate.com/th/post/status/7580049](https://www.gate.com/th/post/status/7580049)  
47. Exploring the SUI Blockchain Network: A New Era of High-Performance Decentralization, accessed June 17, 2025, [https://www.antiersolutions.com/blogs/exploring-the-sui-blockchain-network-a-new-era-of-high-performance-decentralization/](https://www.antiersolutions.com/blogs/exploring-the-sui-blockchain-network-a-new-era-of-high-performance-decentralization/)  
48. All About Directed Acyclic Graphs \- The Sui Blog, accessed June 17, 2025, [https://blog.sui.io/all-about-directed-acyclic-graphs/](https://blog.sui.io/all-about-directed-acyclic-graphs/)  
49. Move-Based Blockchains: Aptos, Sui, and Movement Compared | Bitget News, accessed June 17, 2025, [https://www.bitget.com/news/detail/12560604265652](https://www.bitget.com/news/detail/12560604265652)