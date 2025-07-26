

# **A Comprehensive Architectural and Practical Guide to eBPF Development with Aya**

## **Introduction: The Confluence of Rust and eBPF**

The Linux kernel, the core of modern computing infrastructure, has historically been a challenging environment for innovation. Extending its functionality required either a lengthy and complex process of upstreaming code changes or the development of loadable kernel modules (LKMs), a path fraught with risks of system instability and security vulnerabilities.1 A revolutionary technology, eBPF (extended Berkeley Packet Filter), has fundamentally altered this landscape.

### **A Primer on eBPF: Safe, High-Performance In-Kernel Programmability**

eBPF is a technology that enables the execution of sandboxed, user-supplied programs within the privileged context of the operating system kernel.2 It achieves this without necessitating changes to the kernel's source code or the loading of potentially unstable kernel modules.1 Originating from the classic Berkeley Packet Filter (BPF) designed in 1992 for network packet filtering, eBPF, which appeared in Linux kernel 3.18 in 2014, has expanded this capability to a vast array of kernel subsystems.3

The architecture of eBPF is built on several key principles to ensure safety and performance:

* **Event-Driven Execution**: eBPF programs are not free-running; they are attached to specific "hooks" within the kernel and execute only when those events occur. These hooks include system calls, function entries and exits, network events, and kernel tracepoints.2  
* **The Verifier**: Before any eBPF bytecode is loaded, it undergoes a rigorous static analysis by the kernel's verifier. This component acts as a gatekeeper, ensuring the program is safe to run. It checks for out-of-bounds memory access, invalid register states, and, crucially, guarantees that the program will always run to completion by rejecting any code with unbounded loops that could lock up the kernel.1  
* **Just-in-Time (JIT) Compilation**: Once verified, the generic eBPF bytecode is translated into native machine code by a JIT compiler. This process dramatically improves execution speed, allowing eBPF programs to run with minimal overhead, approaching the performance of natively compiled kernel code.2

This combination of safety, performance, and dynamic programmability has led to eBPF being adopted for a wide range of applications, including networking, observability, tracing, and security.1

### **Introducing Aya: A Modern, Pure-Rust eBPF Framework**

While eBPF is powerful, the traditional development workflow, centered around the C programming language and complex toolchains like BCC (BPF Compiler Collection) or libbpf, presents a significant barrier to entry. It often requires managing kernel header dependencies, intricate build setups, and writing code in two different languages (e.g., C for the kernel, Python or Go for userspace).5

Aya emerges as a modern solution to these challenges, offering a library for building eBPF applications entirely in Rust.3 It is built from the ground up as a pure-Rust implementation, relying only on the

libc crate for making the necessary system calls to interact with the kernel's eBPF subsystem.7 This self-contained nature is a foundational design choice that distinguishes it from other Rust eBPF libraries that act as wrappers around the C-based

libbpf.6

### **The Aya Philosophy: Developer Experience and Compile-Once, Run-Everywhere (CO-RE)**

The core philosophy of Aya is a relentless focus on developer experience and operability.7 This principle informs every aspect of its design. The complexity of traditional eBPF development, with its reliance on C and intricate build systems, has historically limited its accessibility. Aya's primary innovation is not merely technical but strategic: it posits that by drastically improving the developer experience, it can unlock the power of eBPF for a much broader audience of developers.

This strategic bet is evident in its key features. The framework eliminates the need for a C toolchain, kernel headers, and complex build configurations, resulting in remarkably fast build times.7 It provides first-class support for asynchronous programming with popular runtimes like

tokio and async-std, allowing seamless integration into modern Rust applications.7 The use of a project scaffolding system via

cargo-generate further streamlines the initial setup, allowing developers to focus on application logic rather than boilerplate.11

Central to this philosophy is the robust implementation of **Compile-Once, Run-Everywhere (CO-RE)**. By leveraging the BPF Type Format (BTF), a metadata format that describes data types within the kernel, Aya enables the creation of eBPF programs that are portable across different Linux kernel versions.7 A single, self-contained binary can be compiled on a developer's machine and deployed to a wide range of target systems without recompilation, a significant operational advantage that directly addresses a major pain point of the traditional eBPF workflow.10 All of these architectural decisions are subservient to the primary goal of making eBPF development more productive, reliable, and accessible within the rich Rust ecosystem.

## **Section 1: The Architecture of the Aya Ecosystem**

Aya is not a single monolithic library but a well-structured ecosystem of crates, each with a distinct role. This modular architecture reflects the inherent duality of eBPF applications and is designed to provide a clean separation of concerns while maintaining a cohesive development experience within the Cargo ecosystem.

### **The Dual-Component Model: Userspace Orchestration and Kernel-Side Execution**

At its core, every eBPF application consists of two distinct parts that work in concert 3:

1. **The eBPF Program**: This is the code that runs inside the Linux kernel. It is compiled into eBPF bytecode and is subject to the strict constraints of the eBPF virtual machine, such as a limited stack size (512 bytes), no heap access, and no ability to call arbitrary kernel functions.14 This component reacts to kernel events and performs the low-level work of packet filtering, tracing, or security enforcement.  
2. **The Userspace Program**: This is a standard application that acts as the loader and controller for the eBPF program. Its responsibilities include loading the eBPF bytecode into the kernel, attaching it to the desired hooks, creating and manipulating eBPF maps to share data, and receiving events from the kernel-side program.4

Aya elegantly supports this model by enabling both components to be written in Rust, typically organized within a single Cargo workspace. This allows for seamless code sharing, particularly for data structures used in communication, which enhances type safety and maintainability.16

### **Dissecting the Core Crates: A Component-by-Component Analysis**

The aya-rs ecosystem is composed of several key crates that map directly to the dual-component model and the build process.18

#### **aya**

The aya crate is the userspace "conductor" of the orchestra. It is the primary library used in the userspace application to manage the entire lifecycle of eBPF programs and their associated resources.7 Its key responsibilities include:

* **Loading eBPF Code**: It provides the main Ebpf struct, which can load eBPF object files either from the filesystem (Ebpf::load\_file) or directly from a byte slice in memory (Ebpf::load).7  
* **Program Management**: It allows the userspace code to get handles to specific programs within the loaded object file, load them into the kernel, and attach them to kernel hooks (e.g., network interfaces, kprobes, tracepoints).9  
* **Map Interaction**: It provides a high-level, typed API for interacting with eBPF maps defined in the kernel-side code. Userspace can use aya to read from, write to, and iterate over maps.20  
* **BTF and Relocations**: It handles the complexities of using BPF Type Format (BTF) for CO-RE and processes relocations within the eBPF object file, making function calls and access to global data possible.7

#### **aya-ebpf**

This crate is the counterpart to aya and is used exclusively for writing the kernel-side eBPF programs.18 It provides the necessary tools to write safe and correct Rust code that can be compiled into eBPF bytecode. Its features include:

* **eBPF-Safe Environment**: It is a no\_std library, meaning it does not depend on the Rust standard library, which is a requirement for the constrained eBPF environment.11  
* **Program Macros**: It offers a suite of attribute macros like \#\[xdp\], \#\[kprobe\], \#\[classifier\], etc., which designate a Rust function as a specific type of eBPF program and handle the necessary boilerplate.21  
* **Context Types**: It provides strongly-typed context objects (e.g., XdpContext, ProbeContext) that give the eBPF program safe access to the data associated with the event it is handling.11  
* **Map and Helper APIs**: It includes APIs for defining and interacting with eBPF maps from within the kernel code and provides safe wrappers around eBPF helper functions (kernel-provided functions that eBPF programs are allowed to call).21

#### **aya-log & aya-log-ebpf**

This pair of crates provides a sophisticated logging solution that bridges the kernel-userspace divide.

* **aya-log-ebpf**: Used within the eBPF program, it provides logging macros like info\!, warn\!, and error\! that emulate the standard Rust log crate's interface.11  
* **aya-log**: The userspace component that initializes the logging infrastructure. It works by reading log data from a dedicated eBPF map populated by aya-log-ebpf and forwarding it to any logging facade compatible with the log crate, such as env\_logger.16

#### **aya-obj**

This is a lower-level library responsible for parsing eBPF ELF object files.26 It understands the specific ELF section conventions used for eBPF, supports BTF parsing, and handles relocations. While

aya uses aya-obj internally, most application developers will not need to use this crate directly. Its primary audience is developers building low-level eBPF tooling.18

#### **aya-build & bpf-linker**

These are essential components of the build toolchain.

* **aya-build**: A helper crate that provides build-time support for Aya projects.18  
* **bpf-linker**: A critical tool that links the object files produced by rustc into a final, valid eBPF ELF object file that the kernel can accept. It is a required dependency for any Aya project and must be installed separately via cargo install.12

### **The Development Workflow: From cargo-generate to a Deployed Program**

The Aya ecosystem promotes a standardized and streamlined development workflow, abstracting away much of the underlying complexity.

1. **Project Scaffolding**: The process typically begins with cargo-generate, a tool for creating new projects from templates. The official aya-template prompts the user for the project name and the desired eBPF program type, then generates a complete Cargo workspace with a sensible directory structure: one crate for the userspace code, one for the eBPF code, and a common crate for shared data structures.11  
2. **Build Process**: Compilation is a two-stage process orchestrated by a custom build system. The aya-template includes a project-specific xtask crate, which is a convention for creating build scripts in Rust.  
   * cargo xtask build-ebpf: This command compiles the eBPF crate (e.g., myapp-ebpf) for the bpfel-unknown-none target, producing the eBPF ELF object file.  
   * cargo build: This command compiles the userspace crate (e.g., myapp), which will embed or load the eBPF object file produced in the previous step.  
     This two-step process is necessary because the eBPF code and userspace code are compiled for different targets.27  
3. **Running the Application**: The xtask crate also provides a run command. Executing cargo xtask run builds both components if necessary and then executes the final userspace binary with the required privileges (typically via sudo \-E) to load the eBPF program into the kernel.27

The use of the xtask pattern is a deliberate and insightful design choice. eBPF development is inherently more complex than a standard cargo run workflow; it involves cross-compilation, managing file paths to object files, and handling elevated permissions. By encapsulating these steps into a project-local Rust crate (xtask), Aya provides a simple, high-level interface (cargo xtask...) that feels native to the Cargo ecosystem. This avoids the need for external build systems like Makefiles and keeps the entire development process within the familiar and powerful confines of Rust tooling, perfectly aligning with the overarching goal of maximizing developer experience.27

## **Section 2: Mastering eBPF Program Types with Aya**

eBPF's versatility stems from its ability to attach to a wide variety of hooks throughout the Linux kernel. Aya provides high-level abstractions for many of these attachment points, categorizing them into intuitive program types that align with common use cases. These can be broadly grouped into three domains: networking, tracing, and security.3

### **A Taxonomy of eBPF Programs: Networking, Tracing, and Security**

Aya empowers developers to build applications across the full spectrum of eBPF capabilities:

* **Networking**: This is the original domain of BPF and remains one of the most powerful use cases for eBPF. Applications include high-performance load balancers, sophisticated firewalls, and network monitoring tools like Cilium.3  
* **Observability and Tracing**: eBPF provides unprecedented visibility into the inner workings of the kernel and userspace applications. It can be used to build profilers, trace system calls, and monitor system performance with minimal overhead.3  
* **Security**: By hooking into security-sensitive kernel operations, eBPF can be used to enforce security policies, audit system behavior, and build runtime threat detection tools like Falco or Tetragon.3

### **In-Depth Program Type Analysis and Use Cases**

Aya uses attribute macros to define the type of an eBPF program. This declarative approach simplifies development by handling the underlying ELF section naming and attachment logic automatically.

#### **High-Performance Networking**

* **XDP (eXpress Data Path)**: Marked with \#\[xdp\], these programs attach at the earliest possible point in the network stack: directly within the NIC driver.29 This allows for extremely fast packet processing before the kernel allocates significant memory (like an  
  sk\_buff). It is ideal for tasks requiring maximum throughput, such as DDoS mitigation, basic load balancing, and packet filtering. XDP programs return an action code, such as XDP\_PASS (allow packet to continue), XDP\_DROP (silently drop), or XDP\_REDIRECT (forward to another NIC or an AF\_XDP socket).30  
* **TC (Traffic Control) SchedClassifier**: Marked with \#\[classifier\], these programs attach to the kernel's Traffic Control subsystem, specifically to queuing disciplines (qdiscs).31 A key advantage over XDP is that TC classifiers can inspect and manipulate both ingress (incoming) and egress (outgoing) traffic.13 While they execute later in the stack than XDP, giving up a small amount of performance, their versatility makes them suitable for more complex traffic shaping, quality-of-service (QoS) policies, and stateful firewalls.  
* **CgroupSkb**: Marked with \#\[cgroup\_skb\], these programs attach to a Linux cgroup (version 2\) and are triggered by network traffic associated with any process within that group.7 This is the cornerstone of container-aware networking and security, allowing for fine-grained network policies to be applied on a per-container or per-pod basis.

#### **System Tracing & Observability**

* **Kprobes / Kretprobes**: Marked with \#\[kprobe\] and \#\[kretprobe\], these programs dynamically attach to the entry and exit points of almost any function in the kernel, respectively.21 They are incredibly powerful for debugging the kernel, analyzing performance bottlenecks, and tracing function arguments and return values. However, they can be fragile as the kernel functions they attach to may change between versions.  
* **Uprobes / Uretprobes**: The userspace counterparts to kprobes, marked with \#\[uprobe\] and \#\[uretprobe\]. They allow for the dynamic instrumentation of functions within userspace applications and libraries.34 A common use case is tracing encrypted traffic by hooking into functions within libraries like OpenSSL.35  
* **Tracepoints**: Marked with \#\[tracepoint\], these programs attach to stable, static instrumentation points deliberately placed in the kernel source code by kernel developers.11 They are often used for tracing high-level events like system calls (e.g.,  
  sys\_enter\_execve).11 Because they are part of the kernel's stable API, they are much more reliable across kernel updates than kprobes.4

#### **Modern Security Enforcement**

* **LSM (Linux Security Modules)**: Marked with \#\[lsm\], these programs attach to the LSM framework's hooks, which are interspersed at security-critical points throughout the kernel.10 This allows developers to build powerful mandatory access control (MAC) systems, similar in principle to SELinux or AppArmor, but with the flexibility and dynamic nature of eBPF. They can police operations related to files, tasks, and sockets.32

### **Table: Supported eBPF Program Types in Aya**

To aid developers in selecting the appropriate tool, the following table summarizes key eBPF program types available in Aya, their attachment points, and primary use cases. This consolidated view serves as a quick-reference guide, distilling information from various documentation sources into an actionable format. Choosing the correct program type is the first and most critical step in designing an effective eBPF application.

| Program Type (aya macro) | Kernel Hook Point | Primary Use Case | Key Sources |
| :---- | :---- | :---- | :---- |
| \#\[xdp\] | NIC Driver (Ingress) | High-speed packet filtering, DDoS mitigation, load balancing. | 22 |
| \#\[classifier\] | Traffic Control (TC) qdisc (Ingress/Egress) | Sophisticated packet manipulation, traffic shaping, egress filtering. | 13 |
| \#\[cgroup\_skb\] | Cgroup v2 | Network policy enforcement per container/process group. | 7 |
| \#\[kprobe\] / \#\[kretprobe\] | Kernel function entry/exit | Dynamic kernel tracing, debugging, performance analysis. | 21 |
| \#\[uprobe\] / \#\[uretprobe\] | Userspace function entry/exit | Tracing libraries (e.g., OpenSSL), application profiling. | 34 |
| \#\[tracepoint\] | Static kernel tracepoints (e.g., syscalls) | Stable system call tracing, process execution monitoring. | 11 |
| \#\[lsm\] | Linux Security Module hooks | Mandatory Access Control (MAC), system hardening. | 10 |

## **Section 3: The eBPF Program Lifecycle: Loading and Management**

Understanding how an Aya program transitions from Rust source code to executable kernel code is fundamental to effective eBPF development. This lifecycle involves compilation, verification by the kernel, and a flexible loading process managed by the aya userspace library.

### **From Rust to Kernel: The Compilation Pipeline and eBPF Bytecode**

When an aya-ebpf crate is compiled, it undergoes a specialized build process. The Rust compiler (rustc), leveraging the nightly toolchain, compiles the Rust code into LLVM Intermediate Representation (IR). The bpf-linker tool then takes this output and links it into a standard ELF (Executable and Linkable Format) object file.12

The code within this ELF file is not native machine code for the host architecture (like x86-64). Instead, it is **eBPF bytecode**, a platform-agnostic instruction set designed specifically for the eBPF virtual machine that resides within the Linux kernel.1 This bytecode is what gets loaded into the kernel for execution.

One can inspect this generated bytecode using standard LLVM tools. For example, the command llvm-objdump \--section=xdp \-S \<path-to-object-file\> will disassemble the xdp section of an object file, revealing the low-level eBPF instructions.27 This can be an invaluable debugging tool for understanding exactly what the eBPF program is doing. For instance, a simple program that returns

XDP\_PASS (which has a value of 2\) might disassemble to:

0: r0 \= 2  
1: exit

This shows the program loading the value 2 into register r0 (the return value register) and then exiting.37

### **The Kernel's Gatekeeper: Navigating the eBPF Verifier**

Before the kernel accepts and JIT-compiles any eBPF bytecode, it subjects the program to a rigorous static analysis process performed by the **verifier**.2 The verifier's sole purpose is to ensure the program is safe and will not crash, corrupt, or lock up the kernel. It exhaustively traverses every possible execution path of the program, enforcing a strict set of rules 1:

* **Termination Guarantee**: The verifier must prove that the program will always terminate. It does this by disallowing unbounded loops. Any loop must have a constant, known upper bound that can be evaluated at load time.  
* **Memory Safety**: The verifier checks every memory access to ensure it is within the bounds of permitted memory regions, such as the packet data in an XdpContext or the 512-byte stack.  
* **Type Safety**: When BTF is used, the verifier can track pointer types, ensuring that pointers are used correctly and not dereferenced after being passed to certain helper functions that might invalidate them.38  
* **Function Call Restrictions**: eBPF programs cannot call arbitrary kernel functions. They are restricted to a stable set of "helper functions" provided by the kernel API.2

Developers using Aya will inevitably encounter verifier errors. A common example is exceeding the 512-byte stack limit. If a developer declares a large array on the stack, the verifier will reject the program with a message like Looks like the BPF stack limit is exceeded and suggest moving the data to an eBPF map.14 Understanding the verifier's role is key to writing successful eBPF programs.

### **Flexible Loading Mechanisms in Aya**

The aya userspace crate offers two primary methods for loading the eBPF object file, providing flexibility for different deployment scenarios.

#### **Loading from an Object File at Runtime**

The most straightforward method is to load the eBPF program from a standalone ELF object file on the filesystem.

Rust

use aya::Ebpf;

// Load the eBPF object file from the specified path at runtime.  
let mut ebpf \= Ebpf::load\_file("myapp-ebpf.o")?;

This approach, using Ebpf::load\_file(), is flexible as it allows the eBPF program to be updated without recompiling the userspace loader. However, it requires that the .o file be deployed alongside the userspace binary, which can add complexity to packaging and distribution.7

#### **Embedding and Loading from Memory at Compile-Time**

A more robust and commonly used approach in the Aya ecosystem is to embed the eBPF bytecode directly into the userspace binary at compile time. This creates a single, self-contained executable that is easy to deploy, perfectly aligning with the CO-RE philosophy.7 This is achieved using the

include\_bytes\_aligned\! macro in conjunction with Ebpf::load().

Rust

use aya::Ebpf;

// At compile time, include the eBPF object file's bytes directly into the  
// userspace binary. The \`aya::include\_bytes\_aligned\!\` macro ensures correct  
// memory alignment.  
let mut ebpf \= Ebpf::load(aya::include\_bytes\_aligned\!(  
    concat\!(env\!("OUT\_DIR"), "/myapp-ebpf")  
))?;

The xtask build system generated by aya-template is configured to place the compiled eBPF object file in a known location within the OUT\_DIR, making this pattern seamless.17

### **Loading from an In-Memory Byte Array**

The user query specifically asked how to load a program from a byte array. The Ebpf::load() method is designed for precisely this purpose, as it accepts a byte slice (&\[u8\]).19 The

include\_bytes\_aligned\! macro is simply a convenient way to produce such a slice from a file at compile time. If an application obtains the eBPF object file's bytes from another source at runtime (e.g., by downloading it from a central server or generating it dynamically), it can pass that byte slice directly to Ebpf::load().

This demonstrates that Ebpf::load\_file() is effectively a convenience wrapper that reads a file into a byte buffer and then calls the core Ebpf::load() logic.

Rust

use aya::Ebpf;  
use std::fs;

// Example: Manually reading a file into a byte vector at runtime.  
// This \`ebpf\_bytes\` could come from any source, such as a network request.  
let ebpf\_bytes \= fs::read("myapp-ebpf.o")?;

// Load the eBPF program directly from the in-memory byte slice.  
let mut bpf \= Ebpf::load(\&ebpf\_bytes)?;

//... proceed to get programs and maps from the \`bpf\` object.

This capability is crucial for advanced use cases where eBPF programs are managed and distributed dynamically, rather than being statically bundled with the userspace application.

## **Section 4: Bidirectional Communication: The Bridge Between Userspace and Kernel**

Effective communication between the userspace controller and the kernel-side eBPF program is the cornerstone of any non-trivial eBPF application. This bidirectional data flow is what allows for dynamic configuration, state sharing, and the exfiltration of events for analysis. In the eBPF architecture, this communication is mediated almost exclusively by a special set of data structures known as **eBPF maps**.

### **The Central Role of eBPF Maps as the Shared State Substrate**

eBPF maps are generic, persistent key-value stores that reside in kernel memory and are accessible from both the eBPF program and the userspace application.4 When a userspace program loads an eBPF object file using

aya, all the maps defined within that file are created and initialized in the kernel. The userspace program can then obtain file descriptors to these maps, allowing it to interact with them via system calls.20

Maps serve several critical purposes:

* **Sharing State**: They allow the userspace application to pass configuration data down to the eBPF program (e.g., a blocklist of IP addresses).14  
* **Collecting Data**: They enable the eBPF program to push statistics and events up to the userspace for aggregation and analysis (e.g., packet counters).39  
* **Inter-Program Communication**: They can be used to share data between different eBPF programs attached to different hooks.14  
* **Overcoming Stack Limitations**: Since the eBPF stack is limited to 512 bytes, maps can be used as a form of "heap" storage to handle larger data structures that would otherwise not fit on the stack.14

The pattern of defining shared data structures in a common Rust crate, which is then used by both the userspace and eBPF crates, elevates maps from simple key-value stores to a robust, type-safe API contract.17 When the userspace program writes a

struct to a map, the eBPF program can read it as the exact same struct. This compile-time guarantee, enforced by the Rust compiler and Cargo's workspace feature, prevents entire classes of runtime errors caused by mismatched data layouts, a common pitfall in more loosely-coupled systems. This makes the resulting application significantly more reliable and easier to maintain.

### **A Deep Dive into Map Types and Manipulation**

The Linux kernel provides many different map types, each optimized for a specific use case. Aya provides safe, typed wrappers for these, allowing developers to work with them in an idiomatic Rust style.20 Maps are typically defined in the eBPF code using the

\#\[map\] attribute macro.14

#### **Key-Value Stores**

* **HashMap**: The most common map type, implementing a standard hash table. It is used for general-purpose key-value storage and is ideal for implementing lookup tables, such as an IP address blocklist.23  
* **LruHashMap**: A specialized hash map that implements a "Least Recently Used" (LRU) eviction policy. When the map is full and a new element is inserted, the oldest (least recently accessed) element is automatically removed. This is highly effective for implementing caches.14

#### **High-Speed Arrays**

* **Array**: A simple, fixed-size array where the key is an index (u32). It offers fast, direct access to elements.14  
* **PerCpuArray**: A highly optimized version of the Array map designed for scenarios like collecting statistics. Each CPU core gets its own independent copy of the array.14 When an eBPF program running on a specific core updates an entry, it only accesses its local copy, completely avoiding the need for atomic operations or locks. This makes it the most performant way to implement counters. The userspace application can then read the values from all per-CPU arrays and aggregate them.14

#### **High-Throughput Event Streaming: PerfEventArray vs. RingBuf**

For sending streams of event data from the kernel to userspace, two primary mechanisms exist.

* **PerfEventArray**: This is the traditional mechanism, available since kernel 4.3.40 It is a special map type that holds an array of file descriptors, one for each CPU, corresponding to a per-CPU ring buffer in the kernel's perf subsystem.4 The eBPF program uses the  
  bpf\_perf\_event\_output() helper to send data, and the userspace application reads from these buffers. Aya provides both synchronous and asynchronous (AsyncPerfEventArray) APIs for this.40 It is highly versatile and widely supported, making it a reliable choice for handling large volumes of data like network packets or system call events.34  
* **RingBuf**: A more modern alternative introduced in kernel 5.8, the RingBuf is a single-producer, single-consumer (SPSC) lock-free ring buffer.41 It often provides superior performance and a simpler API compared to  
  PerfEventArray because it avoids the complexity of managing multiple per-CPU buffers. Data is reserved, written, and submitted in a single operation, reducing overhead.35

#### **Specialized Networking Maps**

* **SockMap / SockHash**: These maps are designed to hold references (struct sock \*) to TCP sockets. They are primarily used by BPF\_PROG\_TYPE\_SK\_MSG and BPF\_PROG\_TYPE\_SOCK\_OPS programs to redirect socket traffic and enforce socket-level policies.20  
* **DevMap / CpuMap**: These maps are used by XDP programs to perform packet redirection. DevMap holds an array of network device interface indexes, while CpuMap holds CPU indexes. The bpf\_redirect\_map() helper can use these maps to forward a packet to another NIC or to another CPU for processing.22  
* **XskMap (XDP Socket Map)**: This is a specialized map of type BPF\_MAP\_TYPE\_XSKMAP that is essential for AF\_XDP (Address Family \- eXpress Data Path). It holds references to userspace AF\_XDP sockets. An XDP program can use bpf\_redirect\_map() with an XskMap to redirect raw packets directly from the NIC driver to a specific userspace socket, enabling zero-copy, high-performance packet processing.45

### **Practical Guide: Manipulating Maps from Both Sides**

Interacting with maps involves distinct APIs on the eBPF and userspace sides.

#### **eBPF Side (aya-ebpf)**

In the kernel, map operations are often low-level and may require unsafe blocks, as they involve direct pointer manipulation.

Rust

// In eBPF code (e.g., myapp-ebpf/src/main.rs)  
use aya\_ebpf::{  
    macros::map,  
    maps::{HashMap, PerCpuArray},  
};

\#\[map\]  
static mut IP\_COUNTER: HashMap\<u32, u64\> \= HashMap::with\_max\_entries(1024, 0);

\#\[map\]  
static mut PACKET\_STATS: PerCpuArray\<u64\> \= PerCpuArray::with\_max\_entries(1, 0);

// Function to increment a counter for a given IP address  
fn count\_ip(ip: u32) {  
    unsafe {  
        // Get a pointer to the value for the key, or insert 0 if it doesn't exist.  
        let ptr \= IP\_COUNTER.get\_ptr\_mut\_or\_try\_insert(\&ip, &0).unwrap();  
        if\!ptr.is\_null() {  
            // Dereference the pointer and increment the value.  
            \*ptr \+= 1;  
        }  
    }  
}

// Function to increment a global packet counter using a PerCpuArray  
fn increment\_pkt\_count() {  
    unsafe {  
        // Get a mutable pointer to the first (and only) entry in the PerCpuArray.  
        // This access is lock-free as it's local to the current CPU.  
        if let Some(ptr) \= PACKET\_STATS.get\_ptr\_mut(0) {  
            \*ptr \+= 1;  
        }  
    }  
}

Here, get\_ptr\_mut\_or\_try\_insert provides a raw pointer to the value associated with a key, which must be dereferenced within an unsafe block.14 For

PerCpuArray, get\_ptr\_mut is safe from a concurrency perspective but still returns a raw pointer requiring unsafe to dereference.

#### **Userspace Side (aya)**

In userspace, aya provides a safe, high-level API. After loading the eBPF object, you obtain a typed handle to the map.

Rust

// In userspace code (e.g., myapp/src/main.rs)  
use aya::{  
    maps::{HashMap, PerCpuValues},  
    util::online\_cpus,  
};

// Assume \`bpf\` is an initialized \`aya::Ebpf\` object.  
// Get a typed handle to the IP\_COUNTER map.  
let mut ip\_counter: HashMap\<\_, u32, u64\> \=  
    HashMap::try\_from(bpf.map\_mut("IP\_COUNTER").unwrap())?;

// Insert a new value or update an existing one.  
let some\_ip: u32 \= 0x01010101; // 1.1.1.1  
ip\_counter.insert(some\_ip, 100, 0)?;

// Read a value from the map.  
if let Some(count) \= ip\_counter.get(\&some\_ip, 0)? {  
    println\!("Count for 1.1.1.1: {}", count);  
}

// Get a handle to the PACKET\_STATS map.  
let packet\_stats: aya::maps::PerCpuArray\<\_, u64\> \=  
    aya::maps::PerCpuArray::try\_from(bpf.map("PACKET\_STATS").unwrap())?;

// Read values from all CPUs and sum them up.  
let stats\_values: PerCpuValues\<u64\> \= packet\_stats.get(&0, 0)?;  
let total\_packets: u64 \= stats\_values.iter().sum();  
println\!("Total packets processed: {}", total\_packets);

This demonstrates how aya abstracts away the raw system calls, providing an idiomatic Rust interface for map manipulation from userspace.28

## **Section 5: Comprehensive Observability with aya-log**

Effective observability is crucial for debugging and monitoring any system, and eBPF programs are no exception. However, the constrained kernel environment presents unique challenges for logging; standard I/O operations like println\! are forbidden.11 The

aya-log ecosystem provides an elegant and powerful solution that seamlessly integrates eBPF program logs into the standard Rust logging infrastructure.

### **Architecting Logs: How aya-log Bridges the Kernel-Userspace Divide**

The aya-log system is not a special kernel feature but rather a clever abstraction built on the standard eBPF communication primitives previously discussed. It operates using a two-part architecture that leverages eBPF maps to transport log data from the kernel to userspace.16

1. **eBPF Side (aya-log-ebpf)**: This crate provides logging macros (info\!, warn\!, etc.) for use inside the eBPF program.25 When one of these macros is called, it serializes the log message, its arguments, and metadata (like log level) into a predefined format. It then outputs this serialized data as an event into a dedicated eBPF map, typically a  
   PerfEventArray or RingBuf, which is created implicitly by the framework.16  
2. **Userspace Side (aya-log)**: This crate contains the logic to read from that dedicated logging map. The EbpfLogger::init() function finds the log map in the loaded eBPF object, sets up listeners for it, and begins processing the incoming event data. It deserializes each event back into a log record and forwards it to the facade provided by the standard Rust log crate.24

This architecture is powerful because it decouples the eBPF program from the specifics of log presentation. The eBPF code simply emits structured events, and the userspace component handles the formatting and output.

### **Implementation: Instrumenting Code for Seamless Logging**

Integrating aya-log into a project is remarkably straightforward.

#### **eBPF Side Instrumentation**

In the eBPF program, a developer only needs to add the aya-log-ebpf dependency and use the provided macros. The context (ctx) from the eBPF program is passed to the macro to provide additional information.

Rust

// In myapp-ebpf/src/main.rs  
\#\!\[no\_std\]  
\#\!\[no\_main\]

use aya\_ebpf::{macros::xdp, programs::XdpContext};  
use aya\_log\_ebpf::info; // Import the info\! macro  
use network\_types::ip::Ipv4Hdr;

\#\[xdp\]  
pub fn log\_packets(ctx: XdpContext) \-\> u32 {  
    // Example: Log the source IP address of an incoming packet.  
    if let Ok(ipv4) \= ctx.load::\<Ipv4Hdr\>(EthHdr::LEN) {  
        // The first argument to the macro is the context.  
        info\!(\&ctx, "Received packet from source IP: {:i}", ipv4.src\_addr);  
    }  
    //... return an XDP action  
    xdp\_action::XDP\_PASS  
}

The {:i} format specifier is a convention used by aya-log to hint that an integer should be formatted as an IP address.21

#### **Userspace Side Initialization**

In the userspace application, only two lines of code are required to enable the logging pipeline.

Rust

// In myapp/src/main.rs  
use aya::Ebpf;  
use aya\_log::EbpfLogger;  
use log::info;

//... in your async main function...

let mut bpf \= Ebpf::load(aya::include\_bytes\_aligned\!(...))?;

// 1\. Initialize a standard logger facade (e.g., env\_logger).  
env\_logger::init();

// 2\. Initialize the EbpfLogger, linking the eBPF logs to the facade.  
if let Err(e) \= EbpfLogger::init(&mut bpf) {  
    // This can fail if the eBPF program has no log statements.  
    warn\!("Failed to initialize eBPF logger: {}", e);  
}

info\!("Waiting for eBPF logs...");  
//... wait for program to exit...

With this setup, any info\! call in the eBPF program will be captured, transported to userspace, and printed to the console by env\_logger.

### **Viewing the Output: Integrating with the Rust Ecosystem**

The most significant benefit of aya-log's design is its seamless integration with the broader Rust logging ecosystem. Because it forwards messages to the standard log crate, developers can:

* Control log verbosity using the standard RUST\_LOG environment variable (e.g., RUST\_LOG=info cargo xtask run).3  
* Use any compatible logging implementation (env\_logger, tracing-subscriber, etc.) to format logs, send them to files, or export them to centralized logging platforms without changing the eBPF code.

This makes debugging and monitoring Aya applications feel native and consistent with standard Rust development practices, further lowering the barrier to entry for eBPF.

## **Section 6: Advanced Networking with AF\_XDP: A Complete Implementation**

To demonstrate the full power of Aya as an orchestration framework for high-performance networking, this section provides a complete, practical implementation of a dynamic UDP packet steering system. The goal is to build an application that can redirect incoming UDP packets to different processing queues based on their destination port number. This mapping will be dynamically configurable from userspace at runtime. This architecture is a common pattern for building scalable network functions like custom load balancers or protocol gateways.

### **Use Case Introduction: High-Performance, Dynamic UDP Packet Steering**

The system will perform the following actions:

1. An XDP program will be attached to a network interface to inspect all incoming packets at the driver level.  
2. For UDP packets, the program will extract the destination port number.  
3. It will look up this port in a map to determine which hardware queue (and corresponding userspace process) should handle the packet.  
4. The packet will be redirected to an AF\_XDP socket associated with the target queue, enabling zero-copy transfer to userspace.  
5. A userspace application will be responsible for setting up the AF\_XDP sockets, managing the port-to-queue mapping dynamically, and processing the redirected packets.

### **Architectural Blueprint: Combining XDP, AF\_XDP, and Port-to-Queue Mapping**

The data flow is orchestrated through a combination of eBPF programs and maps:

1. **Packet Arrival**: A UDP packet arrives at the network interface card (NIC).  
2. **XDP Execution**: The attached XDP program begins execution.  
3. **Header Parsing**: The program parses the Ethernet, IP, and UDP headers to access the destination port.  
4. **Port-to-Queue Lookup**: The destination port is used as a key to look up a value in a BPF\_MAP\_TYPE\_HASH named PORT\_TO\_QUEUE\_MAP. This map returns the target NIC queue index.  
5. **Socket Lookup**: The retrieved queue index is then used as a key to look up a socket file descriptor in a BPF\_MAP\_TYPE\_XSKMAP named XSK\_MAP. This special map holds references to the AF\_XDP sockets created in userspace.  
6. **Redirection**: The XDP program calls the bpf\_redirect\_map() helper function, passing it the XSK\_MAP and the queue index. This action steers the packet directly to the memory buffer (UMEM) of the corresponding AF\_XDP socket, bypassing the kernel's main network stack.45  
7. **Userspace Consumption**: The Rust userspace application, which has set up and registered the sockets in the XSK\_MAP, receives the packet on the appropriate AF\_XDP socket and can begin processing it.

\!([https://i.imgur.com/g8vUe3z.png](https://i.imgur.com/g8vUe3z.png))

### **Part 1: The eBPF Program (C Implementation)**

For this highly specialized networking task that interfaces with low-level kernel structures and map types like XSKMAP, using C for the eBPF program provides a stable and well-documented foundation. It directly mirrors kernel examples and allows for a clear separation of concerns, letting this report focus on how Aya can orchestrate even non-Rust eBPF code.45

**File: xdp\_redirect\_kern.c**

C

\#**include** \<linux/bpf.h\>  
\#**include** \<bpf/bpf\_helpers.h\>  
\#**include** \<bpf/bpf\_endian.h\>

\#**include** \<linux/if\_ether.h\>  
\#**include** \<linux/ip.h\>  
\#**include** \<linux/udp.h\>

// Map to store the mapping from UDP destination port to a NIC queue index.  
// Key: u16 (port), Value: u32 (queue\_id)  
struct {  
    \_\_uint(type, BPF\_MAP\_TYPE\_HASH);  
    \_\_uint(max\_entries, 1024);  
    \_\_type(key, \_\_u16);  
    \_\_type(value, \_\_u32);  
} port\_to\_queue\_map SEC(".maps");

// Map to store AF\_XDP sockets (XSKs). The kernel uses this map to redirect packets.  
// Key: u32 (queue\_id), Value: u32 (socket fd)  
struct {  
    \_\_uint(type, BPF\_MAP\_TYPE\_XSKMAP);  
    \_\_uint(max\_entries, 64); // Max number of queues to support  
    \_\_type(key, \_\_u32);  
    \_\_type(value, \_\_u32);  
} xsk\_map SEC(".maps");

SEC("xdp")  
int xdp\_port\_redirect(struct xdp\_md \*ctx) {  
    void \*data\_end \= (void \*)(long)ctx-\>data\_end;  
    void \*data \= (void \*)(long)ctx-\>data;

    // Boundary check for Ethernet header  
    struct ethhdr \*eth \= data;  
    if ((void \*)eth \+ sizeof(\*eth) \> data\_end) {  
        return XDP\_PASS;  
    }

    // We only care about IPv4 packets  
    if (eth-\>h\_proto\!= bpf\_htons(ETH\_P\_IP)) {  
        return XDP\_PASS;  
    }

    // Boundary check for IP header  
    struct iphdr \*iph \= data \+ sizeof(\*eth);  
    if ((void \*)iph \+ sizeof(\*iph) \> data\_end) {  
        return XDP\_PASS;  
    }

    // We only care about UDP packets  
    if (iph-\>protocol\!= IPPROTO\_UDP) {  
        return XDP\_PASS;  
    }

    // Boundary check for UDP header  
    struct udphdr \*udph \= (void \*)iph \+ sizeof(\*iph);  
    if ((void \*)udph \+ sizeof(\*udph) \> data\_end) {  
        return XDP\_PASS;  
    }

    \_\_u16 dest\_port \= udph-\>dest;

    // Look up the queue\_id for the destination port  
    \_\_u32 \*queue\_id \= bpf\_map\_lookup\_elem(\&port\_to\_queue\_map, \&dest\_port);  
    if (\!queue\_id) {  
        // If no mapping exists for this port, let the packet proceed normally.  
        return XDP\_PASS;  
    }

    // Redirect the packet to the AF\_XDP socket associated with the queue\_id.  
    // The 0 flag means to drop the packet if the redirect fails.  
    return bpf\_redirect\_map(\&xsk\_map, \*queue\_id, 0);  
}

char LICENSE SEC("license") \= "GPL";

### **Part 2: The Userspace Orchestrator (Rust with Aya)**

This Rust application demonstrates Aya's power. It loads the compiled C eBPF object, sets up the complex AF\_XDP infrastructure, dynamically populates the maps to control the eBPF program's behavior, and processes the redirected packets.

**File: main.rs**

Rust

use anyhow::Context;  
use aya::maps::{HashMap, XskMap};  
use aya::programs::{Xdp, XdpFlags};  
use aya::{include\_bytes\_aligned, Bpf};  
use aya\_log::EbpfLogger;  
use clap::Parser;  
use log::{info, warn};  
use std::net::Ipv4Addr;  
use tokio::{signal, task};

use aya::programs::af\_xdp::{AfXdp, AfXdpError, Umem, UmemConfig};  
use bytes::Bytes;  
use core::mem;  
use std::convert::TryInto;

\#  
struct Opt {  
    \#\[clap(short, long, default\_value \= "eth0")\]  
    iface: String,

    \#\[clap(short, long, default\_value \= "0")\]  
    queue\_id: u32,  
}

\#\[tokio::main\]  
async fn main() \-\> Result\<(), anyhow::Error\> {  
    let opt \= Opt::parse();  
    env\_logger::init();

    // Compile the C eBPF code and include the bytes.  
    // This requires a build.rs script to compile the C code.  
    let mut bpf \= Bpf::load(include\_bytes\_aligned\!(  
        "../../target/bpfel-unknown-none/release/xdp\_redirect\_kern.o"  
    ))?;  
    if let Err(e) \= EbpfLogger::init(&mut bpf) {  
        warn\!("failed to initialize eBPF logger: {}", e);  
    }

    // Load and attach the XDP program.  
    let program: &mut Xdp \= bpf.program\_mut("xdp\_port\_redirect").unwrap().try\_into()?;  
    program.load()?;  
    program.attach(\&opt.iface, XdpFlags::default())  
       .context("failed to attach the XDP program")?;

    // \--- AF\_XDP Setup \---  
    // Create a UMEM (user memory) area to be shared with the kernel for zero-copy.  
    let umem\_config \= UmemConfig::new(4096, 2048.try\_into().unwrap(), 2048, 2048, 0).unwrap();  
    let (umem, mut fill\_queue, mut completion\_queue, mut frames) \=  
        Umem::new(umem\_config, 4096 \* 16, false).unwrap();

    // Create an AF\_XDP socket for queue 0\.  
    let mut socket0 \= AfXdp::new(\&opt.iface, 0, umem.fd().try\_clone()?).unwrap();  
    // Create an AF\_XDP socket for queue 1\.  
    let mut socket1 \= AfXdp::new(\&opt.iface, 1, umem.fd().try\_clone()?).unwrap();

    // \--- Map Population \---  
    // Get a handle to the XSK\_MAP and insert the AF\_XDP sockets.  
    let mut xsk\_map \= XskMap::try\_from(bpf.map\_mut("xsk\_map").unwrap())?;  
    xsk\_map.set(0, socket0.fd(), 0)?;  
    xsk\_map.set(1, socket1.fd(), 0)?;

    // Get a handle to the PORT\_TO\_QUEUE\_MAP and add dynamic mappings.  
    let mut port\_map: HashMap\<\_, u16, u32\> \=  
        HashMap::try\_from(bpf.map\_mut("port\_to\_queue\_map").unwrap())?;

    // Map UDP port 8080 to queue 0\.  
    port\_map.insert(u16::to\_be(8080), 0, 0)?;  
    info\!("Redirecting UDP port 8080 to queue 0");

    // Map UDP port 9090 to queue 1\.  
    port\_map.insert(u16::to\_be(9090), 1, 0)?;  
    info\!("Redirecting UDP port 9090 to queue 1");

    // \--- Asynchronous Packet Processing \---  
    // Spawn a task to process packets on queue 0\.  
    let mut rx0 \= socket0.rx\_queue().unwrap();  
    let mut tx0 \= socket0.tx\_queue().unwrap();  
    let task0 \= tokio::spawn(async move {  
        info\!("\[Queue 0\] Waiting for packets...");  
        loop {  
            let received \= rx0.recv(&mut frames\[..2\]).await.unwrap();  
            for frame in \&frames\[..received\] {  
                info\!("\[Queue 0\] Received packet of length: {}", frame.len());  
            }  
            // In a real app, you would process the packet and maybe send a response.  
            // For this example, we just return the frame to the fill queue.  
            fill\_queue.produce(\&frames\[..received\]);  
        }  
    });

    // Spawn a task to process packets on queue 1\.  
    let mut rx1 \= socket1.rx\_queue().unwrap();  
    let mut tx1 \= socket1.tx\_queue().unwrap();  
    let task1 \= tokio::spawn(async move {  
        info\!("\[Queue 1\] Waiting for packets...");  
        loop {  
            let received \= rx1.recv(&mut frames\[2..4\]).await.unwrap();  
            for frame in \&frames\[2..received+2\] {  
                info\!("\[Queue 1\] Received packet of length: {}", frame.len());  
            }  
            fill\_queue.produce(\&frames\[2..received+2\]);  
        }  
    });

    info\!("Waiting for Ctrl-C...");  
    signal::ctrl\_c().await?;  
    info\!("Exiting...");  
    task0.abort();  
    task1.abort();

    Ok(())  
}

**File: build.rs**

Rust

use std::process::Command;

fn main() {  
    // Compile the C eBPF code using clang.  
    let status \= Command::new("clang")  
       .args(&\[  
            "-I", "/usr/include",  
            "-O2",  
            "-target", "bpf",  
            "-c", "src/xdp\_redirect\_kern.c",  
            "-o", "target/bpfel-unknown-none/release/xdp\_redirect\_kern.o"  
        \])  
       .status()  
       .expect("failed to compile eBPF C code");  
    assert\!(status.success());

    println\!("cargo:rerun-if-changed=src/xdp\_redirect\_kern.c");  
}

### **Full Code Listings and a Step-by-Step Execution Walkthrough**

1. **Prerequisites**: Ensure you have clang, llvm, and the Rust nightly toolchain installed. You also need the kernel headers for the C includes.  
2. **Project Setup**: Create a new Rust binary project (cargo new afxdp\_redirector). Place main.rs, xdp\_redirect\_kern.c, and build.rs inside the src/ directory. Create target/bpfel-unknown-none/release/ directories.  
3. **Dependencies**: Add aya, aya-log, tokio, anyhow, clap, and bytes to your Cargo.toml.  
4. **Compile and Run**:  
   Bash  
   \# Run the build script and the Rust application with sudo  
   RUST\_LOG=info cargo run \-- \--iface \<your-nic\>

   Replace \<your-nic\> with the name of your network interface.  
5. **Testing**: From another machine, send UDP packets to the target machine's IP on ports 8080 and 9090\.  
   Bash  
   \# Send to port 8080  
   echo "hello queue 0" | nc \-u \<target-ip\> 8080

   \# Send to port 9090  
   echo "hello queue 1" | nc \-u \<target-ip\> 9090

6. **Observe Output**: The running Rust application will print log messages indicating which queue received which packet, demonstrating that the dynamic redirection is working as intended.

This example showcases the ultimate strength of Aya: it serves as a powerful, safe, and modern Rust-based control plane for managing even the most complex, low-level eBPF networking logic, regardless of whether that logic was written in Rust or C.

## **Conclusion and Future Directions**

The Aya framework represents a significant leap forward in the eBPF development landscape. By providing a pure-Rust, end-to-end solution, it addresses many of the historical pain points associated with eBPF programming, making this powerful kernel technology more accessible, reliable, and productive for a new generation of systems developers.

### **Summary of Aya's Architectural Strengths and Practical Advantages**

The analysis reveals several key strengths that define Aya's value proposition:

* **Superior Developer Experience**: From project scaffolding with cargo-generate to the encapsulated build process with xtask, Aya is meticulously designed to simplify the developer workflow. Its independence from libbpf and the C toolchain for pure-Rust projects dramatically lowers the barrier to entry.7  
* **Compile-Once, Run-Everywhere (CO-RE)**: Aya's first-class support for the BPF Type Format (BTF) is a cornerstone feature, enabling the creation of portable eBPF applications that can be deployed across various kernel versions without recompilation. This is a critical advantage for operability and maintainability in diverse production environments.7  
* **Safety and Reliability**: By leveraging Rust's powerful type system and ownership model, Aya brings memory safety and compile-time guarantees to eBPF development. The convention of using a common crate for shared data structures effectively creates a type-safe API contract between userspace and the kernel, preventing a wide range of potential runtime errors.16  
* **Seamless Integration**: The framework integrates smoothly with the existing Rust ecosystem. The aya-log crate funnels kernel-side logs directly into the standard log facade, and the support for async/await with tokio and async-std allows eBPF functionality to be incorporated into modern, high-performance asynchronous applications.7

### **The Evolving eBPF Landscape and Aya's Trajectory**

The eBPF ecosystem is one of the most rapidly evolving areas of Linux kernel development.10 New program types, helper functions, and map types are continuously being introduced. Aya has demonstrated a strong commitment to keeping pace with these advancements, incorporating support for modern features like

RingBuf and AF\_XDP.

The growing adoption of Aya in significant open-source projects, such as Red Hat's bpfman, the Kubernetes SIGs' blixt load balancer, and Deepfence's security stack, is a testament to its maturity and readiness for production use.8 As Rust continues to solidify its position as a premier language for systems programming, Aya is poised to become a dominant framework for building the next generation of networking, observability, and security tools powered by eBPF. Resources like the official Aya Book will remain essential for developers to stay current with this dynamic and powerful technology.10

#### **Works cited**

1. A Gentle Introduction to eBPF \- InfoQ, accessed June 27, 2025, [https://www.infoq.com/articles/gentle-linux-ebpf-introduction/](https://www.infoq.com/articles/gentle-linux-ebpf-introduction/)  
2. What is eBPF? An Introduction and Deep Dive into the eBPF Technology, accessed June 27, 2025, [https://ebpf.io/what-is-ebpf/](https://ebpf.io/what-is-ebpf/)  
3. Let's introduce eBPF and Aya \- DEV Community, accessed June 27, 2025, [https://dev.to/littlejo/lets-introduce-ebpf-and-aya-40ji](https://dev.to/littlejo/lets-introduce-ebpf-and-aya-40ji)  
4. 【翻译】Aya: Rust风格的eBPF 伙伴, accessed June 27, 2025, [https://rustcc.cn/article?id=e5e2e832-cd14-44d6-830e-09e8b99b2d49](https://rustcc.cn/article?id=e5e2e832-cd14-44d6-830e-09e8b99b2d49)  
5. Zero to Performance Hero: How to Benchmark and Profile Your eBPF Code in Rust \- InfoQ, accessed June 27, 2025, [https://www.infoq.com/articles/benchmark-profile-ebpf-code/](https://www.infoq.com/articles/benchmark-profile-ebpf-code/)  
6. Go, C, Rust, and More: Picking the Right eBPF Application Stack, accessed June 27, 2025, [https://cloudchirp.medium.com/go-c-rust-and-more-picking-the-right-ebpf-application-stack-7abd1c1ba9f4](https://cloudchirp.medium.com/go-c-rust-and-more-picking-the-right-ebpf-application-stack-7abd1c1ba9f4)  
7. aya \- crates.io: Rust Package Registry, accessed June 27, 2025, [https://crates.io/crates/aya](https://crates.io/crates/aya)  
8. Aya: Home, accessed June 27, 2025, [https://aya-rs.dev/](https://aya-rs.dev/)  
9. Aya is an eBPF library for the Rust programming language, built with a focus on developer experience and operability. \- GitHub, accessed June 27, 2025, [https://github.com/aya-rs/aya](https://github.com/aya-rs/aya)  
10. A curated list of awesome eBPF projects using aya-rs and Rust \- GitHub, accessed June 27, 2025, [https://github.com/aya-rs/awesome-aya](https://github.com/aya-rs/awesome-aya)  
11. My first Aya program \- DEV Community, accessed June 27, 2025, [https://dev.to/littlejo/my-first-aya-program-2j0p](https://dev.to/littlejo/my-first-aya-program-2j0p)  
12. Development Environment \- Aya, accessed June 27, 2025, [https://aya-rs.dev/book/start/development/](https://aya-rs.dev/book/start/development/)  
13. Aya: your tRusty eBPF companion \- Deepfence, accessed June 27, 2025, [https://www.deepfence.io/blog/aya-your-trusty-ebpf-companion](https://www.deepfence.io/blog/aya-your-trusty-ebpf-companion)  
14. Enhancing your Aya program with eBPF maps \- DEV Community, accessed June 27, 2025, [https://dev.to/littlejo/enhancing-your-aya-program-with-ebpf-maps-4hdj](https://dev.to/littlejo/enhancing-your-aya-program-with-ebpf-maps-4hdj)  
15. Getting Started \- Aya, accessed June 27, 2025, [https://aya-rs.dev/book/](https://aya-rs.dev/book/)  
16. Writing eBPF Tracepoint Program with Rust Aya: Tips and Example \- Yuki Nakamura's Blog, accessed June 27, 2025, [https://yuki-nakamura.com/2024/07/06/writing-ebpf-tracepoint-program-with-rust-aya-tips-and-example/](https://yuki-nakamura.com/2024/07/06/writing-ebpf-tracepoint-program-with-rust-aya-tips-and-example/)  
17. How to write an eBPF/XDP load-balancer in Rust | Kong Inc., accessed June 27, 2025, [https://konghq.com/blog/engineering/writing-an-ebpf-xdp-load-balancer-in-rust](https://konghq.com/blog/engineering/writing-an-ebpf-xdp-load-balancer-in-rust)  
18. github:aya-rs:owners \- Crates.io, accessed June 27, 2025, [https://crates.io/teams/github:aya-rs:owners](https://crates.io/teams/github:aya-rs:owners)  
19. aya \- Rust \- Docs.rs, accessed June 27, 2025, [https://docs.rs/aya](https://docs.rs/aya)  
20. aya/aya/src/maps/mod.rs at main \- GitHub, accessed June 27, 2025, [https://github.com/aya-rs/aya/blob/main/aya/src/maps/mod.rs](https://github.com/aya-rs/aya/blob/main/aya/src/maps/mod.rs)  
21. Probes \- Aya, accessed June 27, 2025, [https://aya-rs.dev/book/programs/probes/](https://aya-rs.dev/book/programs/probes/)  
22. xdp in aya\_ebpf\_macros \- Rust \- Docs.rs, accessed June 27, 2025, [https://docs.rs/aya-ebpf-macros/latest/aya\_ebpf\_macros/attr.xdp.html](https://docs.rs/aya-ebpf-macros/latest/aya_ebpf_macros/attr.xdp.html)  
23. aya\_ebpf::maps \- Rust \- Docs.rs, accessed June 27, 2025, [https://docs.rs/aya-ebpf/latest/aya\_ebpf/maps/index.html](https://docs.rs/aya-ebpf/latest/aya_ebpf/maps/index.html)  
24. aya-log \- crates.io: Rust Package Registry, accessed June 27, 2025, [https://crates.io/crates/aya-log](https://crates.io/crates/aya-log)  
25. aya-log-ebpf \- crates.io: Rust Package Registry, accessed June 27, 2025, [https://crates.io/crates/aya-log-ebpf](https://crates.io/crates/aya-log-ebpf)  
26. aya-obj \- crates.io: Rust Package Registry, accessed June 27, 2025, [https://crates.io/crates/aya-obj](https://crates.io/crates/aya-obj)  
27. Aya Rust tutorial Part Three XDP Pass | by steve latif \- Medium, accessed June 27, 2025, [https://medium.com/@stevelatif/aya-rust-tutorial-part-three-xdp-pass-c9b8e6e4baac](https://medium.com/@stevelatif/aya-rust-tutorial-part-three-xdp-pass-c9b8e6e4baac)  
28. Simple Firewall with Rust and Aya | by steve latif \- Medium, accessed June 27, 2025, [https://medium.com/@stevelatif/simple-firewall-with-rust-and-aya-b56373c8bcc6](https://medium.com/@stevelatif/simple-firewall-with-rust-and-aya-b56373c8bcc6)  
29. Hello XDP\! \- Aya, accessed June 27, 2025, [https://aya-rs.dev/book/start/hello-xdp/](https://aya-rs.dev/book/start/hello-xdp/)  
30. XDP \- Aya, accessed June 27, 2025, [https://aya-rs.dev/book/programs/xdp/](https://aya-rs.dev/book/programs/xdp/)  
31. Classifiers \- Aya, accessed June 27, 2025, [https://aya-rs.dev/book/programs/classifiers/](https://aya-rs.dev/book/programs/classifiers/)  
32. Program Types \- Aya, accessed June 27, 2025, [https://aya-rs.dev/book/programs/](https://aya-rs.dev/book/programs/)  
33. Program types (Linux) \- eBPF Docs, accessed June 27, 2025, [https://docs.ebpf.io/linux/program-type/](https://docs.ebpf.io/linux/program-type/)  
34. Uprobes Siblings \- Capturing HTTPS Traffic: A Rust and eBPF Odyssey \- KungFuDev, accessed June 27, 2025, [https://www.kungfudev.com/blog/2023/12/07/https-sniffer-with-rust-aya](https://www.kungfudev.com/blog/2023/12/07/https-sniffer-with-rust-aya)  
35. \[Aya-Rust\] How to share large buffers from kernel space to user space? : r/eBPF \- Reddit, accessed June 27, 2025, [https://www.reddit.com/r/eBPF/comments/1j6bi5k/ayarust\_how\_to\_share\_large\_buffers\_from\_kernel/](https://www.reddit.com/r/eBPF/comments/1j6bi5k/ayarust_how_to_share_large_buffers_from_kernel/)  
36. Writing eBPF Programs with Rust Aya Framework, accessed June 27, 2025, [https://www.ebpf.top/en/post/ebpf\_rust\_aya/](https://www.ebpf.top/en/post/ebpf_rust_aya/)  
37. Aya Rust tutorial Part Four XDP Hello World \- DEV Community, accessed June 27, 2025, [https://dev.to/stevelatif/aya-rust-tutorial-part-four-xdp-hello-world-4c85](https://dev.to/stevelatif/aya-rust-tutorial-part-four-xdp-hello-world-4c85)  
38. How to use bpf\_d\_path correctly in rust aya-bpf fentry program? Verifier rejects pointer with "R1 type=fp expected=ptr\_" \- Stack Overflow, accessed June 27, 2025, [https://stackoverflow.com/questions/79616493/how-to-use-bpf-d-path-correctly-in-rust-aya-bpf-fentry-program-verifier-rejects](https://stackoverflow.com/questions/79616493/how-to-use-bpf-d-path-correctly-in-rust-aya-bpf-fentry-program-verifier-rejects)  
39. Aya Rust Tutorial part 5: Using Maps | by steve latif \- Medium, accessed June 27, 2025, [https://medium.com/@stevelatif/aya-rust-tutorial-part-5-using-maps-4d26c4a2fff8](https://medium.com/@stevelatif/aya-rust-tutorial-part-5-using-maps-4d26c4a2fff8)  
40. AsyncPerfEventArray in aya::maps::perf \- Rust \- Docs.rs, accessed June 27, 2025, [https://docs.rs/aya/latest/aya/maps/perf/struct.AsyncPerfEventArray.html](https://docs.rs/aya/latest/aya/maps/perf/struct.AsyncPerfEventArray.html)  
41. XDP packet capture in Rust with aya \- ReiTW, accessed June 27, 2025, [https://reitw.fr/blog/aya-xdp-pcap/](https://reitw.fr/blog/aya-xdp-pcap/)  
42. eBPF Development Practices: Asynchronously Send to Kernel with User Ring Buffer, accessed June 27, 2025, [https://github.com/eunomia-bpf/bpf-developer-tutorial/blob/main/src/35-user-ringbuf/README.md](https://github.com/eunomia-bpf/bpf-developer-tutorial/blob/main/src/35-user-ringbuf/README.md)  
43. Aya Rust eBPF: Forwarding more packets then sending \- Stack Overflow, accessed June 27, 2025, [https://stackoverflow.com/questions/79126565/aya-rust-ebpf-forwarding-more-packets-then-sending](https://stackoverflow.com/questions/79126565/aya-rust-ebpf-forwarding-more-packets-then-sending)  
44. DevMap in aya\_ebpf::maps::xdp \- Rust \- Docs.rs, accessed June 27, 2025, [https://docs.rs/aya-ebpf/latest/aya\_ebpf/maps/xdp/struct.DevMap.html](https://docs.rs/aya-ebpf/latest/aya_ebpf/maps/xdp/struct.DevMap.html)  
45. Rust and AF\_XDP; Another Load Balancing Adventure | by Ben Parli | Nerd For Tech, accessed June 27, 2025, [https://medium.com/nerd-for-tech/rust-and-af-xdp-another-load-balancing-adventure-42aab450453e](https://medium.com/nerd-for-tech/rust-and-af-xdp-another-load-balancing-adventure-42aab450453e)  
46. AF\_XDP — The Linux Kernel documentation, accessed June 27, 2025, [https://www.kernel.org/doc/html/v6.1/networking/af\_xdp.html](https://www.kernel.org/doc/html/v6.1/networking/af_xdp.html)  
47. bpf-examples/AF\_XDP-example/README.org at main · xdp-project/bpf-examples \- GitHub, accessed June 27, 2025, [https://github.com/xdp-project/bpf-examples/blob/master/AF\_XDP-example/README.org](https://github.com/xdp-project/bpf-examples/blob/master/AF_XDP-example/README.org)  
48. The Aya Book is an introductory book about using the Rust Programming Language and Aya library to build extended Berkley Packet Filter (eBPF) programs. \- GitHub, accessed June 27, 2025, [https://github.com/aya-rs/book](https://github.com/aya-rs/book)