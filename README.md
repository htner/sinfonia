## Sinfonia 一种构建可扩展分布式系统的范式

### 摘要

我们提出一种构建可扩展分布式系统的范式。我们要达到不需要处理消息传递协议的目的，它是现存的分布式系统主要的复杂点。相反的， 开发者只需要在我们提供的Sinfonia服务里设计与利用数据结构 。Sinfonia将应用的数据记录在一个内存结点的集合中，每个结点提供一个线性的地址空间。Sinfonia的核心是一种创新的mini-transaction原语，通过它能高效、一致地访问数据，并且屏蔽了并发与故障带来的复杂。通过使用Sinafonia，我们这几个月来实现两种相差巨大且复杂的应用：一个文件系统集群与一个群通信服务。我们的实现运转良好，规模达数百台机器。

### Categories and Subject Descriptors

C.2.4 \[Computer-Communication Networks\]: Distributed systems—Distributed applications; E.1 \[Data Structures\]: Distributed

data structures

### General Terms

Algorithms, Design, Experimentation, Performance, Reliability

### Keywords

Distributed systems, scalability, fault tolerance, shared memory, transactions, two-phase commit

### 1.介绍

开发者通常使用消息传递协议构建分布式系统，通过在网络中传递消息分享数据。这种范式容易产生错误并且难以使用，因为它使用设计，实现和调试的过程卷入复杂的控制分布式状态协议中。分布式状态指的是应用主机间需要利用与共享的数据，包含元数据，表格，配置和状态信息。处理分布式状态的协议包含复制协议，文件数据，元数据管理，缓存的一致性与组的全体成员。这些协议都非常不容易开发。

我们提出一种构建可扩展分布式系统的范式。透过我们的方案，开发者不再需要处理消息传递协议。相反的，开发者只需要在我们提供的Sinfonia服务里设计与利用数据结构。 因此我们将协议设计问题转换成更容易的数据结构设计问题。我们尤其要处理数据中心基础程序，比如分布式文件系统，锁管理服务，群组通信服务。这些应用必须能容错，可伸缩的，必须提供一致性各可接受的性能。

简言之 ，Sinfonia是一个允许主机间容错的，可伸缩的，一、致地共享应用数据的服务。现存的能让主机间共享数据的服务包括了数据库系统与分布式共享内存（DSM）。数据库性能不足以支持性能高效至关重要的基础应用。因为数据库系统提供比需求更多的功能，导致性能消耗大。例如使用数据库构建的文件系统是一个性能底下无法使用的系统。现存的DSM系统常缺乏基础程序需求的可伸缩性或容错性。第八节一些跟Sinfonia很像的DSM系统。

Sinfonia追求提供一个在功能与可伸缩性中的平衡。实现可伸缩的关键是尽量分离不同的主机间的操作，从而操作可以独立地进行。为了达到这目的，Sinfonia提供一种细粒度的地址空间来保存数据，无须给人印象深刻的结构，比如类型、 模式、元组或者表格， 它们都倾向提高耦合。因此，应用主机间能够在Sinfornia中 相对独立地操作作数据。为预防Sinfonia成为瓶颈，Sinfonia通过多个内存结点构成分布式的系统，内存结点的多少决定了Sinfonia空间大小和带宽。

Sinfonia的核心是一个轻量级的minitransaction原语，应用程序能原子性地访问跟有条件地修改内存结点组的数据。例如， 一个 我们用 Sinfonia 来实现的分布式文件系统。通过 minitransaction , 一个主机可以原子地将一个inode放入一个内存结点，将这个indoe 链接到存在另一个内存结点目录，这些更新都是在前一个inode受约束的进行（为避免竞争）。像数据库事务一样，minitransactions隐藏由于并发执行与错误带来的复杂度。

minitranscation在对于改善性能在很多方面起作用。首先，minitransactions允许用户将批量更新放一起，可以消除多次网络往返的开消。其次，因为限制作用域，minitransactions可以在提交协议中执行。事实上，Sinfonia可以使用二次的网络往返就可以开始，执行， 提交一个minitransaction。作为对比，拥有更多功能与更高级的数据库事务，



