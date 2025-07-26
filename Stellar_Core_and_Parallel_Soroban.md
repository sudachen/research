

# **Architectural Evolution of the Stellar Network: From Core Consensus to Parallel Smart Contract Execution**

## **Part I: The Foundation \- Stellar Core and the Consensus Protocol**

### **Section 1: An Overview of the Stellar Stack Architecture**

The Stellar network is a decentralized, open-source protocol designed for fast and low-cost value transfers. Its architecture is a layered stack of components, each serving a distinct purpose to create a robust and accessible financial infrastructure.1 Understanding this stack is fundamental to comprehending the network's operational flow, from application-level requests down to the core consensus mechanism. The primary components of this stack are Stellar Core, the Horizon API, and the Stellar RPC service.1

#### **Primary Components of the Stellar Stack**

* **Stellar Core:** This is the foundational software that constitutes the backbone of the Stellar network. Each node on the network runs an instance of Stellar Core, which is responsible for two primary functions: maintaining a common, distributed ledger and participating in a consensus process to validate and apply transactions.1 This consensus is achieved every 3-7 seconds, at which point a new ledger is closed and its state is agreed upon by the network.1  
* **Horizon API:** Horizon serves as the primary client-facing interface for applications interacting with the Stellar network's classic functionalities. It is a RESTful HTTP API server that allows developers to programmatically submit transactions and query the network's historical data.1 Applications, wallets, and exchanges typically communicate with Horizon via an SDK, a web browser, or command-line tools like cURL.1 This design deliberately offloads the burden of serving public HTTP requests from Stellar Core, allowing Core to focus exclusively on consensus and ledger maintenance.5 While the Stellar Development Foundation (SDF) provides a public Horizon instance for development, production applications are strongly encouraged to run their own instances for reliability and control.1  
* **Stellar RPC:** With the introduction of advanced smart contract capabilities, the Stellar RPC was developed to serve as the dedicated interface for this new functionality. It is a JSON RPC server that enables users and applications to interact with Soroban smart contracts.1 An application sends a request to the RPC server, which translates it into a format understood by the blockchain nodes and forwards it for processing. The results are then returned to the application via the RPC server.1 Initially known as the Soroban RPC, it was officially rebranded to Stellar RPC with the release of Protocol 22, a change that became fully enforced in Protocol 23, signaling the deep integration of smart contracts into the main Stellar ecosystem.6

#### **Node Roles and Responsibilities**

Anyone can run a Stellar Core node, but these nodes can be configured to perform different roles, contributing to the network's health and functionality in distinct ways.1

* **Validators:** These are nodes configured to actively participate in the Stellar Consensus Protocol (SCP). They generate cryptographic signatures to vote on the validity and order of transaction sets, playing a direct role in securing the network and closing ledgers.2  
* **Watchers:** These nodes run Stellar Core and connect to the peer-to-peer network to maintain an up-to-date copy of the ledger. However, they do not participate in the consensus process. They essentially "watch" the network activity without voting on it.7  
* **Archivers:** An archiver is a full validator that performs the additional task of publishing a history archive. This archive is a record of the network's complete history, stored in a location like Amazon S3 or a local file system. Other nodes can access these archives to catch up if they fall out of sync or to retrieve historical ledger data.5

#### **Interaction Model and Data Flow**

The architectural design enforces a clear separation of concerns. The typical data flow begins with a user interacting with an application (e.g., a digital wallet). The application uses a Stellar SDK to construct a transaction. This transaction is then submitted to an API endpoint, either Horizon for classic operations or the Stellar RPC for smart contract interactions. The API server forwards the transaction to a trusted Stellar Core node, which then broadcasts it to its peers on the p2p network. The transaction enters a pool of pending transactions, from which validators select candidates for the next ledger. Through the SCP process, validators agree on a transaction set, apply it to the current ledger, and a new, globally consistent ledger is created.1 This layered approach ensures that the core consensus engine remains lean, secure, and shielded from direct public internet traffic, enhancing the overall stability and predictability of the network.

### **Section 2: A Deep Dive into Stellar Core Internals**

While the Stellar Stack describes the ecosystem's external architecture, the Stellar Core software itself is a complex system of interacting components. Its design choices reflect a philosophy prioritizing simplicity, determinism, and performance for its specific use case as a payment and asset issuance network.

#### **Core Software Components**

The Stellar Core application is a C++ program composed of several key logical subsystems that work in concert to achieve consensus and maintain the ledger.4

* **Overlay:** This is the peer-to-peer (p2p) networking layer. It manages TCP connections to other Stellar Core nodes and is responsible for "flooding" or broadcasting messages, such as transactions and SCP consensus messages, across the network.4  
* **Herder:** Functioning as the central coordinator, Herder orchestrates the activities of the other subsystems. It is the concrete implementation that connects the abstract SCP logic to the rest of the Stellar system. It translates SCP's abstract concepts of a "slot" and a "value" into the tangible Stellar concepts of a "ledger number" and a "transaction set," respectively.4  
* **SCP (Stellar Consensus Protocol):** This component is the direct implementation of the consensus algorithm. It is designed as an abstract module that could theoretically be used in other distributed systems, with Herder providing the specific context for the Stellar network.5  
* **Ledger Manager:** This subsystem is responsible for managing the ledger itself. Its primary task is to take the transaction set that has been finalized by the SCP process and apply it to the previous ledger state, thereby creating the new current ledger.4  
* **History:** The History module manages the archiving of ledger data to an external, persistent storage system (e.g., a cloud bucket). This archive is crucial for nodes that need to catch up to the network or for services that require access to the full historical record.4  
* **BucketList:** A unique and critical data structure that represents the state of all ledger entries. It is optimized for efficient cryptographic hashing and is central to both consensus and state synchronization.5

#### **Threading Model and Design Philosophy**

Stellar Core employs a specific threading model to ensure deterministic behavior. The core I/O operations and the consensus logic itself run on a single main thread using asynchronous I/O. This design choice avoids the complexities and potential non-determinism of multi-threaded consensus logic. Computationally intensive but parallelizable tasks, such as cryptographic hashing, serialization, and memory copying (memcpy), are offloaded to a pool of worker threads.5 This approach maintains the simplicity and predictability of the state machine's core transition logic while leveraging modern multi-core processors for heavy lifting. The software is also designed to be self-contained, with no secondary process-supervision system, and uses a standard XDR (External Data Representation) format for all canonical data, avoiding proprietary serialization formats.5

#### **The Critical Evolution of Storage Architecture**

The storage backend of Stellar Core has undergone a significant and strategic evolution, moving from a redundant dual-storage system to a highly optimized, unified database. This transition was not merely an incremental improvement but a foundational change that enabled future scalability enhancements.

##### **Legacy Dual-Storage System**

Historically, Stellar Core maintained the ledger state in two separate, redundant locations:

1. **SQL Database:** A standard SQL database (typically PostgreSQL) was used as the primary key-value store. It provided indexed access to ledger entries (like accounts, trustlines, and offers), which was necessary for transaction processing and queries.5  
2. **The BucketList:** Simultaneously, all ledger entries were also stored in the BucketList data structure. The BucketList's primary purpose was to allow for the efficient calculation of a single cryptographic hash representing the entire ledger state at a given point in time.10

This dual system, while functional, suffered from significant inefficiencies. Every ledger close required writing data to both the SQL database and the BucketList, leading to write and disk amplification. Furthermore, a general-purpose SQL database with ACID compliance, while robust, is not optimally designed for Stellar's specific "write once, read many times" access pattern, where the database is effectively immutable during the consensus round.10

##### **The BucketList Data Structure**

The BucketList is a novel data structure that deviates from traditional Merkle trees. It is composed of a fixed number of levels, with each level containing buckets of exponentially increasing size. The structure is temporally organized: ledger entries are placed into buckets based on how recently they were modified. The most recent changes reside in the smallest, lowest-level buckets, which are often held in memory. As entries age without being modified, they are merged into progressively larger, higher-level buckets.5 The cumulative hash of all buckets produces the "BucketList hash," which is included in the ledger header to represent the state of the ledger. This temporal structure is highly efficient for both hashing and for network synchronization, as a node that is out of sync only needs to download the specific buckets that have changed since it went offline.10

##### **The Advent of BucketListDB**

The most significant architectural shift in Stellar's storage layer was the development of **BucketListDB**. This innovation transformed the BucketList from a hash-optimized structure into a fully-fledged, high-performance key-value database, thereby eliminating the need for the separate SQL backend for ledger state.10

This was achieved by augmenting the BucketList with two types of indexes:

* **Bloom Filters:** A lightweight, probabilistic data structure used to quickly determine which level of the BucketList a specific ledger entry might be in, avoiding a costly search through every bucket.11  
* **Bucket Indexes:** Once the correct level is identified, a bucket index maps the ledger key directly to the entry's file offset within that bucket, enabling fast lookups even in multi-gigabyte bucket files.10

The migration to BucketListDB as the default and, eventually, mandatory storage backend (enforced in Protocol 21\) yielded dramatic performance improvements, including a disk usage reduction of approximately 65% and a 400% increase in read speeds.11

However, the importance of this transition extends far beyond these immediate performance gains. The design of BucketListDB was a crucial strategic enabler for the network's future. A key feature built into its core is support for **conflict-free parallel I/O**.12 The ability to handle simultaneous, non-conflicting reads and writes to the state is an absolute prerequisite for any system aiming to execute transactions in parallel. The legacy SQL backend would have been a severe bottleneck in such an environment. Therefore, the development and deployment of BucketListDB was a foundational piece of engineering, deliberately laying the groundwork years in advance for the eventual implementation of Parallel Soroban.

| Metric | Legacy (SQL \+ BucketList) | Modern (BucketListDB) |
| :---- | :---- | :---- |
| **Read Performance** | Slower, limited by SQL performance | \~400% faster due to optimized indexes and direct access 11 |
| **Write Performance** | Inefficient due to write amplification (writing to two systems) 10 | Faster, single write path, optimized for Stellar's workload 12 |
| **Disk Usage** | High due to redundant storage of ledger state 10 | \~65% lower, state is stored only once 12 |
| **Hashing Mechanism** | BucketList | BucketList (integrated within BucketListDB) 10 |
| **Support for Parallel I/O** | No, limited by ACID compliance and single-writer nature of SQL | Yes, designed for conflict-free parallel reads and writes 12 |

### **Section 3: The Stellar Consensus Protocol (SCP) in Detail**

The security and efficiency of the Stellar network are anchored by the Stellar Consensus Protocol (SCP), a novel consensus algorithm developed by Professor David Mazières. SCP provides a method for achieving distributed agreement that is uniquely suited for a network focused on financial transactions, prioritizing speed, low cost, and decentralized control.

#### **Theoretical Underpinnings of SCP**

SCP is a construction of a consensus model known as **Federated Byzantine Agreement (FBA)**.13 This model represents a significant departure from traditional Byzantine Fault Tolerance (BFT) systems, which require a closed, unanimously agreed-upon list of participants. FBA, by contrast, enables

**open membership**, where anyone can join the network as a node.15

* **Quorum Slices:** The central concept that enables FBA is the **quorum slice**. Instead of global agreement on membership, each node in the network independently configures its own quorum slice—a list of other nodes that it considers trustworthy and sufficient for reaching an agreement.15 For example, a node might decide it will agree with a transaction set if 3 out of 5 specific other nodes also agree. That set of 3 nodes constitutes one possible quorum slice for that node.16  
* **Quorum Formation:** A network-wide agreement, or **quorum**, emerges from the intersection of these individual quorum slices. If enough nodes have overlapping quorum slices, the network can reach a global consensus on the state of the ledger without any central authority dictating the participants.17 This decentralized approach to trust is what allows SCP to be both flexible and secure.  
* **Protocol Properties:** Every consensus protocol must make trade-offs between three desired properties: safety, liveness, and fault tolerance. SCP is explicitly designed to prioritize **safety and fault tolerance over liveness**.16  
  * **Safety:** The network will never reach a state where different honest nodes agree on conflicting transaction histories. In other words, the blockchain will not fork.  
  * **Fault Tolerance:** The system can continue to operate correctly even if some nodes fail or behave maliciously.  
  * **Liveness:** The network is able to process new transactions. By prioritizing safety, SCP accepts the possibility that the network could temporarily halt (lose liveness) if a quorum cannot be formed, considering this preferable to the risk of a fraudulent transaction being confirmed.16

#### **The Two-Phase Protocol in Practice**

SCP achieves consensus through a two-phase process, which relies on a mechanism called **federated voting**. In this process, nodes exchange statements about proposed transaction sets, and a statement progresses through three stages of agreement: **Vote, Accept, and Confirm**.16

1. **Nomination Protocol:** This is the first phase, where the goal is to narrow down the universe of possible transaction sets to a manageable list of candidates. Nodes vote for transaction sets they believe should be included in the next ledger. Through rounds of federated voting, nodes observe the votes within their quorum slices and converge on a common set of nominated values. Once a node confirms its first candidate, it stops nominating new ones, ensuring that all honest nodes will eventually agree on the same list of candidates to consider.16  
2. **Ballot Protocol:** Once a candidate transaction set has been nominated, the ballot protocol begins. This phase is designed to ensure that the entire network can unanimously commit to one and only one of the nominated sets. It involves two distinct steps:  
   * **Prepare:** A node broadcasts a "prepare" message to signal its readiness to commit to a specific ballot (a combination of a candidate set and a counter). This step verifies that the node's quorum slice is also willing to commit.  
   * **Commit:** Once a node sees that its quorum slice is prepared to commit, it broadcasts a "commit" message. When a quorum of nodes has committed to the same ballot, that transaction set is considered finalized by the network.16

#### **Comparative Analysis**

Compared to more widely known consensus mechanisms like Proof-of-Work (PoW) and Proof-of-Stake (PoS), SCP offers distinct advantages that align with Stellar's mission. It avoids the computationally intensive and energy-consuming mining process of PoW and the capital-locking requirements of PoS.17 This results in significantly lower transaction costs (a fraction of a cent), fast transaction finality (typically 3-5 seconds), and a much smaller environmental footprint, making it highly suitable for high-volume payments and micropayments.3

## **Part II: The Programmability Layer \- The Introduction of Soroban**

For years, the Stellar network excelled as a highly efficient payment and asset issuance platform. However, the rise of Decentralized Finance (DeFi) and other complex on-chain applications exposed the limitations of its existing programmability model. The strategic introduction of Soroban marked a pivotal evolution, transforming Stellar from a specialized payment rail into a full-fledged, high-performance smart contract platform.

### **Section 4: Soroban: A New Paradigm for Stellar**

To fully appreciate the significance of Soroban, it is essential to first understand the evolution of "smart contracts" on the Stellar network.

#### **From Composable Transactions to a Turing-Complete Platform**

Before Soroban, the term "Stellar Smart Contract" (SSC) referred not to code executed by a virtual machine (VM), but to the combination of standard Stellar transactions with a set of powerful on-chain constraints.19 Developers could create complex logic by composing transactions and leveraging features such as:

* **Multisignature:** Requiring signatures from multiple parties to authorize an operation.  
* **Atomicity (Batching):** Grouping multiple operations into a single transaction that either succeeds or fails as a whole.  
* **Sequence:** Enforcing a specific order for a series of transactions.  
* **Time Bounds:** Defining a time window during which a transaction is valid.

While this model was remarkably effective for use cases like escrows, payment channels, and decentralized exchanges, it lacked the Turing-complete expressiveness of platforms like Ethereum, limiting the complexity of applications that could be built.19

Recognizing this limitation, the Stellar Development Foundation made the strategic decision in 2022 to develop and integrate Soroban, a new, native smart contract platform designed to support sophisticated DeFi protocols and beyond, without compromising the network's core strengths.21

| Feature | "Classic" Stellar Smart Contracts | Soroban Smart Contracts |
| :---- | :---- | :---- |
| **Programmability Model** | Composition of transactions and on-chain constraints 19 | Turing-complete logic executed by a VM 22 |
| **Language** | N/A (defined by transaction structure) | Rust, with potential for other Wasm-compatible languages 23 |
| **Execution Environment** | Stellar Core's transaction processing logic | WebAssembly (Wasm) Virtual Machine 25 |
| **State Management** | Changes to standard ledger entries (accounts, balances) 19 | Contract-specific storage (Instance, Persistent, Temporary) with a rent model 26 |
| **Complexity** | Limited expressiveness, optimized for payments and asset swaps 20 | High expressiveness, capable of complex DeFi logic 28 |
| **Use Cases** | Escrow, payment channels, decentralized exchange (DEX), atomic swaps 19 | Lending/borrowing protocols, AMMs, DAOs, complex derivatives 21 |

#### **Defining Soroban: Design Principles and Technology**

Soroban was engineered from the ground up with a "batteries-included" philosophy, providing developers with a comprehensive and user-friendly toolkit.24 Its architecture is guided by three core design principles:

**performance, sustainability, and security**.23

* **Technology Stack:** Soroban decisively chose **Rust** as its primary development language and **WebAssembly (Wasm)** as its compilation target.23 This choice was strategic. It allows Soroban to leverage the mature ecosystem, tooling, and, most importantly, the safety guarantees of Rust (such as memory safety and prevention of entire classes of bugs like re-entrancy) and the performance and portability of Wasm. This approach contrasts with creating a new, proprietary language and VM, which would come with a steeper learning curve and a less mature security landscape.23  
* **Key Architectural Features:**  
  * **Multi-Dimensional Fee Model:** Moving beyond the single "gas" metric of other platforms, Soroban prices transactions based on a vector of resources, including CPU instructions, memory usage, and I/O operations (bytes read/written).23 This allows for more precise resource accounting and more efficient packing of diverse transactions into a block, which can increase overall throughput and better align fees with actual computational cost.23  
  * **State Archival and Rent:** To combat the long-term problem of "state bloat" that plagues many blockchains, Soroban implements a state rent model. All persistent contract data and code require rent to be paid in XLM to remain in the active ledger state. If the rent for a piece of data expires, it is moved to an off-chain archive. This data is not lost; it can be securely restored to the live ledger by paying the accrued rent.23 This mechanism ensures that the active state remains manageable and that the cost of storing data is borne by those who use it, promoting long-term network sustainability.  
  * **Built-in Authorization Framework:** Security is simplified through a native authorization framework. Instead of developers writing complex and error-prone authorization logic from scratch, Soroban allows contracts to request specific, granular permissions from a user. A developer can then enforce this authorization with a single, simple line of code, such as user.require\_auth(). The Soroban host environment handles the complex underlying mechanics of signature verification and replay prevention, significantly reducing the attack surface and the likelihood of common security vulnerabilities.23

### **Section 5: Soroban's Technical Architecture**

Soroban's design translates its core principles into a concrete technical architecture comprising a specialized execution environment, a nuanced storage model, and a rich developer ecosystem.

#### **Execution Environment**

At the heart of Soroban is its execution environment, which is powered by a **WebAssembly (Wasm) virtual machine**.25 When a developer writes a smart contract in Rust, it is compiled into a highly optimized and portable Wasm bytecode. This bytecode is what gets deployed to the Stellar network. During a transaction, the Soroban runtime within Stellar Core loads and executes this Wasm code, providing a sandboxed and deterministic environment. The use of Wasm not only ensures high performance but also opens the door for future support of other programming languages that can compile to Wasm, such as AssemblyScript, Go, or C++.22

#### **Storage Model Deep Dive**

A critical aspect for any Soroban developer to master is its tripartite storage model. Each storage type is designed for a specific purpose, and misuse can lead to inefficient, expensive, or even vulnerable contracts. All storage is accessed via the env.storage() object within a contract.27

* **Instance Storage:** This storage type is designed for small pieces of data that are intrinsically tied to the contract instance itself, such as an administrator's address, configuration flags, or the address of a token the contract manages.26 Its defining characteristic—and its biggest pitfall—is that its  
  **entire contents are loaded into memory every time any function of the contract is invoked**, regardless of whether the function uses that data. This makes it extremely fast for accessing small, frequently needed configuration data. However, if a developer uses Instance Storage for data that can grow without bounds (e.g., a list of all users who have interacted with the contract), the cost of every transaction will increase over time, potentially leading to a denial-of-service (DoS) vulnerability where the contract becomes too expensive to use.26  
* **Persistent Storage:** This is the primary storage type for the vast majority of contract data that needs to persist across transactions and over long periods. Data in Persistent Storage is subject to the state rent mechanism described earlier.27 Unlike Instance Storage, it is not loaded automatically. The contract must explicitly read the specific keys it needs, making it highly scalable and efficient for managing large amounts of state. This is the appropriate place to store user balances, allowance maps, and other application-specific data that can grow over time.26  
* **Temporary Storage:** This storage type is a cost-effective solution for data that is short-lived or can be easily recreated. Entries in Temporary Storage have a defined Time-to-Live (TTL) and are automatically and permanently deleted from the ledger when their lifetime expires.26 This is ideal for use cases like implementing one-time-use authorization nonces or oracle price data that only needs to be valid for a short period.

#### **Developer Tooling and Ecosystem**

Adhering to its "batteries-included" philosophy, Soroban provides a comprehensive suite of tools designed to create a seamless and productive development workflow.24

* **Soroban CLI:** A powerful command-line interface that is the developer's primary tool for compiling, deploying, and interacting with smart contracts.29  
* **SDKs:** The Stellar Development Foundation maintains official SDKs, most notably in Rust for contract development and JavaScript/TypeScript for client-side application development, which simplify interaction with the Stellar RPC and transaction building.29  
* **Local Sandbox:** A crucial tool that allows developers to run a local, in-memory instance of the Soroban environment. This enables rapid, iterative development, testing, and debugging of contracts entirely on a local machine without needing to deploy to a test network for every change, drastically shortening the development cycle.24

Together, these components create an architecture that is not only powerful and scalable but also accessible and developer-friendly, lowering the barrier to entry for building secure and efficient decentralized applications on Stellar.

## **Part III: The Scalability Leap \- Parallel Soroban and Protocol 23**

The introduction of Soroban provided the Stellar network with the necessary programmability to host complex applications. The next evolutionary step was to ensure the platform could scale to meet the demands of high-throughput DeFi and real-world asset tokenization. This was achieved through the Protocol 23 network upgrade, a landmark release centered on **Parallel Soroban**, a mechanism that fundamentally re-architected transaction processing to unlock massive concurrency.

### **Section 6: The Rationale and Mechanics of Parallel Soroban (CAP-0063)**

Traditional blockchain architectures process transactions sequentially, one after another. This creates an inherent bottleneck: a single complex or slow transaction can delay all subsequent transactions in a block, severely limiting the network's overall throughput.33 The goal of parallelization is to break this bottleneck by executing multiple, non-conflicting transactions simultaneously.

#### **The Core Concept: Conflict-Free Concurrency via Footprints**

Soroban's approach to parallelization is deterministic and conflict-free, a design that contrasts with the "optimistic" parallelization models used by some other blockchains. In an optimistic model, a network executes all transactions in parallel and then retrospectively checks for conflicts, discarding and re-running any work that was invalidated.33 This can lead to wasted computation and unpredictable performance.

Soroban avoids this by requiring transactions to declare their dependencies upfront. This is achieved through a mechanism called **"footprints"**. Every Soroban transaction must include a footprint that explicitly specifies which ledger entries (i.e., pieces of state) it intends to **read** and which it intends to **write**.23 By having this read/write set information available before execution, the network can precisely identify which transactions are independent of one another and can be safely executed in parallel without any risk of conflict.33 If Transaction A only reads state

X and Transaction B only reads state Y, they can run concurrently. Similarly, if both Transaction A and B read state X but neither writes to it, they can also run concurrently. A conflict only arises if one transaction attempts to write to a piece of state that another transaction is either reading or writing.

#### **The Scheduling Algorithm of CAP-0063**

While the concept of footprints existed in Soroban's design from early on, there was no protocol-level mechanism to enforce a parallel-friendly transaction schedule. Validators could theoretically build blocks that were difficult to parallelize. The Core Accepted Proposal (CAP) 0063, titled "Parallelism-Friendly Transaction Scheduling," introduced a new transaction set structure in Protocol 23 to solve this and guarantee predictable parallelization.35

The algorithm organizes transactions within a ledger into a specific structure:

1. **Stages:** A transaction set is divided into a sequence of one or more **'stages'**. These stages must be processed sequentially. This provides a mechanism to handle dependencies between large groups of transactions.35  
2. **Clusters:** Within each stage, transactions are grouped into multiple **'clusters'**. A key property of this structure is that every cluster within a given stage is completely independent of every other cluster in that same stage—meaning their footprints do not overlap.35  
3. **Parallel Execution:** Because all clusters within a stage are independent, they can be executed **in parallel** by the validator, with the work distributed across multiple available CPU cores. The validator processes Stage 1 (executing all its clusters in parallel), then moves to Stage 2 (executing all its clusters in parallel), and so on.35

This staged approach provides a simple yet powerful way to manage conflicts. For instance, imagine a scenario with one "hot" contract entry that is frequently written to (e.g., updating the total supply of a popular token). A transaction that writes to this entry can be placed in a cluster in Stage 1\. Then, dozens of other transactions that only need to *read* that same entry can be grouped into separate clusters in Stage 2, all of which can then be executed in parallel after the initial write is complete.35

This new parallel execution model also necessitated slight adjustments to the logic governing Time-to-Live (TTL) extensions for ledger entries and the calculation of fee refunds to ensure their behavior remained consistent and deterministic in a concurrent environment.35

### **Section 7: The Outcome and System-Wide Impact of Parallel Soroban**

The implementation of Parallel Soroban, alongside other coordinated upgrades in Protocol 23, has had a profound impact on the Stellar network's performance, codebase, and overall capability.

#### **Performance and Throughput Analysis**

The most direct outcome of CAP-0063 is a **significant increase in the per-ledger CPU capacity**.36 By leveraging multiple CPU cores, validators can now process a much higher volume of computational work within the same 5-second ledger closing time.

This CPU enhancement was strategically complemented by other CAPs in Protocol 23 that targeted I/O performance:

* **CAP-0062 (Soroban Live State Prioritization)** and **CAP-0066 (In-Memory Soroban State)** effectively moved the "live" Soroban state (the data most likely to be accessed) into RAM.36 This dramatically increases the per-ledger read capacity and reduces transaction costs by eliminating expensive disk reads from the critical path of smart contract execution.36

While precise, universally applicable TPS (Transactions Per Second) figures are difficult to state due to variance in transaction complexity, the architectural changes provide a clear mechanism for substantially higher throughput. The Stellar ecosystem uses tools like supercluster and its MaxTPSMixed mission to conduct performance testing. This mission generates a configurable mix of classic payments, Soroban contract uploads, and Soroban contract invocation transactions to benchmark the network's maximum sustainable TPS under realistic, mixed workloads.38

#### **Impact on Codebase and Data Structures**

The Protocol 23 upgrade introduced several significant changes to the Stellar Core codebase and its data structures:

* A new transaction set structure was implemented to represent the **stages and clusters** required by the parallel scheduling algorithm.35  
* The state storage was formally separated into a **"Live State" BucketList** and a **"Hot Archive" BucketList** to facilitate the in-memory state optimizations of CAP-0062.36  
* The use of **BucketListDB became mandatory**, and several legacy SQL tables related to ledger state (e.g., ACCOUNT, TRUSTLINE, CONTRACT\_DATA) were unconditionally dropped from the Stellar Core database schema.39  
* New **host functions** were added to the Soroban environment, such as a contract executable getter (CAP-0068) and string/byte conversion utilities (CAP-0069), providing more tools for smart contract developers.36

This set of coordinated changes demonstrates a holistic approach to scalability, addressing CPU, I/O, and state management in a single, major network upgrade.

| CAP Number | Title | Core Objective | Impact on the Network |
| :---- | :---- | :---- | :---- |
| **CAP-0062** | Soroban Live State Prioritization | Separate live state from archived state into different databases (Live State BucketList and Hot Archive BucketList).36 | Improves performance by focusing validator resources on live state; foundational step for in-memory state and full state archival. |
| **CAP-0063** | Parallelism-Friendly Transaction Scheduling | Introduce a new transaction set structure (stages and clusters) to enable parallel execution of Soroban transactions across multiple CPU cores.35 | Massively increases per-ledger CPU capacity and transaction throughput for smart contracts. |
| **CAP-0065** | Soroban Module Cache | Introduce a cache for compiled Wasm modules to avoid redundant compilation work within a transaction.36 | Reduces computational overhead and lowers costs for transactions that invoke the same contract multiple times. |
| **CAP-0066** | In-Memory Soroban State | Move the reading of live Soroban state from disk to RAM.36 | Drastically increases per-ledger read capacity, significantly improving throughput and reducing fees for smart contract invocations. |
| **CAP-0067** | Unified Events for Stellar Assets | Allow standard Stellar "classic" operations (e.g., payments, trustline changes) to emit Soroban-style events.36 | Creates a unified event system, making it easier for indexers and dApps to track all value movements on the network. |

#### **A Convergent Evolution in Blockchain Scalability**

The architectural pattern adopted by Stellar for parallel execution is not an isolated innovation. It represents a form of convergent evolution seen across the most advanced high-throughput blockchains. Solana's parallel runtime, Sealevel, is perhaps the most well-known example of a system that achieves concurrency through a similar mechanism. Solana transactions must also declare upfront all the accounts (state) they will read from or write to during execution.34

This shared approach—requiring explicit state dependency declaration—is a powerful solution to the scalability trilemma. It allows chains to break the sequential execution bottleneck without resorting to the potentially wasteful and complex nature of optimistic concurrency models. The fact that leading platforms like Stellar and Solana have independently arrived at this same fundamental design pattern underscores its efficacy and positions it as a state-of-the-art approach to achieving on-chain scalability. It indicates that Stellar's architectural choices are not just internal improvements but are aligned with the cutting edge of distributed systems research and engineering.

### **Section 8: Implications for Developers and the Future Trajectory**

The Protocol 23 upgrade and the activation of Parallel Soroban represent a significant leap forward for the Stellar network's capabilities. For developers building on Stellar, this evolution brings both powerful new opportunities and a set of critical changes that require attention and action.

#### **Breaking Changes and Required Developer Actions**

Migrating to a Protocol 23-compatible environment is not optional for active developers. Several breaking changes were introduced that necessitate updates to tooling and codebases.

* **Mandatory SDK Upgrades:** This is the most critical action item. Developers **must** upgrade any Stellar SDKs they use (e.g., JavaScript, Python, Go, Rust) to the latest versions that support Protocol 23\. Older SDKs will be unable to correctly construct, sign, or parse transactions for the upgraded network.6  
* **RPC Renaming and Deprecation:** The soroban-rpc name and its associated packages and Docker images are fully deprecated and were removed starting with Protocol 23\. All developer tooling, scripts, and infrastructure configurations must be updated to use the official stellar-rpc name.6  
* **Prohibition of Memos in Soroban Transactions:** CAP-0063 introduced native support for multiplexed accounts (M-accounts) in Soroban. As this provides a more robust and structured way to route funds to sub-accounts, the use of the memo field in transactions that contain Soroban operations is now prohibited.36 Applications that previously relied on memos for this purpose must migrate to using M-accounts.  
* **Other Technical Changes:** Developers should also be aware of more subtle, non-protocol-level changes that could affect data parsing and contract development. These include modifications to how large integers are represented in the XDR-to-JSON conversion process and extensions to the contract specification format to better support events.36

#### **Updated Smart Contract Paradigms**

While the parallel execution of transactions is a protocol-level optimization, it has implications for how developers can design the most efficient smart contracts.

The parallelism is largely **transparent** to the contract author; one does not need to write multi-threaded Rust code or manage concurrency primitives. The Soroban runtime and the validator's scheduler handle the parallel execution automatically based on the transaction footprints.23

However, an advanced understanding of the footprint mechanism can inform better contract design. For example, developers can architect their applications to minimize write conflicts on hot-spot data entries, thereby maximizing the potential for their transactions to be scheduled in parallel. The dramatic increase in throughput and the reduction in costs, particularly for read-heavy operations, enable a new class of complex dApps that were previously infeasible or cost-prohibitive on Stellar. This includes more sophisticated Automated Market Makers (AMMs), lending protocols supporting a wider array of collateral types, and more intricate cross-chain applications that can now leverage larger event payloads for communication.40

#### **Concluding Analysis: A Synthesized Trajectory**

The architectural evolution of the Stellar network, from its inception to the launch of Parallel Soroban, reveals a clear and deliberate multi-year strategy. The journey can be viewed in three distinct phases:

1. **Foundation:** The creation of a lean, fast, and robust payment network built upon the novel Stellar Consensus Protocol (SCP). This phase prioritized speed, low cost, and reliability for its core use case: global payments and asset issuance.  
2. **Programmability:** The introduction of Soroban, which layered a modern, developer-friendly, and secure smart contract platform onto the existing network. This expanded Stellar's capabilities from simple transactions to complex, programmable financial logic, opening the door to the world of DeFi.  
3. **Scalability:** The activation of Parallel Soroban and its associated performance upgrades in Protocol 23\. This phase addressed the critical need for high throughput, ensuring that the newly programmable network could scale to handle real-world, enterprise-grade transaction volumes.

Each phase built logically upon the last. The efficiency of SCP provided a solid base. The design of Soroban with its upfront "footprints" anticipated the need for parallelization. The rollout of BucketListDB provided the necessary high-performance, parallel-ready storage backend. Finally, Protocol 23 delivered the parallel execution engine itself.

This strategic trajectory has positioned the Stellar network to uniquely compete in the blockchain space. It now combines its foundational strengths—a proven, high-speed consensus mechanism and a global network of financial on/off-ramps (Anchors)—with a state-of-the-art, scalable smart contract platform.25 This synthesis of real-world financial integration and high-performance decentralized computation makes Stellar a compelling platform for the next generation of financial services, particularly those focused on the tokenization of real-world assets and the creation of accessible, efficient, and scalable DeFi applications.

#### **Works cited**

1. Learn About the Core Components and Architecture of the Stellar Network | Stellar Docs, accessed July 4, 2025, [https://developers.stellar.org/docs/learn/fundamentals/stellar-stack](https://developers.stellar.org/docs/learn/fundamentals/stellar-stack)  
2. Component overview \- Catalyst Blockchain Manager \- IntellectEU, accessed July 4, 2025, [https://docs.catalyst.intellecteu.com/stellar/Getting-started/component-overview.html](https://docs.catalyst.intellecteu.com/stellar/Getting-started/component-overview.html)  
3. Stellar Architecture: How It Works | by Okereke Innocent | Xord \- Medium, accessed July 4, 2025, [https://medium.com/xord/stellar-architecture-how-it-works-df9fd9470c75](https://medium.com/xord/stellar-architecture-how-it-works-df9fd9470c75)  
4. Stellar \- From Vision to Architecture \- desosa 2021, accessed July 4, 2025, [https://2021.desosa.nl/projects/stellar/posts/vision-to-architecture/](https://2021.desosa.nl/projects/stellar/posts/vision-to-architecture/)  
5. stellar-core/docs/architecture.md at master \- GitHub, accessed July 4, 2025, [https://github.com/stellar/stellar-core/blob/master/docs/architecture.md](https://github.com/stellar/stellar-core/blob/master/docs/architecture.md)  
6. Protocol 23 Upgrade Guide \- Stellar, accessed July 4, 2025, [https://stellar.org/blog/developers/protocol-23-upgrade-guide](https://stellar.org/blog/developers/protocol-23-upgrade-guide)  
7. Administration \- Stellar Core | Stellar Developers, accessed July 4, 2025, [https://docs.stellarcn.org/stellar-core/software/admin.html](https://docs.stellarcn.org/stellar-core/software/admin.html)  
8. stellar-core/src/scp/readme.md at master \- GitHub, accessed July 4, 2025, [https://github.com/stellar/stellar-core/blob/master/src/scp/readme.md](https://github.com/stellar/stellar-core/blob/master/src/scp/readme.md)  
9. Stellar Behind the Scenes. Stellar is one of the platforms that… | by ..., accessed July 4, 2025, [https://medium.com/coinmonks/stellar-behind-the-scenes-5ab66025892d](https://medium.com/coinmonks/stellar-behind-the-scenes-5ab66025892d)  
10. stellar-core/src/bucket/readme.md at master \- GitHub, accessed July 4, 2025, [https://github.com/stellar/stellar-core/blob/master/src/bucket/readme.md](https://github.com/stellar/stellar-core/blob/master/src/bucket/readme.md)  
11. Stellar | Data Structure: BucketlistDB, accessed July 4, 2025, [https://stellar.org/blog/developers/data-structure-bucketlistdb](https://stellar.org/blog/developers/data-structure-bucketlistdb)  
12. Upcoming Database Changes in Protocol 21 \- Stellar, accessed July 4, 2025, [https://stellar.org/blog/developers/upcoming-database-changes-in-protocol-21](https://stellar.org/blog/developers/upcoming-database-changes-in-protocol-21)  
13. How Does Stellar Consensus Protocol Work | by CryptoBuyClub \- Latest Crypto Buying Guide | Medium, accessed July 4, 2025, [https://medium.com/@cryptobuyclub/how-does-stellar-consensus-protocol-work-1aec171406cd](https://medium.com/@cryptobuyclub/how-does-stellar-consensus-protocol-work-1aec171406cd)  
14. The Stellar Consensus Protocol: A Federated Model for Internet-level Consensus | John P Conley, accessed July 4, 2025, [https://johnpconley.com/wp-content/uploads/2021/01/stellar-consensus-protocol.pdf](https://johnpconley.com/wp-content/uploads/2021/01/stellar-consensus-protocol.pdf)  
15. Understanding Stellar: A Deep Dive into Its Architecture and Use Cases \- The Gila Herald, accessed July 4, 2025, [https://gilaherald.com/understanding-stellar-a-deep-dive-into-its-architecture-and-use-cases/](https://gilaherald.com/understanding-stellar-a-deep-dive-into-its-architecture-and-use-cases/)  
16. Overview of the Stellar Consensus Protocol (SCP) and Transaction Validation | Stellar Docs, accessed July 4, 2025, [https://developers.stellar.org/docs/learn/fundamentals/stellar-consensus-protocol](https://developers.stellar.org/docs/learn/fundamentals/stellar-consensus-protocol)  
17. Understanding Stellar Consensus Protocol (SCP) \- hashnode.dev, accessed July 4, 2025, [https://joannetich-1720158416688.hashnode.dev/understanding-stellar-consensus-protocol-scp](https://joannetich-1720158416688.hashnode.dev/understanding-stellar-consensus-protocol-scp)  
18. Understanding the Stellar consensus protocol \- ProHoster, accessed July 4, 2025, [https://prohoster.info/en/blog/administrirovanie/razbiraemsya-v-protokole-konsensusa-stellar-2](https://prohoster.info/en/blog/administrirovanie/razbiraemsya-v-protokole-konsensusa-stellar-2)  
19. How to create Stellar Smart Contracts? \- LeewayHertz, accessed July 4, 2025, [https://www.leewayhertz.com/create-stellar-smart-contracts/](https://www.leewayhertz.com/create-stellar-smart-contracts/)  
20. Mastering Stellar Smart Contracts 2024 Ultimate Guide for Developers \- Rapid Innovation, accessed July 4, 2025, [https://www.rapidinnovation.io/post/how-to-create-stellar-smart-contracts](https://www.rapidinnovation.io/post/how-to-create-stellar-smart-contracts)  
21. Smart Contract Basics \- Stellar, accessed July 4, 2025, [https://stellar.org/learn/smart-contract-basics](https://stellar.org/learn/smart-contract-basics)  
22. Unlocking the Potential of Stellar Soroban: Developer's Guide to Validation Cloud's Node API \- blog, accessed July 4, 2025, [https://blog.validationcloud.io/unlocking-the-potential-of-stellar-soroban-developers-guide-to-validation-clouds-node-api](https://blog.validationcloud.io/unlocking-the-potential-of-stellar-soroban-developers-guide-to-validation-clouds-node-api)  
23. Soroban: The Smart Contract Platform Designed for ... \- Stellar, accessed July 4, 2025, [https://stellar.org/blog/developers/soroban-the-smart-contract-platform-designed-for-developers](https://stellar.org/blog/developers/soroban-the-smart-contract-platform-designed-for-developers)  
24. Soroban: A New Smart Contract Standard \- Stellar, accessed July 4, 2025, [https://stellar.org/blog/developers/soroban-a-new-smart-contract-standard](https://stellar.org/blog/developers/soroban-a-new-smart-contract-standard)  
25. Understanding Soroban | Stellar Smart Contract Platform \- Oodles Blockchain, accessed July 4, 2025, [https://blockchain.oodles.io/blog/soroban-stellar-smart-contract/](https://blockchain.oodles.io/blog/soroban-stellar-smart-contract/)  
26. How to develop securely on Soroban? Storage types with unbounded data \- Veridise, accessed July 4, 2025, [https://veridise.com/blog/learn-blockchain/how-to-develop-securely-on-soroban-storage-types-with-unbounded-data/](https://veridise.com/blog/learn-blockchain/how-to-develop-securely-on-soroban-storage-types-with-unbounded-data/)  
27. Soroban Data Locations & State Management – JamesBachini.com, accessed July 4, 2025, [https://jamesbachini.com/soroban-data-state-management/](https://jamesbachini.com/soroban-data-state-management/)  
28. Stellar Native Assets and Soroban Smart Contracts \- Cryptix AG, accessed July 4, 2025, [https://cryptix.ag/stellar-native-assets-and-soroban-smart-contracts/](https://cryptix.ag/stellar-native-assets-and-soroban-smart-contracts/)  
29. Soroban | Smart Contracts Platform on Stellar, accessed July 4, 2025, [https://stellar.org/soroban](https://stellar.org/soroban)  
30. Understanding Soroban: A Deep Dive into Stellar's Smart Contract Platform \- Medium, accessed July 4, 2025, [https://medium.com/@adityavksingh11617/understanding-soroban-a-deep-dive-into-stellars-smart-contract-platform-introduction-1342d51d12ab](https://medium.com/@adityavksingh11617/understanding-soroban-a-deep-dive-into-stellars-smart-contract-platform-introduction-1342d51d12ab)  
31. Building on Stellar Soroban? Grab this security checklist to avoid vulnerabilities | Smart contract audits from Veridise, accessed July 4, 2025, [https://veridise.com/blog/audit-insights/building-on-stellar-soroban-grab-this-security-checklist-to-avoid-vulnerabilities/](https://veridise.com/blog/audit-insights/building-on-stellar-soroban-grab-this-security-checklist-to-avoid-vulnerabilities/)  
32. Smart Contract Development with Soroban and Hardhat \- Stellar Docs, accessed July 4, 2025, [https://developers.stellar.org/docs/learn/migrate/evm/smart-contract-deployment](https://developers.stellar.org/docs/learn/migrate/evm/smart-contract-deployment)  
33. Parallelization in Blockchains: What it is and How it Works \- Hashlock, accessed July 4, 2025, [https://hashlock.com/blog/blockchain-parallel-transaction-execution](https://hashlock.com/blog/blockchain-parallel-transaction-execution)  
34. Sealevel — Parallel Processing Thousands of Smart Contracts \- Solana, accessed July 4, 2025, [https://solana.com/en/news/sealevel---parallel-processing-thousands-of-smart-contracts](https://solana.com/en/news/sealevel---parallel-processing-thousands-of-smart-contracts)  
35. stellar-protocol/core/cap-0063.md at master \- GitHub, accessed July 4, 2025, [https://github.com/stellar/stellar-protocol/blob/master/core/cap-0063.md](https://github.com/stellar/stellar-protocol/blob/master/core/cap-0063.md)  
36. Announcing Protocol 23 \- Stellar, accessed July 4, 2025, [https://stellar.org/blog/developers/announcing-protocol-23](https://stellar.org/blog/developers/announcing-protocol-23)  
37. Dev Meeting: Protocol 23 Upgrades – Partial State Archival & In-Memory Soroban State : r/Stellar \- Reddit, accessed July 4, 2025, [https://www.reddit.com/r/Stellar/comments/1icyscc/dev\_meeting\_protocol\_23\_upgrades\_partial\_state/](https://www.reddit.com/r/Stellar/comments/1icyscc/dev_meeting_protocol_23_upgrades_partial_state/)  
38. supercluster/doc/measuring-transaction-throughput.md at main ..., accessed July 4, 2025, [https://github.com/stellar/supercluster/blob/main/doc/measuring-transaction-throughput.md](https://github.com/stellar/supercluster/blob/main/doc/measuring-transaction-throughput.md)  
39. Releases · stellar/stellar-core \- GitHub, accessed July 4, 2025, [https://github.com/stellar/stellar-core/releases](https://github.com/stellar/stellar-core/releases)  
40. Stellar Network Upgrade: Increasing Smart Contract Limits | by Scopuly \- Medium, accessed July 4, 2025, [https://scopuly.medium.com/stellar-network-upgrade-increasing-smart-contract-limits-96dee4a294dd](https://scopuly.medium.com/stellar-network-upgrade-increasing-smart-contract-limits-96dee4a294dd)  
41. Stellar | Blockchain Network for Smart Contracts, DeFi, Payments & Asset Tokenization, accessed July 4, 2025, [https://stellar.org/](https://stellar.org/)