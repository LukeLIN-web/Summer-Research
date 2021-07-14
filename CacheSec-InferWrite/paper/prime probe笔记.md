# prime probe笔记

## Abstract

We present an effective implementation of the PRIME+PROBE side-channel attack against the lastlevel cache. We measure the capacity of the covert channel the attack creates and demonstrate a cross-core, cross-VM attack on multiple versions of GnuPG.跨核跨虚拟机的攻击 Our technique achieves a high attack resolution without relying on weaknesses in the OS or virtual machine monitor or on sharing memory between attacker and victim.
Keywords—Side-channel attack; cross-VM side channel;
covert channel; last-level cache; ElGamal ,一种非对称加密算法;

Infrastructure-as-a-service (IaaS) cloud-computing
services provide virtualized system resources to end users, supporting each tenant租户 in a separate virtual machine (VM). Fundamental to the economy of clouds is high resource utilization achieved by sharing: providers co-host multiple VMs on a single hardware platform, relying on the underlying潜在的 virtual-machine monitor (VMM) to isolate VMs and schedule system resources.
While virtualization creates the illusion of strict isolation and exclusive resource access, in reality the virtual resources map to shared physical resources, creating the potential of interference between co-hosted VMs. A malicious恶意的 VM may learn information on data processed by a victim VM [32, 42, 43] and even conduct side-channel attacks on cryptographic implementations 

Previously demonstrated side channels with a resolution sufficient for cryptanalysis attacked the L1 cache. However, as Figure 1 shows, the L1 Data and Instruction caches (denoted L1 D$ and L1 I$) are private to each processor core. This limits the practicability of such attacks, as VMMs are not very likely to co-locate multiple owners’ VMs on the same core. In contrast, the last-level cache (LLC) is typically shared between all cores of a package, and thus constitutes a much more realistic attack vector. 一般最后一级被所有core 分享, 攻击比较常见

However, the LLC is orders of magnitude larger 大好几个数量级
and much slower to access than the L1 caches, which drastically reduces the temporal resolution of observable events and thus channel bandwidth, making most published LLC attacks unsuitable for cryptanalysis [32, 42, 43]. An exception is the FLUSH+RELOAD attack [22, 45], which relies on memory sharing to achieve high resolution. Virtualization vendors explicitly advise against sharing memory between VMs [39], and no IaaS provider is known to ignore this advice [36], so this attack also fails in practice.   现有的很多不能攻击加密的llc, flush reload 往往提供商会建议别共享内存, 所以也没有机会.

We show that an adaptation of the PRIME+PROBE
technique [28] can be used for practical LLC attacks. We exploit hardware features that are outside the control of the cloud provider (inclusive caches) or are controllable but generally enabled in the VMM for performance reasons (large page mappings).  Beyond that, we make no assumptions on the hosting environment, other than that the attacker and victim will be co-hosted on the same processor package.    prime probe可以利用运营商无法控制或通常开启的硬件特性

We demonstrate an asynchronous PRIME+ PROBE attack on the LLC that does not require sharing cores or memory between attacker and victim, does not exploit VMM weaknesses and works on typical server platforms, even with the unknown LLC hashing schemes in recent Intel processors (Section IV); 不用共享内存, 在服务器平台就可以attack, 不用知道llc hashing 方案

• We develop two key techniques to enable efficient LLC based PRIME+PROBE attacks: an algorithm for the attacker to probe exactly one cache set without knowing the virtual-address mapping, and using temporal access patterns instead of conventional spatial access patterns to identify the victim’s security-critical accesses (Section IV);
• We measure the achievable bandwidth of the cross-VM covert timing channel to be as high as 1.2 Mb/s (Section V);
• We show a cross-VM side-channel attack that extracts a key from secret-dependent execution paths, and demonstrate it on Square-and Multiply modular exponentiation in an ElGamal decryption implementation (Section VI);
• We furthermore show that the attack can also be used on secret-dependent data access patterns, and demonstrate it on the sliding-window modular exponentiation implementation of ElGamal in the latest GnuPG version (Section VII).

## II. BACKGROUND

### A. Virtual address space and large pages

 A process executes in its private virtual address space, composed of pages, each representing a contiguous range of addresses. The typical page size is 4 KiB, although processors also support larger pages, 2 MiB and 1 GiB on the ubiquitous无处不在的 64-bit x86 (“x64”) processor architecture. Each page is mapped to an arbitrary任意 frame in physical memory.
In virtualized environments there are two levels of address-space virtualization. The first maps the virtual addresses of a process to a guest’s notion of physical addresses, i.e. the VM’s emulated physical memory. The second maps guest physical addresses to physical addresses of the processor. For our purposes, the guest physical addresses are irrelevant, and we use virtual address for the (guest virtual) addresses used by application processes, and physical address to refer to actual (host) physical addresses. 一个是把处理器的虚拟地址映射到模拟的物理地址, 一个是把来宾的物理地址映射到实际的物理地址. (来宾的物理地址 是不是就是模拟的物理地址?  )

Translations from virtual pages to physical frames are stored in page tables. Processors cache recently used page table entries in the translation look-aside buffer (TLB). The TLB is a scarce processor resource with a small number of entries. Large pages use the TLB more efficiently, since fewer entries are needed to map a particular region of memory. As a result, the performance of applications with large memory footprints大内存占用, such as Oracle databases or high-performance computing applications, can benefit from using large pages. For the same reason, VMMs, such as VMware ESX and Xen HVM, also use large pages for mapping guest physical memory [38].

large page用tlb效率高, 

