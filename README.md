# Distributed-Storage
分布式存储技术调研

## 对象存储

**对象存储有机融合了NAS可跨平台和SAN扩展性良好的优势，广泛应用于大规模集群系统。对象存储架构可以分为用户组件和存储组件两部分。**

![image](https://github.com/lus-oa/Distributed-Storage/assets/122666739/510621f6-9a8a-404b-afb7-9d8ac57ee001)


- **用户组件**向用户应用程序提供一些逻辑的数据结构，比如文件、路径及如何访问这些数据结构的接口。

- **存储组件**位于智能存储设备上，负责物理磁盘上具体数据块的组织。用户对存储设备的访问接口变为对象接口。这不同于传统的文件系统，文件系统的存储部分位于主机侧，对用户提供块接口。

![image](https://github.com/lus-oa/Distributed-Storage/assets/122666739/818a6dd1-ac61-4a6b-bef7-d08caac54f75)

**对象存储在数据共享、自管理和安全性方面具有如下特性：**

- **数据共享**:存储对象具有文件和块的优点，对象存储设备中的对象可以像数据块一样被直接访问，同时，对象接口还能像文件一样在不同操作系统平台上实现数据共享。

- **自管理**:对象存储设备的存储空间不再需要运行在主机上的文件系统管理而由存储设备自己管理和分配。最直接的效果是将空间管理从存储应用中剥离存储设备具有自管理特性，进而可以通过重新组织数据来提高性能，调整备份失败恢复等的策略。

- **安全性**:对象是关于一个存储设备的逻辑字节集合，它包含存储方法、数据属性和存储安全策略等。因此，对象存储系统在基于文件的数据布局、服务质量和安全性等方面有很大的改善。

### 对象存储核心组件

对象存储重新划分了数据存储的访问、控制、管理等基础存储功能，将原本文件系统的数据布局(即逻辑到物理的映射关系)在对象这一层实现，包含以下几种核心概念：

![image](https://github.com/lus-oa/Distributed-Storage/assets/122666739/41950341-60b1-4450-b036-7d6180c5f77c)

**1.对象**：对象存储系统中存储数据最基本的单位，也是最核心和基础的概念。对象负责存储一个或多个文件数据，同时包含数据的相关属性，这些属性用于定义基于文件的访问方法、安全策略等。每个对象都有唯一的对象标识。客户端通过对象标识、起始位置、数据长度等参数来访问该对象。

**2.OSD**:OSD(Object-based Storage Device，基于对象的存储设备)具有一定程度的智能，配备有单独的CPU、内存、网络系统和磁盘系统。OSD 的核心功能包括3个方面：
  - OSD 用于存储数据，能有效地管理与对象相关的数据并将它们放置在磁盘中。OSD 不提供块接口，当客户端发起数据请求时，根据其唯一ID 和偏移量进行数据访问。
  - OSD 具备智能数据布局的功能，这依赖于OSD内配备的专门 CPU 和内存资源，它们协同工作以优化数据布，进而提升磁盘性能。
  - OSD负责管理对象元数据，对象元数据的结构似于传统的inode，通常包含有关数据块和对象长度的详细信息。与这些元数据位于文件服务器权限之下的传统NAS系统相比，对象存储架构将这种管理功能集中在系统内，显著减少了客户的开销。

**3.MDS**:MDS(Metadata Service，元数据服务器)用于指导客户端与对象之间的交互，主要提供3个功能。
  - 一是对象存储访问。MDS 构建并维护文件目录树使客户端能够直接与对象交互，MDS 在这个过程中会授予客户端访问权限，每个访问请求在获得许可之前都会经过 OSD 的验证。
  - 二是管理文件和目录。MDS为存储系统内的文件建立了层次结构，支持目录和文件的创建、删除及访问控制等。
  - 三是维护客户端缓存一致性。为了提高客户端性能，对象存储文件系统通常包含客户端缓存机制。每当客户端缓存的文件发生更改时，MDS就会通知客户端，损示缓存刷新以避免因缓存数据不一致而引起的问题。

**4.对象存储文件系统的客户端**:与传统文件系统类似，计算节点需要运行对象存储文件系统的客户端，进而提供访问 OSD的能力，该客户端允许应用程序像访问标准POSIX文件系统一样对OSD进行读写操作。

## 并行存储

**并行存储主要应用于HPC(High Performance Computing，高性能计算)领域。**

高性能计算集群通过互连技术将大量计算机系统连接在一起，通过并行计算技术将所有被连接系统组织起来共同处理大型计算问题，其运算速度极高。

例如，我国的超级计算机“神威·太湖之光”的峰值运算速度达到了每秒12.5亿亿次。

**高性能计算对处理器、内存、存储等均有极高的要求。**
- 长期以来，存储技术一直是信息技术发展的短板，在高性能计算领域，这一问题更为显著:当成千上万的处理器单元在完成大型计算任务时，存储系统有限的I/O并发能力将难以满足计算集群的数据访问需求。
- 并行文件系统正是在这一背景下被广泛研究与应用，其包含的数据缓存与共享、细粒度并发控制、对并行计算编程模型的原生支持等技术在提升存储 I/O 并行能力方面扮演重要角色。

### 并行存储的架构特点

![image](https://github.com/lus-oa/Distributed-Storage/assets/122666739/35fa3019-fd75-4ecf-b68b-709a4b5f49f5)

随着基于闪存的固态盘逐步普及，在 I/0节点上配置 SSD 用以提供聚合读写带宽成为新的趋势。

以我国“神威·太湖之光”超级计算机为例，其使用开源文件系统 Lustre 作为后端存储系统。Lustre 使用MDS(Metadata Service，元数据服务器)和OSS(Object Storage Server，对象存储服务器)分别存储元数据和数据。MDS和OSS按两个一组实行主-从备份，每组服务器管理若干存储节点上的MDT(Metadata Target，元数据目标)及OST(Object Storage Target，对象存储目标)。

### 并行存储的关键技术

并行文件系统的核心关键技术主要包括以下几个方面：

- **并行I/O**:并行I/O尽可能将数据分布至多个存储节点，利用多个存储设备的并行访及并行通道获得聚合I/O带宽，从而充分发挥硬件资源能力。

- **元数据管理**:工作进程在访问文件数据时，必须提前访问元数据服务器，获取相关文件数据的位置信息。分布式架构下多个元数据服器可以并行处理客户端发起的元数据请求，然而，文件系统目录树结构的相邻层次是存在依赖关系的，例如，某些元数据操作(例如创建、删除或重命名文件等)需要同时修改多个元数据项，如果这些元数据项被分散至不同的服务器，则需依赖分布式事务机制进行协调修改。

- **缓存技术**:高性能计算集群一般采用存算分离架构，计算节点发起的数据请求均需要通过网络传输至存储节点，引发大量网络I/O操作，导致数据访问带宽受限，延迟上升。随着NVMe高端固态硬盘等新型存储介质的出现，出现了在计算节点配备高速存储介质进行读写缓存的方案。

## P2P

![image](https://github.com/lus-oa/Distributed-Storage/assets/122666739/4b5f3db8-5688-4258-95b1-81910494a1b1)

P2P 技术在即时通信、文件共享、流媒体等领域快速发展，并成为构建大规模互联网应用的重要技术之一。

P2P存储系统是一种去中心化架构，其存储节点以一种功能对等的方式组织成一个存储网络。在传统的 C/S 架构中，请求访问由客户端发起，服务器接收并处理，而在P2P网络架构下，各节点不再有“客户端”和“服务器”形式上的区分，每个节点既是请求的发起端也是请求的处理者，去掉了中心化的概念。

### P2P特性

**P2P存储系统架构具有如下特性：**

- **去中心化**：P2P存储系统将存储资源对等分布在所有节点上，不需要像传统存储系统一样引入集中式的元数据服务器等，因此具有去中心化的特点。
- **可扩展性**:在P2P存储系统中，用户数量的增加不仅意味着服务需求的增加，也意味着系统资源总量和服务能力同步扩充，从而自始至终都能满定用户需求，理论上具有无限扩展的能力。
- **可用性**：由于存储服务天然分散在各用户节点，部分节点或网络发生故障对其他部分造成的影响很小，从而具备较强的可用性。
- **隐私性**:P2P存储系统不需要中心化节点进行请求的集中处理，因此,用户的个人存储数据遭到窃听和泄露的可能性大幅减小。
- **负载均衡**:在一个去中心化的对等架构下，数据请求不再需要发往服务端进行集中处理，每个服务器既是客户端也是服务端，并且存储资源均匀分布在各个节点，从而实现了整个系统的负载均衡。

### P2P关键技术

![image](https://github.com/lus-oa/Distributed-Storage/assets/122666739/ea29df40-07e7-4385-91ae-45478a83b49f)

**1.结构化覆盖网络**

  结构化覆盖网络的目标是将各节点在应用层进行统一互连和组织，保证任意两个节点之间可以相互通信，同时可以容忍任意节点的动态加入和退出。

  为了保证各节点网络互通，首先需要通过全局共识的命名空间管理各节点，并给每个节点提供全局唯一的命名。P2P 网络中的节点名称一般为命名空间中的一个唯一值。
  
  以Chord路由算法为例，其命名空间为一个环形空间，如右图所示，新的节点加入系统时，它会从环中选择一个位置，将其数值作为自己的唯一标符。当节点获取唯一标识后，节点间的邻接关系也被确定。通过左右邻接关系可以进一步获取其IP地址。

- 例如，节点13和节点3为节点0的左右相邻节点，由于更早加入的节点13 和节点3已经获取其各自的邻接关系，进一步地，节点0可以知道整个系统内各节点的信息。通过上述方法，整个系统的各节点路由表会被同步更新。

**2.数据分发技术**

![image](https://github.com/lus-oa/Distributed-Storage/assets/122666739/9bccf908-4ecf-44fa-97e5-c076320806fa)

在结构化覆盖网络中存储数据，还需要对数据命名，从而决定其具体的存储位置。

- 例如，如果文件A的命名为7，则其应该存储在命名为7的节点上。然而，当前没有节点7存在，可以定义一个基于区间的负责机制。让这个文件位置之后的最近一个节点负责这些文件的存储。因此，负责文件7的节点为节点9。但是某些节点加入或退出后，上述文件到节点的映射关系也会相应发生发变化，导致部分数据无法访问。

为此，主要存在两种数据分发机制，即 DHT(Distributed Hash Table，分布式散列表)直接数据分发和基于目录的简洁分发。

- **DHT直接数据分发**是指将数据直接存放在负责该数据所对应位置的节点。当某一节点离开系统后，其后继节点代为负责其存储的数据。该方法实现简单，但缺陷很明显:在系统成员发生变化时，会引入大量的数据移动，导致网络带宽的浪费。
- **基于目录的简洁分发**则将数据随意分发至网络中的任意无关节点上，然后将这些节点的位置映射作为目录信息存放在基于 DHT 机制所对应的节点中。当读取数据时，先通过目录节点获取文件的具体映射关系，然后访问数据实际存储的节点进行数据读取。这种间接数据分发方式配合数据冗余技术保证了在节点成员变化时仅需要挪动少量目录信息，而不用挪动数据本身。然而，间接分发机制增加了网络跳数，对响应延迟有一定的影响。

**3.数据冗余技术**

![image](https://github.com/lus-oa/Distributed-Storage/assets/122666739/43d74a94-1653-4d31-ac0e-d66700b47366)![image](https://github.com/lus-oa/Distributed-Storage/assets/122666739/3142f860-ddc9-418e-b05e-c022a53827b7)

在一个P2P 覆盖网络中，海量存储节点可以自由加入和退出系统，因此,P2P存储系统必须引入数据冗余技术来克服节点动态变化带来的可用性问题。

目前，使用最为广泛的数据冗余技术包括多副本冗余和纠删码冗余。

- **多副本冗余技术**即同时保存要存储数据的多份副本，并存储在安全的节点中。
- **纠删码**是指将要存储的数据先切分为k份，然后通过编码算法生成r份校验数据井将(k+r)份数据分布到安全的不同节点上。通过纠删码技术，任意t份数据(t≥k)均可以用来恢复原始数据。

## 分布式键值存储系统

![image](https://github.com/lus-oa/Distributed-Storage/assets/122666739/e431ab9e-e629-461b-a317-71cae03e322f)


**分布式键值存储系统将键值对(key-value)存储在大量的服务器中，像应用提供简洁的键值访问接口：通过唯一的主键去查询（get）、插入（put）或删除（delete）对应的键值对。
由于键值对之间没有依赖关系，多以分布式键值存储系统的操作不需要分布式协调，具有极佳的扩展性。**

分布式键值存储系统按照规模可分为两种：跨数据中心和单数据中心。
- **跨数据中心分布式键值存储系统**用于全球性业务，数据在多个数据中心进行复制冗余，一致性保证较低，但可用性高，能够容忍整个数据中心的崩溃。
- **单数据中心分布式键值存储系统**将键值数据存储在单个数据中心里，一般保证强一致性，并且可以利用数据中心内部特殊硬件（如RDMA网卡）进行性能优化。

### **针对特殊场景，分布式键值存储系统会提供一些专门接口以扩展语义。**

- **范围查找**:范围查找会返回用户指定键区间的所有键值对。
例如，当某个布式键值存储系统的主键为用户ID，值为用户数据，则能采用范围查找来获得户ID在[10,100]区间中的所有用户数据。若分布式键值存储系统支持范围查找则每个数据分区使用的底层索引一般为有序索引，如LSM树和 B+树。
- **事务操作**: 事务操作保证对多个键值对的访问是原子性的。
这里原子性包括两部分:
    - **持久化的原子性**：即事务内对多个键值的修改必须全部被持久化在存储介质中，而不会存在部分修改;
    - **并发的原子性**：即多个事务操作并发执行的结果与这些事务串行执行的结果相同。
      
由于一个事务操作中的多个键值对可能被存储在不同的服务器中，分布式键值存储系统通过轻量级的分布式并发控制协议(如乐观并发控制)及事务日志来保证事务操作的原子性。
- **二级索引**:支持二级索引的分布式键值存储系统允许用户使用二级键访问键值对。
例如，某个分布式键值存储系统的主键为用户 ID，二级键为用户出生年月，值为用户数据，则能访问特定出生年月的用户对应的数据。与主键的唯一性不同，通过某个二级键可以访问多个数据。

### 典型分布式键值存储系统

**RAMCloud**

RAMCloud是美国斯坦福大学的研究者提出的分布式键值存储系统，它的要设计目标是实现低延迟。RAMCloud在DRAM 中维护所有的数据，并目依赖高速网卡和用户态网络栈进行服务器间通信。

![image](https://github.com/lus-oa/Distributed-Storage/assets/122666739/66aad3ea-00f4-4500-9168-81ee7e6c23db)

上展示了 RAMCloud 的系统架构。

RAMCloud 运行在单个数据中心内部包含多台存储服务器、一台协调服务器及多台客户端服务器。
其中每台存储服务器包含 Master 和 Backup 两个组件:
  - **Master(控制器)** 将键值对存储在 DRAM中，并服务户端的请求;
  - **Backup(备份)** 将其他 Master 中的键值对冗余存储在磁盘或闪存中。

协调服务器高可靠地存储着集群配置信息，以及数据分区到存储服务器的位置映射。应用服务器缓存着位置映射，可以直接向存储服务器发出存储请求。




