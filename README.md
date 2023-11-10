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
