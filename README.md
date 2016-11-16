## Sinfonia 一种构建可扩展分布式系统的范式

### 摘要

我们提出一种构建可扩展分布式系统的范式。我们要达到不需要处理消息传递协议-现存的分布式系统中主要的复杂点。相反，开发者只要设计和控制我们提供并称之为Sinfonia的数据结构。Sinfonia将应用的数据记录在一个内存结点的集合中，每个结点提供一个线性的地址空间。Sinfonia的核心是一种创新的mini-transaction原语，通过它能高效、一致地访问数据，并且屏蔽了并发与故障带来的复杂。通过使用Sinafonia，我们这几个月来实现两种相差巨大且复杂的应用：一个文件系统集群与一个群通信服务。我们的实现运转良好，规模达数百台机器。

### Categories and Subject Descriptors

C.2.4 \[Computer-Communication Networks\]: Distributed systems—Distributed applications; E.1 \[Data Structures\]: Distributed

data structures

### General Terms

Algorithms, Design, Experimentation, Performance, Reliability

### Keywords

Distributed systems, scalability, fault tolerance, shared memory, transactions, two-phase commit

1. 介绍

开发者通常使用消息传递协议构建分布式系统，通过在网络中传递消息分享数据。这种范式容易产生错误并且难以使用，因为它使用设计，实现和调试的过程卷入复杂的控制分布式状态协议中。分布式状态指的是应用主机跟其他主机需要利用与共享的数据，包含元数据，表格，配置和状态信息。处理分布式状态的协议包含复制协议，文件数据，元数据管理，缓存的一致性与组的全体成员。这些协议都非常不容易开发。

我们提出一种构建可扩展分布式系统的范式。透过我们的方案，开发者不再需要处理消息传递协议。相反的，开发者只需要在我们提供的Sinfonia服务里设计与利用数据结构。 因此我们将协议设计问题转换成更容易的数据结构设计问题。我们尤其要处理数据中心基础程序，比如分布式文件系统，锁管理服务，群组通信服务。这些应用必须能容错，可伸缩的，必须提供一致性各可接受的性能。

1. 



  

