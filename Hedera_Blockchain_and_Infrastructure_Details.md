

# **Hedera: An In-Depth Analysis of Hashgraph Technology, Enterprise Governance, and Ecosystem Development**

## **Section 1: The Hedera Paradigm: A Departure from Traditional Blockchain**

Hedera emerges as a distinct class of public distributed ledger, engineered to address the persistent challenges of performance, security, and governance that have constrained the mainstream adoption of first and second-generation blockchain technologies.1 It is not a blockchain but a public network built upon the hashgraph consensus algorithm, a novel approach to achieving distributed consensus that offers significant architectural and performance advantages.3 This foundational technology, combined with a unique governance model, positions Hedera as a platform optimized for enterprise-grade and high-throughput decentralized applications.1

### **1.1 The Hashgraph Consensus Algorithm: A Technical Deep Dive**

The core innovation of Hedera is the hashgraph consensus algorithm, invented by Dr. Leemon Baird.2 Unlike blockchains that organize transactions into a linear, single chain of blocks, hashgraph employs a Directed Acyclic Graph (DAG) data structure to record the flow of transactions.1 This fundamental difference in data structure is the primary source of its efficiency and performance.

#### **Data Structure and Efficiency**

In a traditional blockchain, if two miners create a block at roughly the same time, the network must eventually choose one block to continue the chain and discard the other, creating an "orphan" block. This process is inherently inefficient, as the computational effort and bandwidth used to create and propagate the orphaned block are wasted.4 To prevent the network from being overwhelmed by a proliferation of competing branches, blockchains like Bitcoin employ mechanisms such as Proof-of-Work (PoW) to artificially slow down the rate of block creation.4

Hashgraph, conversely, incorporates every container of transactions into the ledger. No transaction data is ever discarded. All branches of the graph are woven together into a single, cohesive whole, leading to near-perfect efficiency in bandwidth usage.4 This architectural design eliminates the need for PoW, enabling the network to process transactions as quickly as they can be propagated through the network.4

#### **Gossip-about-Gossip Protocol**

To achieve this, hashgraph utilizes a "gossip-about-gossip" protocol for communication between nodes.2 When a node receives a new transaction, it bundles it into an "event" along with a timestamp, a digital signature, and the cryptographic hashes of the two most recent events it has knowledge of (its "parents"). It then randomly selects another node on the network and "gossips" this event to it.2 The receiving node incorporates this new information and repeats the process, gossiping to another random node.

This protocol ensures that information spreads exponentially and rapidly throughout the network.5 The "gossip-about-gossip" component refers to the fact that when a node sends an event, it includes not just the new transaction data but also the hashes of previous events, effectively communicating the history of how information has flowed through the network.9 This creates the DAG structure, which is the hashgraph itself—a verifiable record of who gossiped what to whom, and when.2

#### **Virtual Voting and Consensus Timestamping**

Based on the rich historical data contained within the hashgraph, nodes can determine the consensus order of transactions without needing to send additional voting messages across the network. This process is known as "virtual voting".4 Each node can independently run the same algorithm on its local copy of the hashgraph to calculate what every other node would vote, and thereby determine which transactions have reached consensus.

A transaction achieves consensus once it is known that over two-thirds of the network nodes have received it. The consensus timestamp assigned to a transaction is not the time it was created, but rather the median of the times that each node in the network first received that transaction.5 This method ensures a fair ordering of transactions, as no single node can unduly influence the sequence, a critical feature for applications like decentralized exchanges where order matters.11 Once a consensus timestamp is determined, the transaction is considered final and immutable with 100% certainty.4

#### **Asynchronous Byzantine Fault Tolerance (aBFT)**

The hashgraph consensus algorithm is mathematically proven to be asynchronous Byzantine Fault Tolerant (aBFT), which is considered the highest possible level of security for a distributed system.4 The term "Byzantine Fault Tolerance" refers to a system's ability to reach consensus even if some nodes fail or act maliciously. "Asynchronous" means the system does not need to make any assumptions about how long it takes for messages to pass between nodes, making it resilient to network delays or Distributed Denial of Service (DDoS) attacks.11

This aBFT guarantee was formally verified by Carnegie Mellon University using Coq, a formal proof verification system, confirming the mathematical claims laid out in the original hashgraph tech report.4 This provides a level of security assurance comparable to PoW blockchains but without the immense energy consumption and low throughput.10

### **1.2 Genesis and Evolution: From Patented IP to Open Source**

Hedera's journey from a proprietary technology to a fully open-source public utility reflects a strategic evolution designed to balance commercial innovation with the core tenets of decentralization required for broad adoption.

The foundational work began between 2012 and 2014, when Dr. Leemon Baird started tackling the problem of achieving distributed consensus at scale.13 This research culminated in the development of the hashgraph algorithm in 2015-2016. Shortly thereafter, Dr. Baird and his business partner, Mance Harmon, formed Swirlds, Inc. to commercialize the technology for private, permissioned enterprise use cases.10 The Hashgraph whitepaper was published on May 31, 2016, and the technology was battle-tested in private pilots with organizations like credit unions, which required its bank-grade security features.11

In 2017, with seed funding raised by Swirlds, the formation of Hedera began.13 The founders developed the core governance philosophy centered around a council of leading enterprises, a structure designed to ensure stability and prevent the forks that have plagued other public networks.13 The public launch event for Hedera was held in New York City on March 13, 2018, unveiling the vision for a public network built on the patented hashgraph algorithm.13 The Hedera mainnet went live on August 24, 2018, at which point 50 billion HBAR tokens were minted.11

Initially, the hashgraph algorithm was patented by Swirlds, and Hedera operated under an exclusive license.2 This proprietary model, while protecting the intellectual property, became a point of contention within the broader crypto community, which overwhelmingly favors open-source development. A closed-source foundation was perceived as a barrier to trust, creating fears of vendor lock-in and centralized control.

Recognizing this as a significant obstacle to long-term growth and developer adoption, the Hedera Governing Council made a pivotal strategic decision. In 2022, the Council voted to purchase the full intellectual property rights to the hashgraph algorithm from Swirlds and make the code fully open source under the permissive Apache 2.0 license.2 This move was a direct response to market feedback and a necessary step to compete on a level playing field with other open-source Layer 1 networks.

The commitment to open-source principles was further solidified in September 2024, when Hedera transferred its entire codebase to the Linux Foundation to be managed as a vendor-neutral project named "Hiero".2 This final step ensures that the future development of the core protocol is not controlled by any single entity, including Swirlds or Hashgraph, thereby cementing its status as a public good. This multi-year transition from a patented technology to a foundation-managed open-source project illustrates a pragmatic approach: using IP protection to secure initial innovation and attract enterprise partners, then strategically relinquishing that control to foster a broader, more trustless, and collaborative developer ecosystem.

### **1.3 The Founding Visionaries: Dr. Leemon Baird and Mance Harmon**

The creation and direction of Hedera are driven by its two co-founders, whose complementary expertise in deep technology and business strategy has shaped the project's trajectory.

**Dr. Leemon Baird** is the inventor of the hashgraph consensus algorithm and serves as the Chief Scientist at Hashgraph.4 His academic and professional background is extensive, holding a PhD in Computer Science from Carnegie Mellon University, which he reportedly completed faster than any student in the institution's history.11 He has over two decades of technology experience, including serving as a Professor of Computer Science at the US Air Force Academy and as a senior scientist in several research labs.11 With numerous patents and over 100 publications in peer-reviewed journals on topics ranging from computer security and cryptography to machine learning, Dr. Baird is the primary technical architect behind Hedera's core technology.11

**Mance Harmon** is an experienced technology executive and entrepreneur who partnered with Dr. Baird to commercialize hashgraph and co-found Hedera.2 Harmon's career spans over 20 years in leadership roles at multinational corporations, government agencies, and startups.11 He previously served as the Head of Architecture and Labs at Ping Identity, managed a large-scale software program for the Missile Defense Agency, and was the Course Director for Cybersecurity at the US Air Force Academy.11 As the business leader, Harmon has been instrumental in shaping Hedera's enterprise-focused strategy, its unique governance model, and its path to market.6

In a move to further decentralize the Hedera ecosystem, both Dr. Baird and Harmon transitioned in 2022 from their direct roles at Hedera to leadership positions at Swirlds Labs (now rebranded as Hashgraph).13 Hashgraph is a separate entity that now spearheads much of Hedera's marketing, product innovation, and technical development, functioning as a key contributor to the open-source Hiero project rather than its sole proprietor.16

## **Section 2: Network Architecture and Corporate Governance**

Hedera's infrastructure is a meticulously designed system that separates core responsibilities to optimize for performance, scalability, and accessibility. This technical architecture is managed by a novel governance framework, the Hedera Council, which is a core component of its value proposition for enterprise users seeking stability and predictability.

### **2.1 A Multi-Layered Network Architecture**

The Hedera network consists of three primary components: clients, consensus nodes, and mirror nodes. Each plays a distinct role in the processing and retrieval of transaction data.

#### **Clients**

Clients are the entry point to the network for end-users and decentralized applications. A client is any piece of software, typically utilizing a Hedera SDK, that allows a user to create, cryptographically sign, and submit transactions to the network.5 When creating a transaction, the client can specify parameters such as the maximum transaction fee they are willing to pay (in HBAR) and, for smart contract interactions, the maximum gas limit.5 Once signed by the necessary private keys, the client can submit the transaction to any of the consensus nodes on the network.5

#### **Consensus Nodes (The Mainnet)**

The consensus nodes form the core of the Hedera network, often referred to as the mainnet. These are the machines that actively participate in the hashgraph consensus algorithm.5 Their primary responsibilities are to receive transactions from clients, gossip events to each other to build the hashgraph, reach consensus on the validity and timestamp of transactions, and store the latest state of the public ledger.5

A critical architectural decision for performance is that consensus nodes only persist the latest state of the network (e.g., account balances, smart contract data), not the entire transaction history.5 This keeps the nodes lightweight and fast, enabling high throughput. Currently, the consensus nodes are permissioned, meaning they are owned and operated exclusively by the members of the Hedera Governing Council.5 This model was chosen to ensure maximum stability and security during the network's formative years.

#### **Mirror Nodes**

To provide access to the full, historical record of transactions, Hedera employs a parallel network of mirror nodes.5 These nodes do not participate in consensus. Instead, they receive a continuous, verified stream of transaction data from the consensus nodes and store it in a query-friendly database.8 This architectural separation of concerns is vital: it allows the consensus layer to remain optimized for speed, while the mirror network is optimized for data retrieval, analytics, and exploration.5

Developers can query mirror nodes to get detailed information about past transactions, account history, and other on-ledger data without burdening the consensus nodes.5 While Hedera provides a public mirror node for developers, it is rate-limited to encourage serious applications to run their own mirror nodes, giving them greater control and performance.19

#### **Transaction Lifecycle**

The journey of a transaction through this architecture follows a clear, efficient path:

1. A client application creates a transaction and signs it with the required private keys.5  
2. The client submits the signed transaction to any available consensus node.5  
3. The receiving node performs initial validation (e.g., checking if the paying account has a sufficient balance) and, if successful, packages the transaction into a new event. It then gossips this event to another random node.5  
4. The event spreads exponentially throughout the network via the gossip-about-gossip protocol.5  
5. Within seconds, over two-thirds of the nodes have received the event, and through the virtual voting algorithm, they independently reach consensus on its final timestamp and order in the hashgraph.5  
6. The transaction is now considered final and immutable. The state of the ledger (e.g., account balances) is updated across all consensus nodes.4  
7. A verified record of the transaction is then streamed from the consensus nodes to the mirror network, where it is stored permanently and becomes available for public query.5

### **2.2 The Hedera Council: A Framework for Enterprise-Grade Governance**

Hedera's governance model is one of its most defining features and is deliberately designed to appeal to risk-averse enterprises. Rather than being governed by an anonymous group of miners or a fluctuating set of token holders, Hedera is owned and governed by the Hedera Council, a body of up to 39 world-leading organizations.1

This model provides a level of stability and accountability that is rare in the public DLT space. Enterprises considering building on a public ledger are often deterred by the risks of network forks (like the Ethereum/Ethereum Classic split), unpredictable changes in protocol rules, and a lack of clear legal or regulatory recourse. The Hedera Council is designed to mitigate these specific concerns. By placing governance in the hands of globally recognized and legally accountable institutions, Hedera offers a platform where the rules of the network are managed with the same rigor as critical internet infrastructure. This is not just a feature but a core product offering: governance-as-a-service, selling stability and predictability to a market that desperately needs it.

#### **Mission and Structure**

The Council's mission is to provide stable, decentralized governance that steers the network's strategic direction, ensures its operational integrity, and guarantees it will never fork.13 The Council is structured as a Delaware Limited Liability Company (LLC), providing a formal legal framework for its operations.20 Membership is diversified across industries (tech, finance, legal, telecom, academia) and geographies to prevent any single industry or region from having undue influence.17

#### **Roles and Responsibilities**

Council members have clearly defined responsibilities that go beyond simple voting 18:

* **Node Operation:** The primary technical responsibility of each member is to host and maintain one of the network's initial consensus nodes, ensuring the core infrastructure is run by trusted, high-availability operators.17  
* **Software and Policy Governance:** Members have equal voting rights on all critical network decisions. This includes approving updates to the platform's codebase, making changes to the network's fee schedule, and setting network policies.21  
* **Treasury Management:** The Council oversees the management of the Hedera Treasury, allocating funds (in HBAR) to foster the growth and development of the network and its ecosystem, most notably through the creation of the HBAR Foundation.13  
* **Committee Participation:** Members are expected to contribute their domain expertise to various committees that handle specific aspects of governance. These include the Technical Steering & Product Committee, the Legal & Regulatory Committee, the Membership Committee, and the Treasury Management & Coin Economics Committee.18

#### **Membership and Voting**

To ensure decentralization and prevent stagnation, Council membership is subject to strict rules 18:

* **Term Limits:** Members serve three-year terms and can serve for a maximum of two consecutive terms. This forces rotation and brings new perspectives into the Council over time.20  
* **Equal Voting:** Each member has a single, equal vote on all matters, regardless of their size or how much HBAR they hold. This prevents control from being consolidated by a few powerful entities.17  
* **Investment:** The financial cost of joining is a symbolic $100 initial capital contribution. The true investment is the significant commitment of internal resources—legal, compliance, technical, and executive time—required to fulfill the duties of a council member. This ensures that only organizations with a serious, long-term interest in the network's success will join.21

#### **The Path to Permissionless Nodes**

While the current model provides stability, Hedera has a publicly stated roadmap to transition towards a permissionless node model.5 In this future state, anyone who meets certain technical and staking requirements will be able to operate a consensus node and earn fees for contributing to the network's security and operation.11 This transition is a core component of Hedera's long-term decentralization strategy and is intended to address criticism of the current permissioned structure, ultimately aiming to combine the stability of its council-based governance with the broad participation of a fully permissionless network.11

#### **Table 2.2.1: Profile of the Hedera Governing Council (Selected Members)**

The following table illustrates the diversity and global reach of the organizations governing the Hedera network.

| Member Name | Industry/Sector | Geographic Region | Notable Contribution/Role |
| :---- | :---- | :---- | :---- |
| **Google** | Technology / Cloud | North America | Provides cloud infrastructure expertise and enterprise credibility.2 |
| **IBM** | Technology / Consulting | North America | Brings deep enterprise DLT experience and global consulting reach.2 |
| **Dell Technologies** | Technology / Hardware | North America | Contributes expertise in decentralized infrastructure and hardware security.13 |
| **LG Electronics** | Consumer Electronics | APAC | Represents a major global manufacturer, exploring supply chain and IoT use cases.13 |
| **Boeing** | Aerospace & Defense | North America | Provides insight into complex, high-stakes supply chain and data management.13 |
| **Deutsche Telekom** | Telecommunications | EMEA | A leading global telecom provider, anchoring network infrastructure expertise.2 |
| **DLA Piper** | Legal | Global | A top global law firm providing legal and regulatory guidance to the Council.13 |
| **London School of Economics (LSE)** | Academia | EMEA | Contributes academic rigor, research, and economic modeling expertise.13 |
| **Shinhan Bank** | Financial Services | APAC | A leading South Korean bank, driving innovation in finance and stablecoin remittances.13 |
| **Ubisoft** | Gaming & Entertainment | EMEA | A major video game publisher exploring Web3 gaming and NFT applications.13 |
| **abrdn** | Asset Management | EMEA | A global investment company focused on the tokenization of financial assets.13 |
| **ServiceNow** | Enterprise Software | North America | A leader in digital workflow automation, bringing enterprise software perspective.13 |

## **Section 3: Performance Benchmarks and Economic Model**

Hedera's design choices are directly reflected in its network performance metrics and the economic principles governing its native token, HBAR. The platform's value proposition is heavily reliant on delivering high throughput, fast finality, and predictable, low-cost transactions, all while maintaining a sustainable and energy-efficient footprint.

### **3.1 Network Performance Under Scrutiny**

Hedera's performance claims are central to its positioning as a "third-generation" public ledger, designed to support applications at a scale that is not feasible on earlier blockchain networks.23

#### **Throughput (Transactions Per Second \- TPS)**

The Hedera network is architected to handle a high volume of transactions. While the network's theoretical capacity is designed to be unlimited through future sharding, the current mainnet is throttled to ensure stability.23 For its core services, including HBAR cryptocurrency transfers and the Hedera Consensus Service (HCS), the network is capable of processing over 10,000 transactions per second (TPS).5 It is important to distinguish this capacity from the real-time TPS observed on network explorers. The real-time TPS reflects the actual current usage of the network, which can fluctuate from single digits during quiet periods to thousands of transactions per second during periods of high demand from applications.24

#### **Latency (Time to Finality)**

One of Hedera's most significant performance advantages is its low latency to finality. A transaction on Hedera achieves 100% finality—meaning it is confirmed, immutable, and cannot be reversed or altered—within 3 to 5 seconds.23 Live network dashboards often show this metric averaging around 2.9 seconds.23 This contrasts sharply with the probabilistic finality of PoW blockchains like Bitcoin, where a transaction is only considered increasingly secure over time and may require 10 to 60 minutes for a high degree of confidence.23 This rapid finality is essential for use cases requiring real-time settlement, such as retail payments, online gaming, and financial trading.23

#### **Transaction Fees**

The fee model on Hedera is a cornerstone of its enterprise-focused strategy, designed for predictability and affordability. Unlike the volatile, auction-based gas fee markets of networks like Ethereum, Hedera's transaction fees are fixed in U.S. dollars ($USD).12 Users pay for transactions using HBAR, but the amount of HBAR required is determined by the live HBAR-USD exchange rate at the time of the transaction.21

This provides absolute cost predictability for businesses, allowing them to budget their operational expenses accurately. The fees are also extremely low, making high-volume applications economically viable.7 For example:

* A standard HBAR transfer costs approximately $0.0001 USD.23  
* A transfer of a token created on the Hedera Token Service (HTS) costs $0.001 USD.29  
* Minting a new NFT costs $0.05 USD.30

This fee structure makes business models centered on micropayments or the logging of millions of IoT data points feasible on Hedera, whereas they would be prohibitively expensive on many other public networks.

#### **Energy Efficiency and Sustainability**

As a proof-of-stake network that does not rely on energy-intensive mining, Hedera is exceptionally efficient. A study by University College London (UCL) identified Hedera as the most sustainable public DLT among those analyzed.15 The energy consumption per transaction is cited as 0.00017 kWh, a minuscule fraction of the energy used by Bitcoin or even Ethereum post-Merge.15

Beyond its inherent efficiency, Hedera is committed to being a carbon-negative network. The Hedera Council purchases carbon offsets on a quarterly basis to more than compensate for the network's small carbon footprint.4 This focus on sustainability is not merely an environmental statement but a calculated business strategy. In an era where Environmental, Social, and Governance (ESG) criteria are paramount for large corporations, offering a "green" DLT provides a significant competitive advantage and removes a major reputational barrier to adoption for publicly-traded and environmentally-conscious enterprises.33

### **3.2 The HBAR Token Economy**

The native cryptocurrency of the Hedera network, HBAR, is integral to the platform's operation, security, and economic model. It serves a dual role as both fuel for network services and a tool for securing the network.1

#### **Dual Role of HBAR**

1. **Network Fuel:** HBAR is the utility token used to pay for all transactions and services on the network. This includes fees for transferring HBAR or other tokens, executing smart contracts, writing data to the Consensus Service, and storing files.23 These fees are collected to compensate the consensus node operators for providing the necessary bandwidth, computation, and storage resources to run the network.23  
2. **Network Security:** Hedera uses a proof-of-stake consensus mechanism. HBAR holders can contribute to the network's security by staking their tokens to a consensus node.7 The amount of HBAR staked to a node determines its weighted influence in the consensus process. Transactions are validated by nodes representing over two-thirds of the network's total staked HBAR.23 In return for helping to secure the network, both the node operators and the individual stakers receive staking rewards, which are paid out from network fees.23

#### **Token Supply**

The total supply of HBAR is fixed and was created at the network's genesis. A total of 50 billion HBAR were minted, and the protocol does not allow for any further minting.11 The circulating supply, which is the number of tokens available on the open market, is a subset of this total supply and is released over time according to a pre-defined distribution schedule managed by the Hedera Council.24

### **Table 3.1.1: Comparative Analysis of Network Performance Metrics**

To contextualize Hedera's performance claims, the following table compares its key metrics against those of Bitcoin and Ethereum, the first and second-generation leaders in the space.

| Metric | Hedera | Ethereum | Bitcoin |
| :---- | :---- | :---- | :---- |
| **Transactions Per Second (TPS)** | 10,000+ (throttled capacity) | \~15-30 | \~4-7 |
| **Transaction Finality** | 3-5 seconds (absolute) | 10-20 seconds (probabilistic) | 10-60 minutes (probabilistic) |
| **Average Fee (USD)** | \~$0.0001 \- $0.01 | $0.68+ | $10+ |
| **Energy per Transaction (Wh)** | 0.003 | 9.956 | 2,927,000 |

Data sourced from official Hedera documentation and comparative analyses.23 Fees for Ethereum and Bitcoin are highly variable and represent averages at the time of the source publication.

## **Section 4: The Developer's Toolkit: Building on Hedera**

Hedera provides a comprehensive suite of services, tools, and software development kits (SDKs) designed to facilitate the creation of decentralized applications. The platform's strategy is two-pronged: it offers powerful, native services that leverage the unique capabilities of the hashgraph algorithm, while also providing full compatibility with the Ethereum Virtual Machine (EVM) to lower the barrier to entry for the world's largest blockchain developer community.

### **4.1 Core Network Services (APIs)**

Developers interact with the Hedera network primarily through three core services, each exposed via an API.5

#### **Hedera Token Service (HTS)**

The Hedera Token Service (HTS) allows for the native configuration, minting, and management of both fungible and non-fungible tokens (NFTs).4 Unlike tokens on Ethereum (such as ERC-20s), which are defined and managed by smart contract logic, HTS tokens are a native primitive of the Hedera ledger itself. This native implementation means that operations like transferring or minting tokens are executed with the same high performance and low, fixed cost as a simple HBAR transfer, without invoking the more expensive and slower Smart Contract Service.12 HTS is designed for interoperability, with its token standards mapping directly to the widely adopted ERC-20 and ERC-721 formats, ensuring compatibility with wallets and applications across the Web3 ecosystem.12

#### **Hedera Consensus Service (HCS)**

The Hedera Consensus Service (HCS) offers developers direct, low-level access to the hashgraph consensus algorithm.4 It allows any application, whether a Web3 dApp or a traditional Web2 system, to submit messages to the Hedera network and receive a trusted, immutable, and fairly ordered consensus timestamp.5 HCS effectively functions as a decentralized and verifiable logging service or message bus. It is ideal for use cases that require a high-integrity audit trail, such as supply chain tracking, data integrity verification, or serving as a decentralized ordering service for other private or public blockchain frameworks.7

#### **Hedera Smart Contract Service (HSCS)**

The Hedera Smart Contract Service (HSCS) provides a platform for deploying and executing smart contracts written in Solidity, the programming language popularized by Ethereum.1 Hedera's smart contract engine is designed to be equivalent to the Ethereum Virtual Machine (EVM), a crucial feature for developer adoption.15 This EVM equivalence means that the vast ecosystem of existing Ethereum development tools, libraries, and expertise is directly transferable to Hedera. Developers can use familiar tools like Hardhat, Truffle, Ethers.js, and Web3.js to compile, test, and deploy their Solidity contracts on Hedera with minimal or no code changes.12 The service is also highly performant, capable of processing up to 15 million gas per second, a throughput metric comparable to what an entire Ethereum block aims to achieve.12

This dual approach of offering both powerful native services and familiar EVM compatibility is a pragmatic strategy. It allows Hedera to attract developers from the large Ethereum ecosystem with the promise of lower fees and higher performance, effectively meeting them where they are. Once these developers are in the ecosystem, they can be introduced to the unique advantages of native services like HTS and HCS, which offer performance and cost efficiencies that are not possible within the confines of the EVM alone.

### **4.2 Software Development Kits (SDKs) and Tools**

To simplify interaction with its core services, Hedera provides a range of SDKs and supporting development tools.4

#### **Official and Community SDKs**

Hedera and its open-source community, under the Hiero project, maintain official SDKs for several popular programming languages, all licensed under Apache 2.0 39:

* **Java SDK:** A robust SDK for enterprise and Android development.39  
* **JavaScript SDK:** The most common choice for web and dApp development, with support for React Native via Expo.39  
* **Go SDK:** A high-performance SDK suitable for backend services and infrastructure tooling.39  
* **Other Languages:** Official SDKs are also available for Swift, Rust, and C++, with a community-maintained SDK for Python, ensuring broad language support.39

These SDKs provide a high-level, language-specific interface that abstracts away the low-level gRPC calls to the network nodes. They allow developers to easily perform fundamental operations such as configuring a client to connect to the mainnet, testnet, or a local development network; creating accounts and managing cryptographic keys; and building, signing, and submitting all types of transactions and queries to the network.44

#### **Developer Environment and Tools**

The developer ecosystem is supported by a variety of tools designed to streamline the development and deployment lifecycle:

* **Local Network Setup:** Developers can run a complete Hedera network on their local machine using Docker. This provides a free and isolated environment for developing and testing applications without consuming real HBAR on the public testnet.10  
* **JSON-RPC Relay:** This is a critical piece of infrastructure that makes Hedera's EVM compatible with the broader Ethereum ecosystem. The open-source JSON-RPC relay translates the standard Ethereum JSON-RPC API calls (used by tools like MetaMask and Hardhat) into the native Hedera API format, acting as a universal adapter.12  
* **Wallet Integrations:** The ecosystem provides tools and SDKs for easy integration with both native Hedera wallets (like HashPack and Blade) and EVM wallets. The Hedera Wallet Snap, for instance, is a plugin that allows users to interact with Hedera dApps directly from their MetaMask wallet, significantly reducing friction for users coming from other EVM chains.12

The existence of these native services, particularly HTS and HCS, represents Hedera's primary long-term strategic advantage. While EVM compatibility is a necessary gateway for adoption, it is a feature offered by many competing networks. The native services, however, provide capabilities that are fundamentally different and more efficient than what can be achieved with smart contracts alone. The ability to mint and transfer tokens for a fraction of a cent (HTS) or to create a high-throughput, verifiable audit log for any application (HCS) enables business models that are simply not practical on other platforms. The ultimate success of Hedera will likely be driven by the adoption of these unique primitives, which allow developers to build a new class of decentralized applications.

## **Section 5: The Hedera Ecosystem in Focus: 10 Flagship Projects**

The theoretical advantages of the Hedera network are best understood through the practical applications being built upon it. A diverse ecosystem of projects is leveraging Hedera's specific features—low fees, high throughput, native tokenization, and consensus-as-a-service—to build solutions across DeFi, real-world asset (RWA) tokenization, gaming, and the creator economy. An analysis of these flagship projects reveals a clear trend towards use cases that require enterprise-grade performance and auditability.

### **5.1 SaucerSwap: DeFi and Automated Market Making**

SaucerSwap is the premier decentralized exchange (DEX) on the Hedera network, providing a suite of DeFi services including token swaps, liquidity provision, yield farming, and staking.33 As an automated market maker (AMM), it allows users to trade digital assets in a peer-to-peer fashion without a central intermediary.50

**Leverage of Hedera's Features:**

* **Hybrid Service Use:** SaucerSwap utilizes a combination of Hedera's services. The core trading logic and liquidity pools are managed by Solidity smart contracts running on the Hedera Smart Contract Service (HSCS), taking advantage of Hedera's EVM compatibility.47  
* **HTS Efficiency:** It facilitates swaps of tokens created with the Hedera Token Service (HTS), which is significantly faster and cheaper than trading standard ERC-20 tokens that rely solely on smart contract interactions.48  
* **Cost-Effectiveness:** The viability of a DEX is heavily dependent on low transaction fees. Hedera's predictable, sub-cent fee model is a critical enabler for SaucerSwap, allowing it to offer a more cost-effective trading experience compared to Ethereum-based counterparts.47  
* **Governance via HCS:** The protocol uses the Hedera Consensus Service (HCS) to maintain a transparent and immutable log for its governance structure, ensuring that all policy decisions are verifiably recorded.47

### **5.2 Hashport: Cross-Network Interoperability**

Hashport is an enterprise-grade interoperability protocol, or "bridge," that enables the secure transfer of digital assets between Hedera and other major blockchain networks, including Ethereum, Polygon, Avalanche, and BNB Chain.27

**Leverage of Hedera's Features:**

* **HCS as a Security Layer:** The Hedera Consensus Service is the bedrock of Hashport's security model. When an asset is bridged from one network to another, HCS is used to create a final, immutable, and consensus-timestamped record of the event. This serves as the ultimate source of truth for the bridge's validators, providing a higher degree of security than many other bridge designs.27  
* **Efficient Token Minting with HTS:** When assets are transferred *to* Hedera, Hashport uses HTS to mint a representative "wrapped" token (e.g., wETH). HTS allows this to be done with high efficiency and minimal cost.27  
* **Fast Finality:** Hedera's 3-5 second finality allows Hashport to confirm the initial "locking" of assets on the source chain with great speed and certainty, significantly improving the user experience of the bridging process.27

### **5.3 Stader Labs: Liquid Staking for HBAR**

Stader Labs is a multi-chain liquid staking platform that operates on Hedera, allowing HBAR holders to stake their tokens to help secure the network while retaining liquidity.33 Users deposit HBAR and in return receive HBARX, a liquid token that represents their staked position and accrues rewards.55

**Leverage of Hedera's Features:**

* **Native Staking Integration:** Stader's smart contracts directly interact with Hedera's native proof-of-stake mechanism, delegating users' HBAR to network nodes to earn rewards.55  
* **HTS for Liquid Tokens:** The HBARX token is created using the Hedera Token Service. The efficiency of HTS is crucial for minting and transferring these liquid staking derivatives at scale.55  
* **DeFi Composability:** As an HTS token, HBARX is natively compatible with the entire Hedera DeFi ecosystem. Users can, for example, provide their HBARX as liquidity on SaucerSwap, enabling them to earn both staking rewards and trading fees simultaneously, a concept known as "yield stacking".53

### **5.4 Zuse Market: NFT Launchpad and Marketplace**

Zuse Market is a prominent NFT marketplace and launchpad on Hedera, with a mission to simplify the process of creating, buying, and selling digital collectibles.33

**Leverage of Hedera's Features:**

* **Cost-Effective NFTs via HTS:** Zuse is built entirely on the Hedera Token Service for its NFT operations. This allows creators to launch entire collections for a fixed fee of $1 and mint individual NFTs for just $0.05, with transfers costing a mere $0.001. These predictable micro-fees make large-scale NFT projects economically viable.31  
* **Protocol-Enforced Royalties:** HTS includes a native royalty feature that can be configured when an NFT is created. Zuse leverages this to ensure that creators automatically receive a percentage of all secondary sales, a more robust and secure method than relying on marketplace-level enforcement.31  
* **Atomic Swaps:** The platform utilizes Hedera's native atomic swap capability, which allows for the trustless exchange of an NFT for HBAR (or another token) in a single, indivisible transaction, eliminating counterparty risk in peer-to-peer sales.32

### **5.5 Calaxy: The Creator Economy and Social Finance**

Co-founded by NBA player Spencer Dinwiddie, Calaxy is a "social marketplace" designed to empower creators to monetize their content and engage with their fans directly, bypassing traditional social media intermediaries.33

**Leverage of Hedera's Features:**

* **Micropayments:** Hedera's extremely low transaction fees (e.g., $0.001 for a USDC transfer) are fundamental to Calaxy's business model, enabling creators to sell low-cost digital items or offer paid micro-interactions.29  
* **NFTs as Experiences:** Each unique creator experience offered on the platform, such as a personalized video shout-out, is backed by an NFT minted on HTS, providing the fan with a verifiable and ownable digital receipt of their interaction.62  
* **Native Wallets and Tokens:** Every Calaxy user is provided with a non-custodial Hedera wallet, abstracting away the complexities of Web3 for a mainstream audience. The platform's native utility token, $CLXY, is an HTS token used for governance and in-app transactions.62

### **5.6 HeadStarter: A Launchpad for Ecosystem Growth**

HeadStarter is a project accelerator and Initial DEX Offering (IDO) launchpad dedicated to curating, supporting, and providing decentralized funding for promising new projects building on Hedera.33

**Leverage of Hedera's Features:**

* **Decentralized Fundraising:** HeadStarter uses Hedera smart contracts to facilitate its IDOs. This allows the community to invest in new projects by swapping HBAR or stablecoins for the project's new HTS-based token in a secure and transparent manner.66  
* **Performance for Token Sales:** The platform explicitly chose Hedera for its high throughput and low fees. These features are essential for conducting a public token sale, which can involve thousands of participants transacting in a short period, without network congestion or exorbitant costs.66

### **5.7 Bitcarbon (Diamond Standard): Real-World Asset (RWA) Tokenization**

Diamond Standard is a pioneering project in the tokenization of real-world assets, specifically creating a regulated, fungible commodity from physical diamonds. Bitcarbon is the fungible token that represents fractional ownership of this diamond commodity.33

**Leverage of Hedera's Features:**

* **HTS for Asset Representation:** Diamond Standard uses HTS to create "Commodity Tokens," where each token is a digital title directly linked to a specific, audited, and insured set of diamonds held in secure vaults. The Bitcarbon token is also an HTS token, enabling fractional investment in this physical asset pool.70  
* **Transparency and Auditability:** The Hedera public ledger provides an immutable and transparent record of ownership, bringing unprecedented trust and traceability to the traditionally opaque diamond market.70  
* **Scalability for Liquid Markets:** Hedera's high TPS and low fees are critical for enabling a liquid and efficient secondary market for these diamond-backed tokens, allowing them to be traded as easily as any other digital asset.70

### **5.8 Dovu: Enterprise-Grade Sustainability and Carbon Offsetting**

Dovu is a platform focused on environmental sustainability, creating a marketplace for the tokenization and trading of carbon offsets.33

**Leverage of Hedera's Features:**

* **HCS for Data Integrity:** Dovu leverages the Hedera Consensus Service to create a verifiable and auditable log of carbon credit data, from origination to retirement. This ensures the integrity of the data and helps prevent issues like the double-counting of offsets, a critical requirement for a credible carbon market.37  
* **HTS for Carbon Credits:** The platform's carbon credit tokens (e.g., cDOV) are issued on the Hedera Token Service, allowing for the efficient, low-cost transfer and retirement of these assets on-ledger.33  
* **ESG and Brand Alignment:** Dovu's core mission of sustainability aligns perfectly with Hedera's own commitment to being a carbon-negative network, creating a powerful narrative and strategic synergy.33

### **5.9 Red Swan: Tokenization of Commercial Real Estate**

Red Swan is a digital investment platform that specializes in tokenizing high-value commercial real estate, making this traditionally illiquid asset class accessible to a broader range of investors through fractional ownership.38

**Leverage of Hedera's Features:**

* **HTS for Security Tokens:** Red Swan uses the Hedera Token Service to issue security tokens that represent legal ownership shares in specific commercial properties. HTS's built-in compliance features, such as the ability to freeze accounts or enforce KYC, are valuable for regulated assets.  
* **Efficiency and Transparency:** By recording ownership and transactions on Hedera's ledger, Red Swan brings efficiency, transparency, and immutability to real estate transactions, reducing the reliance on slow and costly traditional intermediaries.38

### **5.10 The HBAR Foundation: Fueling Ecosystem Development**

The HBAR Foundation is not a dApp but rather a critical, independent organization that acts as a centralized growth engine for the decentralized Hedera network.11 Funded with a substantial allocation of 10.7 billion HBAR from the Hedera Treasury, its sole mission is to accelerate the growth of the ecosystem by providing grants, resources, and strategic support to builders.13

Role in the Ecosystem:  
The Foundation is a pragmatic solution to the "cold start" problem faced by new Layer 1 networks. Instead of waiting for organic growth, the Foundation strategically deploys capital to fund the development of essential infrastructure (wallets, explorers, bridges) and to incentivize promising applications across DeFi, NFTs, gaming, and sustainability.75 Many of the projects listed above, such as HeadStarter and Zuse Market, have received grants and support from the HBAR Foundation, demonstrating its pivotal role as a catalyst for innovation on the network.75 This centralized, venture-style approach, while in tension with the purest ideals of decentralization, has proven to be a powerful tool for competing in the highly competitive public ledger market.  
The collective analysis of these projects reveals a clear pattern: the most compelling and differentiated use cases on Hedera are those that lean heavily into the unique strengths of HTS and HCS. The ecosystem is carving out a distinct niche in Real-World Asset (RWA) tokenization and enterprise-grade audit trails, domains where the low, predictable costs and verifiable data logging provide a decisive advantage over other platforms.

## **Section 6: Stablecoins: The Cornerstone of a Digital Economy**

Stablecoins are a critical piece of infrastructure for any distributed ledger aiming for mainstream adoption, serving as a bridge between the traditional financial system and the digital asset economy. On Hedera, the stablecoin ecosystem is anchored by the native integration of USD Coin (USDC) and supported by tools designed to empower institutional issuance.

### **6.1 The Stablecoin Landscape on Hedera**

The primary stablecoin on the Hedera network is USD Coin (USDC), issued by the regulated financial technology firm Circle.29

#### **USDC on Hedera**

USDC was the first stablecoin to launch on Hedera and is deeply integrated into its DeFi applications, serving as the primary on-chain unit of account for dollar-denominated value.76 A key characteristic of USDC on Hedera is that it is a

*natively issued* token using the Hedera Token Service (HTS), not a bridged or wrapped version from another chain.29 This native implementation means that USDC inherits the full performance and economic benefits of the Hedera network:

* **High Performance:** Transactions settle with finality in 3-5 seconds.29  
* **Low, Predictable Fees:** A USDC transfer on Hedera costs a fixed fee of approximately $0.001 USD, paid in HBAR, regardless of the transaction value.29 This makes it highly suitable for payments and micropayments.

#### **Backing and Reserves**

USDC is a fully fiat-collateralized stablecoin, designed to maintain a 1:1 peg with the U.S. dollar. Each USDC in circulation is 100% backed by highly liquid cash and cash-equivalent assets held in segregated reserves.29 To ensure transparency and trust, Circle provides monthly attestations of these reserves, conducted by a major third-party accounting firm.79

#### **Commodity-Backed Alternatives**

While USDC provides a fiat-backed stable value, the ecosystem also features innovative alternatives. Bitcarbon, from Diamond Standard, is positioned as an inflation-hedged commodity token. It is a fungible token backed by a dynamic pool of physical, investment-grade diamonds, offering a store of value that is not tied to a fiat currency.33

### **6.2 The Stablecoin Studio: Empowering Institutional Issuers**

Recognizing the growing interest from financial institutions and enterprises in issuing their own digital currencies, Hedera has developed the Stablecoin Studio. This is an open-source Software Development Kit (SDK) designed to be an all-in-one toolkit for institutional issuers to easily build, deploy, and manage their own stablecoins on the network.30

The development of this studio is a clear strategic move. It is not intended for retail users but is a B2B offering aimed squarely at the future of finance, where banks, payment providers, and corporations may issue their own tokenized dollars or other digital assets. By providing the essential infrastructure for compliance, control, and integration, Hedera is positioning itself as the foundational layer for this emerging wave of regulated, private-label stablecoins.

**Key Features of Stablecoin Studio:**

* **Comprehensive Administrative Controls:** The SDK provides issuers with the necessary tools to manage their token in a compliant manner, including the ability to mint, burn, freeze, wipe, and pause token activity.30  
* **Compliance-First Architecture:** It has built-in support for Hedera's native Know Your Customer (KYC) and Anti-Money Laundering (AML) account flags. It also offers integrations with qualified custody providers (such as Hex Trust and Zodia) and on-chain oracles for transparent proof-of-reserve reporting.30  
* **Hybrid Token Model:** The studio utilizes a hybrid approach, leveraging the high performance and low cost of the Hedera Token Service for the underlying token, while using the Hedera Smart Contract Service to implement custom logic, such as programmable issuance schedules or unique governance rules.30

### **6.3 HBAR-to-Stablecoin Exchange Pathways**

While Hedera-native USDC is technologically efficient, a significant practical challenge for users is the current lack of direct on-ramps and off-ramps from major centralized exchanges (CEXs) like Binance or Coinbase.81 These exchanges support trading and withdrawal of HBAR, but not the HTS-based version of USDC. This creates a "liquidity paradox": the asset is highly performant but not easily accessible, representing a major friction point for user adoption.

To acquire Hedera-native USDC, users must typically follow one of two pathways:

#### **Pathway 1: On-Chain Swaps via Decentralized Exchanges (DEXs)**

This is the most common method for existing crypto users. The process involves multiple steps:

1. Purchase HBAR on a major CEX.  
2. Withdraw the HBAR to a Hedera-compatible wallet, such as HashPack or Blade.  
3. Connect the wallet to a Hedera-based DEX like SaucerSwap or HeliSwap.  
4. Execute a swap from HBAR to USDC on the DEX.81

To cash out, the process must be reversed: swap USDC back to HBAR on the DEX, then transfer the HBAR to a CEX to sell for fiat currency.81

#### **Pathway 2: Direct Fiat On-Ramps**

For users who prefer to use fiat currency directly, several Hedera wallets, including HashPack, have integrated third-party on-ramp services like Banxa and MoonPay.83 These services allow users to buy HBAR or USDC directly within the wallet using a credit card or bank transfer. While more direct, this pathway often involves higher fees (ranging from 3% to over 8%) and requires the user to complete a separate KYC process with the on-ramp provider.81

Overcoming the CEX liquidity gap is a critical strategic challenge for the Hedera ecosystem. The lack of direct support is a classic "chicken-and-egg" problem: exchanges may cite a lack of user demand for the asset, while the cumbersome user experience required to acquire it naturally suppresses that demand. The successful listing of native USDC on major exchanges would be a significant catalyst for growth, dramatically reducing friction and unlocking broader adoption for payments and DeFi on Hedera.

## **Section 7: Analyst's Conclusion and Forward Outlook**

Hedera presents a compelling and technologically distinct alternative in the crowded landscape of public distributed ledgers. Its foundation on the hashgraph consensus algorithm, coupled with a unique enterprise-led governance model, results in a platform with demonstrable strengths in performance, cost-efficiency, and stability. However, its strategic choices also present a unique set of challenges related to decentralization perceptions and ecosystem maturity that will be critical to its long-term success.

### **7.1 Summary of Strengths**

* **Technological Superiority:** The hashgraph consensus mechanism is objectively more efficient than traditional blockchain designs. It delivers high throughput (10,000+ TPS capacity), fast and absolute transaction finality (3-5 seconds), and exceptional energy efficiency, providing a robust technical foundation for scalable applications.  
* **Enterprise-Grade Governance:** The Hedera Council, composed of globally recognized and diverse organizations, offers a level of stability, predictability, and accountability that is highly attractive to large corporations, institutions, and regulated industries. This model effectively mitigates the risks of contentious forks and erratic governance that have hindered enterprise adoption of other public networks.  
* **Cost-Effective and Predictable Economics:** Hedera's decision to denominate transaction fees in USD and keep them fixed at extremely low levels is a significant competitive advantage. This provides the cost predictability required for businesses to build sustainable, high-volume applications, a feat that is often impossible on networks with volatile gas fee markets.  
* **Specialization in RWA and Audit Trails:** The Hedera ecosystem is developing a clear product-market fit around the tokenization of real-world assets (RWA) and the creation of verifiable data logs. The native Hedera Token Service (HTS) and Hedera Consensus Service (HCS) provide foundational primitives that are uniquely suited for these high-growth, enterprise-focused markets.

### **7.2 Summary of Challenges and Strategic Risks**

* **The Decentralization Narrative:** Despite a clear and committed roadmap towards a permissionless architecture, the Hedera network is, at present, operated by a permissioned set of council members. This creates a persistent perception of centralization within the broader crypto community, which may limit adoption by developers and users who prioritize the "cypherpunk" ethos of maximal decentralization above all else.  
* **Ecosystem and Liquidity Immaturity:** While growing rapidly, the ecosystem of developers, applications, and on-chain liquidity is still smaller than that of established competitors like Ethereum. A critical bottleneck remains the friction in accessing native Hedera assets, particularly USDC, due to a lack of direct support from major centralized exchanges. This increases the barrier to entry for new users and hampers the network effect.  
* **Intense Competition:** Hedera operates in a fiercely competitive Layer 1 market. It must contend not only with the entrenched network effects of Ethereum but also with a host of other high-performance blockchains and DAG-based projects, all vying for the same pool of developer talent, user attention, and capital.

### **7.3 Forward Outlook and Key Signposts for the Future**

Hedera's future trajectory will be defined by its ability to execute on its strategic roadmap and capitalize on its unique strengths. Several key developments will serve as critical signposts of its progress:

* **The Permissionless Transition:** The successful rollout of permissionless consensus nodes is the single most important milestone on Hedera's roadmap. The manner in which this is executed, and its subsequent impact on network performance, security, and decentralization, will be a defining test of the project's long-term vision.  
* **The Enterprise Adoption Flywheel:** A crucial indicator of success will be the conversion of pilot projects and proofs-of-concept from council members and other enterprises into large-scale, high-volume production applications. A single major use case—such as a national payment system, a global supply chain platform, or a widely adopted tokenized asset from a member institution—could trigger a cascade of further enterprise adoption.  
* **Bridging the Liquidity Gap:** The listing of native Hedera assets, especially HTS-based USDC, on top-tier centralized exchanges is a vital catalyst for growth. Achieving this would dramatically reduce user friction, unlock retail liquidity, and more seamlessly integrate Hedera's DeFi ecosystem with the broader crypto market.  
* **The Rise of Institutional Stablecoins:** The adoption of the Stablecoin Studio by major financial institutions or enterprises to issue their own regulated stablecoins would represent a transformative, long-term victory. This would validate Hedera's strategy to become the foundational infrastructure for the B2B side of the digital currency revolution.

### **7.4 Final Assessment for the Technically-Savvy Professional**

For the technically-savvy developer, investor, or corporate strategist, Hedera represents a highly-engineered and strategically differentiated distributed ledger technology. It has made a deliberate and transparent trade-off, prioritizing enterprise-grade performance, stability, and governance over the immediate pursuit of maximal, anonymous decentralization.

For organizations focused on building high-throughput, low-cost, and regulatory-compliant applications, Hedera offers a compelling and technologically robust platform. Its strengths are particularly pronounced in the burgeoning fields of Real-World Asset (RWA) tokenization, micropayments, and verifiable data integrity, where its native services provide a distinct competitive edge.

Potential stakeholders must, however, weigh these significant advantages against the current realities of a maturing ecosystem and tangible liquidity friction. Hedera's ultimate success will hinge on its ability to flawlessly execute its decentralization roadmap, leverage its powerful governance model to catalyze a flywheel of meaningful enterprise adoption, and solve the critical on-ramping challenges that currently constrain its network effect. It is a long-term project built on sound engineering principles, with a clear vision for becoming the trust layer for the enterprise-driven digital economy.

#### **Works cited**

1. Hedera Hashgraph (HBAR) Explained: A Beginner's Guide \- OSL, accessed July 22, 2025, [https://www.osl.com/hk-en/academy/article/hedera-hashgraph-hbar-explained-a-beginners-guide](https://www.osl.com/hk-en/academy/article/hedera-hashgraph-hbar-explained-a-beginners-guide)  
2. Hedera (distributed ledger) \- Wikipedia, accessed July 22, 2025, [https://en.wikipedia.org/wiki/Hedera\_(distributed\_ledger)](https://en.wikipedia.org/wiki/Hedera_\(distributed_ledger\))  
3. www.osl.com, accessed July 22, 2025, [https://www.osl.com/hk-en/academy/article/hedera-hashgraph-hbar-explained-a-beginners-guide\#:\~:text=Hedera%20Hashgraph%20is%20a%20decentralized,faster%20and%20more%20secure%20transactions.](https://www.osl.com/hk-en/academy/article/hedera-hashgraph-hbar-explained-a-beginners-guide#:~:text=Hedera%20Hashgraph%20is%20a%20decentralized,faster%20and%20more%20secure%20transactions.)  
4. What is Hedera? | Hedera, accessed July 22, 2025, [https://hedera.com/learning/hedera-hashgraph/what-is-hedera-hashgraph](https://hedera.com/learning/hedera-hashgraph/what-is-hedera-hashgraph)  
5. How it works \- Hedera, accessed July 22, 2025, [https://hedera.com/how-it-works](https://hedera.com/how-it-works)  
6. Hedera Hashgraph Co-founders Mance Harmon and Dr. Leemon, accessed July 22, 2025, [https://hedera.com/blog/hedera-hashgraph-co-founders-mance-harmon-and-dr-leemon-baird-win-emerging-company-ceo-and-technology-inventor-at-tech-titans-awards](https://hedera.com/blog/hedera-hashgraph-co-founders-mance-harmon-and-dr-leemon-baird-win-emerging-company-ceo-and-technology-inventor-at-tech-titans-awards)  
7. Solving Blockchain's Scalability Problem: A Deep Dive into Hedera Hashgraph, accessed July 22, 2025, [https://bladewallet.io/blog/solving-blockchains-scalability-problem-a-deep-dive-into-hedera-hashgraph/](https://bladewallet.io/blog/solving-blockchains-scalability-problem-a-deep-dive-into-hedera-hashgraph/)  
8. Unveiling the High Throughput of Hedera Hashgraph: A Deep Dive into Hashgraph Consensus \- Flagship.FYI, accessed July 22, 2025, [https://flagship.fyi/outposts/blockchains/unveiling-the-high-throughput-of-hedera-hashgraph-a-deep-dive-into-hashgraph-consensus/](https://flagship.fyi/outposts/blockchains/unveiling-the-high-throughput-of-hedera-hashgraph-a-deep-dive-into-hashgraph-consensus/)  
9. Hedera Hashgraph, accessed July 22, 2025, [https://hedera.com/learning/hedera-hashgraph](https://hedera.com/learning/hedera-hashgraph)  
10. Hedera Hashgraph (HBAR) explained: A beginner's guide \- Cointelegraph, accessed July 22, 2025, [https://cointelegraph.com/learn/articles/hedera-hashgraph-hbar](https://cointelegraph.com/learn/articles/hedera-hashgraph-hbar)  
11. Hedera was founded by Dr. Leemon Baird & Mance Harmon. The hashgraph's underlying technology has solved the “blockchain trilemma” combining high-throughput, low predictable fees, and finality in seconds. \- Reddit, accessed July 22, 2025, [https://www.reddit.com/r/Hedera/comments/12hhi2u/hedera\_was\_founded\_by\_dr\_leemon\_baird\_mance/](https://www.reddit.com/r/Hedera/comments/12hhi2u/hedera_was_founded_by_dr_leemon_baird_mance/)  
12. DeFi on Hedera, accessed July 22, 2025, [https://hedera.com/use-cases/defi](https://hedera.com/use-cases/defi)  
13. Hedera's Journey | Hedera, accessed July 22, 2025, [https://hedera.com/journey](https://hedera.com/journey)  
14. 96: Mance Harmon, co-founder and CEO of Hedera Hashgraph \- Jay Kim Show, accessed July 22, 2025, [https://jaykimshow.com/podcast/96-mance-harmon/](https://jaykimshow.com/podcast/96-mance-harmon/)  
15. Hedera: Hello future, accessed July 22, 2025, [https://hedera.com/](https://hedera.com/)  
16. About Us | Hashgraph, accessed July 22, 2025, [https://www.hashgraph.com/about/](https://www.hashgraph.com/about/)  
17. Hedera Council: Pioneering Best-In-Class Governance, accessed July 22, 2025, [https://hederacouncil.org/](https://hederacouncil.org/)  
18. COUNCIL OVERVIEW \- Hedera, accessed July 22, 2025, [https://hedera.com/Hedera\_COUNCIL-OVERVIEW\_SEPTEMBER\_2021.pdf](https://hedera.com/Hedera_COUNCIL-OVERVIEW_SEPTEMBER_2021.pdf)  
19. How to Look Up Transaction History on Hedera Using Mirror, accessed July 22, 2025, [https://hedera.com/blog/how-to-look-up-transaction-history-on-hedera-using-mirror-nodes-back-to-the-basics](https://hedera.com/blog/how-to-look-up-transaction-history-on-hedera-using-mirror-nodes-back-to-the-basics)  
20. COUNCIL OVERVIEW \- Hedera, accessed July 22, 2025, [https://hedera.com/Hedera\_COUNCIL-OVERVIEW](https://hedera.com/Hedera_COUNCIL-OVERVIEW)  
21. What is the Hedera Governing Council & what are the roles of these HBAR council members? \- Reddit, accessed July 22, 2025, [https://www.reddit.com/r/Hedera/comments/138ub32/what\_is\_the\_hedera\_governing\_council\_what\_are\_the/](https://www.reddit.com/r/Hedera/comments/138ub32/what_is_the_hedera_governing_council_what_are_the/)  
22. What are the responsibilities of Council members? \- Hedera Help, accessed July 22, 2025, [https://hedera.zendesk.com/hc/en-us/articles/360007276378-What-are-the-responsibilities-of-Council-members](https://hedera.zendesk.com/hc/en-us/articles/360007276378-What-are-the-responsibilities-of-Council-members)  
23. HBAR (ℏ) | Hedera, accessed July 22, 2025, [https://hedera.com/hbar](https://hedera.com/hbar)  
24. Hedera Network Performance and Token Economics, accessed July 22, 2025, [https://hedera.com/blog/hederas-token-economics](https://hedera.com/blog/hederas-token-economics)  
25. Hedera Processes Thousands of Transactions Per Second… See, accessed July 22, 2025, [https://hedera.com/blog/hedera-processes-thousands-of-tps-see-how-that-number-is-calculated](https://hedera.com/blog/hedera-processes-thousands-of-tps-see-how-that-number-is-calculated)  
26. Hedera \[TPS, Max TPS, Block Time & TTF\] | Chainspect, accessed July 22, 2025, [https://chainspect.app/chain/hedera](https://chainspect.app/chain/hedera)  
27. hashport | Hedera, accessed July 22, 2025, [https://hedera.com/users/hashport](https://hedera.com/users/hashport)  
28. What is cryptocurrency Hedera (HBAR) Hashgraph and how does it work? \- Kriptomat, accessed July 22, 2025, [https://kriptomat.io/cryptocurrency-prices/hedera-hashgraph-hbar-price/what-is/](https://kriptomat.io/cryptocurrency-prices/hedera-hashgraph-hbar-price/what-is/)  
29. USDC \- Hedera, accessed July 22, 2025, [https://hedera.com/users/usdc](https://hedera.com/users/usdc)  
30. Stablecoin Studio on Hedera, accessed July 22, 2025, [https://hedera.com/stablecoin-studio](https://hedera.com/stablecoin-studio)  
31. NFTs on Hedera, accessed July 22, 2025, [https://hedera.com/use-cases/nfts](https://hedera.com/use-cases/nfts)  
32. HashAxis \- Hedera, accessed July 22, 2025, [https://hedera.com/users/hashaxis](https://hedera.com/users/hashaxis)  
33. Top 10 Projects Driving Innovation in the Hedera Hashgraph Ecosystem \- Bitrue FAQ, accessed July 22, 2025, [https://support.bitrue.com/hc/en-001/articles/40957208695833-Top-10-Projects-Driving-Innovation-in-the-Hedera-Hashgraph-Ecosystem](https://support.bitrue.com/hc/en-001/articles/40957208695833-Top-10-Projects-Driving-Innovation-in-the-Hedera-Hashgraph-Ecosystem)  
34. Beginner's Guide to Hedera Hashgraph & HBAR \- ChangeHero, accessed July 22, 2025, [https://changehero.io/blog/hedera-hbar-guide/](https://changehero.io/blog/hedera-hbar-guide/)  
35. HBARUSD Charts and Quotes \- TradingView, accessed July 22, 2025, [https://www.tradingview.com/symbols/HBARUSD/](https://www.tradingview.com/symbols/HBARUSD/)  
36. Hedera Price: HBAR Live Price Chart, Market Cap & News Today | CoinGecko, accessed July 22, 2025, [https://www.coingecko.com/en/coins/hedera](https://www.coingecko.com/en/coins/hedera)  
37. Real-World Asset Tokenization \- Hedera, accessed July 22, 2025, [https://hedera.com/use-cases/real-world-asset-tokenization](https://hedera.com/use-cases/real-world-asset-tokenization)  
38. Top 10 Hedera Hashgraph dApps in 2024 You Need to Know \- Rejolut, accessed July 22, 2025, [https://rejolut.com/blog/top-10-hedera-hashgraph-dapps/](https://rejolut.com/blog/top-10-hedera-hashgraph-dapps/)  
39. SDKs | Hedera, accessed July 22, 2025, [https://docs.hedera.com/hedera/sdks-and-apis/sdks](https://docs.hedera.com/hedera/sdks-and-apis/sdks)  
40. Java SDK \- Hedera, accessed July 22, 2025, [https://hedera.com/open-source/project/java-sdk](https://hedera.com/open-source/project/java-sdk)  
41. sdk \- com.hedera.hashgraph \- Maven Repository, accessed July 22, 2025, [https://mvnrepository.com/artifact/com.hedera.hashgraph/sdk](https://mvnrepository.com/artifact/com.hedera.hashgraph/sdk)  
42. @hashgraph/sdk \- npm, accessed July 22, 2025, [https://www.npmjs.com/package/@hashgraph/sdk](https://www.npmjs.com/package/@hashgraph/sdk)  
43. Go SDK \- Hedera, accessed July 22, 2025, [https://hedera.com/open-source/project/go-sdk](https://hedera.com/open-source/project/go-sdk)  
44. Build Your Hedera Client, accessed July 22, 2025, [https://docs.hedera.com/hedera/sdks-and-apis/sdks/client](https://docs.hedera.com/hedera/sdks-and-apis/sdks/client)  
45. How to Start Developing on Hedera: Back to the Basics \- YouTube, accessed July 22, 2025, [https://www.youtube.com/watch?v=Skx6b8uK9ks](https://www.youtube.com/watch?v=Skx6b8uK9ks)  
46. Hedera Hashgraph | Online Course \- Udacity, accessed July 22, 2025, [https://www.udacity.com/course/hedera-hashgraph--cd13594](https://www.udacity.com/course/hedera-hashgraph--cd13594)  
47. SaucerSwap: The Pioneering DEX on Hedera, accessed July 22, 2025, [https://hedera.com/users/saucerswap](https://hedera.com/users/saucerswap)  
48. SaucerSwap \- DeFi Overview, TVL Analysis \- DappRadar, accessed July 22, 2025, [https://dappradar.com/dapp/saucerswap](https://dappradar.com/dapp/saucerswap)  
49. SaucerSwap | Leading Crypto Trading Protocol on Hedera \- Fast ..., accessed July 22, 2025, [https://www.saucerswap.finance/](https://www.saucerswap.finance/)  
50. SaucerSwap: What is it and Why is it Important? \- GetBlock.io, accessed July 22, 2025, [https://getblock.io/marketplace/projects/sauserswap/](https://getblock.io/marketplace/projects/sauserswap/)  
51. FAQ \- hashport, accessed July 22, 2025, [https://www.hashport.network/faq/](https://www.hashport.network/faq/)  
52. hashport – Worlds of possibility., accessed July 22, 2025, [https://www.hashport.network/](https://www.hashport.network/)  
53. What is Stader Labs? Liquid Staking Platform for Bigger Rewards \- Nansen, accessed July 22, 2025, [https://www.nansen.ai/post/what-is-stader-labs-liquid-staking-platform-for-bigger-rewards](https://www.nansen.ai/post/what-is-stader-labs-liquid-staking-platform-for-bigger-rewards)  
54. Liquid Staking \- How Staking HBAR Can Earn You staking rewards, accessed July 22, 2025, [https://www.staderlabs.com/hedera/](https://www.staderlabs.com/hedera/)  
55. Building liquidity across chains \- Stader Labs, accessed July 22, 2025, [https://www.staderlabs.com/blogs/stader-labs-building-liquidity-across-chains/](https://www.staderlabs.com/blogs/stader-labs-building-liquidity-across-chains/)  
56. SaucerSwap WHBAR Migration Announcement \- Medium, accessed July 22, 2025, [https://medium.com/@SaucerSwap/saucerswap-whbar-migration-announcement-295e04b01d5e](https://medium.com/@SaucerSwap/saucerswap-whbar-migration-announcement-295e04b01d5e)  
57. Zuse Market \- Project Overview, Analytics, and Data \- DappRadar, accessed July 22, 2025, [https://dappradar.com/dapp/zuse-market](https://dappradar.com/dapp/zuse-market)  
58. HashPack 2022 Q1 review, accessed July 22, 2025, [https://www.hashpack.app/post/hashpack-2022-q1-review](https://www.hashpack.app/post/hashpack-2022-q1-review)  
59. Topic: Calaxy \- Flagship.FYI, accessed July 22, 2025, [https://flagship.fyi/topic/calaxy/](https://flagship.fyi/topic/calaxy/)  
60. Welcome to Calaxy\!, accessed July 22, 2025, [https://help.calaxy.com/en/articles/7339529-welcome-to-calaxy](https://help.calaxy.com/en/articles/7339529-welcome-to-calaxy)  
61. Calaxy | Hedera, accessed July 22, 2025, [https://hedera.com/users/calaxy](https://hedera.com/users/calaxy)  
62. Calaxy social marketplace launches app built on Hedera \- The Paypers, accessed July 22, 2025, [https://thepaypers.com/payments/news/calaxy-social-marketplace-launches-app-built-on-hedera](https://thepaypers.com/payments/news/calaxy-social-marketplace-launches-app-built-on-hedera)  
63. Social Marketplace Calaxy Launches To Revolutionize Creator Monetization in $250 Billion Creator Economy \- PR Newswire, accessed July 22, 2025, [https://www.prnewswire.com/news-releases/social-marketplace-calaxy-launches-to-revolutionize-creator-monetization-in-250-billion-creator-economy-301898356.html](https://www.prnewswire.com/news-releases/social-marketplace-calaxy-launches-to-revolutionize-creator-monetization-in-250-billion-creator-economy-301898356.html)  
64. Wallet FAQ \- Calaxy Help Center, accessed July 22, 2025, [https://help.calaxy.com/en/articles/8168542-wallet-faq](https://help.calaxy.com/en/articles/8168542-wallet-faq)  
65. HashPack announces Calaxy Token ($CLXY) support, accessed July 22, 2025, [https://www.hashpack.app/post/hashpack-announces-calaxy-token-clxy-support](https://www.hashpack.app/post/hashpack-announces-calaxy-token-clxy-support)  
66. HeadStarter \- Hedera, accessed July 22, 2025, [https://hedera.com/users/headstarter](https://hedera.com/users/headstarter)  
67. HeadStarter | The Launchpad of the Hedera Hashgraph ecosystem, accessed July 22, 2025, [https://headstarter.org/](https://headstarter.org/)  
68. thoughts on HeadStarter? : r/Hedera \- Reddit, accessed July 22, 2025, [https://www.reddit.com/r/Hedera/comments/v7vz96/thoughts\_on\_headstarter/](https://www.reddit.com/r/Hedera/comments/v7vz96/thoughts_on_headstarter/)  
69. Headstarter Discussion : r/Hedera \- Reddit, accessed July 22, 2025, [https://www.reddit.com/r/Hedera/comments/ud9no7/headstarter\_discussion/](https://www.reddit.com/r/Hedera/comments/ud9no7/headstarter_discussion/)  
70. Diamond Standard | Hedera, accessed July 22, 2025, [https://hedera.com/users/diamond-standard](https://hedera.com/users/diamond-standard)  
71. Topic: Bitcarbon \- Flagship.FYI, accessed July 22, 2025, [https://flagship.fyi/topic/bitcarbon/](https://flagship.fyi/topic/bitcarbon/)  
72. Commodity Creation \- Diamond Standard, accessed July 22, 2025, [https://www.diamondstandard.co/commodity-creation](https://www.diamondstandard.co/commodity-creation)  
73. The Diamond Standard & Hedera \- Reddit, accessed July 22, 2025, [https://www.reddit.com/r/Hedera/comments/128xeh1/the\_diamond\_standard\_hedera/](https://www.reddit.com/r/Hedera/comments/128xeh1/the_diamond_standard_hedera/)  
74. HeadStarter Launchpad Will Accelerate Development of Applications Built on the Hedera Network \- GlobeNewswire, accessed July 22, 2025, [https://www.globenewswire.com/news-release/2022/04/01/2414999/0/en/HeadStarter-Launchpad-Will-Accelerate-Development-of-Applications-Built-on-the-Hedera-Network.html](https://www.globenewswire.com/news-release/2022/04/01/2414999/0/en/HeadStarter-Launchpad-Will-Accelerate-Development-of-Applications-Built-on-the-Hedera-Network.html)  
75. Origin Launchpad Emerges From Stealth To Deliver Expertly Curated Investments on Hedera Network \- Business Wire, accessed July 22, 2025, [https://www.businesswire.com/news/home/20250220542051/en/Origin-Launchpad-Emerges-From-Stealth-To-Deliver-Expertly-Curated-Investments-on-Hedera-Network](https://www.businesswire.com/news/home/20250220542051/en/Origin-Launchpad-Emerges-From-Stealth-To-Deliver-Expertly-Curated-Investments-on-Hedera-Network)  
76. Circle Expands USDC Multichain Ecosystem with Support on the Hedera Network, accessed July 22, 2025, [https://www.prnewswire.com/news-releases/circle-expands-usdc-multichain-ecosystem-with-support-on-the-hedera-network-301401698.html](https://www.prnewswire.com/news-releases/circle-expands-usdc-multichain-ecosystem-with-support-on-the-hedera-network-301401698.html)  
77. Who's behind Zuse Market? : r/Hedera \- Reddit, accessed July 22, 2025, [https://www.reddit.com/r/Hedera/comments/vynw5o/whos\_behind\_zuse\_market/](https://www.reddit.com/r/Hedera/comments/vynw5o/whos_behind_zuse_market/)  
78. Circle Expands USDC Multichain Ecosystem with Support on the Hedera Network, accessed July 22, 2025, [https://ffnews.com/newsarticle/circle-expands-usdc-multichain-ecosystem-with-support-on-the-hedera-network/](https://ffnews.com/newsarticle/circle-expands-usdc-multichain-ecosystem-with-support-on-the-hedera-network/)  
79. USDC | Powering global finance. Issued by Circle., accessed July 22, 2025, [https://www.circle.com/usdc](https://www.circle.com/usdc)  
80. Powering Compliant Stablecoins on Hedera \- YouTube, accessed July 22, 2025, [https://www.youtube.com/watch?v=1VrFazcB\_rk](https://www.youtube.com/watch?v=1VrFazcB_rk)  
81. Storing my USDC on Hashgraph. but how? : r/Hedera \- Reddit, accessed July 22, 2025, [https://www.reddit.com/r/Hedera/comments/18od5hx/storing\_my\_usdc\_on\_hashgraph\_but\_how/](https://www.reddit.com/r/Hedera/comments/18od5hx/storing_my_usdc_on_hashgraph_but_how/)  
82. www.usdc.com, accessed July 22, 2025, [https://www.usdc.com/learn/how-to-get-usdc-on-hedera\#:\~:text=Use%20a%20decentralized%20crypto%20exchange,to%20access%20a%20decentralized%20exchange.](https://www.usdc.com/learn/how-to-get-usdc-on-hedera#:~:text=Use%20a%20decentralized%20crypto%20exchange,to%20access%20a%20decentralized%20exchange.)  
83. How to fund my Hedera Wallet with HBAR? | by HeliSwap \- Medium, accessed July 22, 2025, [https://medium.com/@heliswap/how-to-fund-my-hashpack-wallet-with-hbar-24d3f59f0af](https://medium.com/@heliswap/how-to-fund-my-hashpack-wallet-with-hbar-24d3f59f0af)  
84. How to buy HBAR and USDC in HashPack using Banxa, accessed July 22, 2025, [https://www.hashpack.app/post/how-to-buy-hbar-and-usdc-in-hashpack-using-banxa](https://www.hashpack.app/post/how-to-buy-hbar-and-usdc-in-hashpack-using-banxa)