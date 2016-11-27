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

minitranscation在对于改善性能在很多方面起作用。首先，minitransactions允许用户将批量更新放一起，可以消除多次网络往返的开销。其次，因为限制作用域，minitransactions可以在提交协议中执行。事实上，Sinfonia可以使用二次的网络往返就可以开始，执行， 提交一个minitransaction。作为对比，拥有更多功能与更高级的数据库事务， 光提交 阶段 就有二次的网络往返 ，在开始与执行阶段还需要额外的开销。第三，minitransactions可以并行执行，拥有复制方案，提供高可靠性，低延迟。

我们验证Sinfonia是通过在二个复杂且相差大的应用中使用：一个分布式文件系统SinfoniaFS与一个集群聊天服务SinfoniaGCS。这些应用以难以实现兼得扩展性与容错性而著名：系统要达到这些目标是结构复杂，几年的努力才能完成。通过使用Sinfonia，我们构造它们分别只需要3900与3500行代码，一个与两个人月的工作量。在SinfoniaFS，Sinfonia用来贮藏文件系统的数据，集群中每个结点 使用minitransactions原子性地检索与更新文件数据、属性、分配与回收空间。在SinfoniaBGCS，Sinfonia保存有序的消息，用户使用minitransactions增加新的消息到消息序列里。

经验显示Sinfonia和使用它的应用伸缩性好，运行良好。Sinfonia在 单 个结点的情况下每秒可低延迟地执行数千个minitransactions，随着系统数据增大总处理能力扩大得很好。SinfoniaFS通过单个内存结点执行得跟NFS服务一样好，不像NFS服务，SinfoniaFS很容易就扩展到数千个结点。SinfoniaGCS扩展性要强于Spread, Spread是一个有强大处理能力的集群通迅服务的实现。

### 2.假设与目标

我们考虑分布式系统只在一个数据中心内部。一个数据中心由很多连接相当好的机器组成的场所。它能从数十到数千台机器上跑数十到数千个应用。网络延迟值很小，大多数方差也小，跟复杂的异步系统。网络切割有时会在一个数据中心发生，但它是罕见的;当发生时，我们可以暂停服务，因此最大可能地数据中心会不可用。数据中心中的应用设计成可信任的而非恶意的。（访问协议是一个跟现有功能互不相关的，可以合并到Sinfonia，但目前还没有这样做。）注意这些假设不包含一个很大区域的网络，端到端的系统或者互联网。

这个数据中心出现故障：一个结点可能会崩溃，更罕见的， 可能所有结点都崩溃（例如，因为电力中断），错误会在不可预知的时间发生。单独的机器是可靠的，但许多台机器的话崩溃就是经常的。我们不考虑拜占廷问题。硬盘提供可靠的存储，也就是说硬盘提供了足够的可靠性给了目标应用。这要求选择硬盘必须要小心的：选择是多样的，从花销少的硬盘到高端的磁盘阵列。可靠的存储可能会坏掉，我们需要处理它，但有时坏掉是罕见的。

我们的目标是帮助开发者构造分布式的基础应用，这些应用用来支撑其他的应用。例如包括锁管理，分布式文件系统，集群通信系统，分布式名字管理服务。这些应该需要提供可靠的，一致的，伸缩性。伸缩性是通过扩大系统数量会成比例地增长系统容量。在本文中，容量是指处理容量，用来衡量总的吞吐量。

### 3. 设计

我们现在来描述Sinfonia的原理与设计

**3.1 原理**

设计Sinfonia是基于以下两个原理：

_原理1: 减少操作耦合来获取可伸缩性。_耦合是指操作间会相互依赖，这会限制不同主机行的并行执行，因此会阻碍可伸缩性。Sinfonia通过不提供结构化的数据避免耦合。

_原理2: 在扩展组件之前将它可靠化。_我们首先使用独立的Sinfonia结点具有容错性，然后再扩展系统加入更多结点。因此我们避免大系统中有许多不可靠的组件带来的复杂性。

**3.2 基础组件**

Sinfonia包含了一个内存结点集合和一个在应用结点运行的用户库。内存结点保存数据，在RAM或者可靠存储中，由应用需求决定。用户库实现了操作内存结点的数据的机制。将内存结点与应用结点放在同一个主机是可能的，但逻辑上他们是分离的。

每个内存结点保存一系列原始的无解释的标准长度的字节;在本文，字节的长度是1byte.这些字节会组织成一个无结构的线性地址空间。每个内存结点会有一个单独的地址空间，所以Sinfonia的数据会通过一个对（内存结点id, 地址）来引用。我们也尝试设计一个单一全局的地址空间透明映射到内存结点，但是这个设计难以扩大因为它缺乏结点局部化。结点局部化是指将一起访问的数据放置到同一个结点。比如说，我们的分布式文件系统努力将一个inode, 它的链接链表，和它的文件数据放到同一个内存结点（如果空间允许）。这在一个透明映射的空间地址是难以实现的。结点局部化的相反是数据分割，它会将一起访问的数据分散到许多结点。数据分割改善单用户的吞吐量，但是我们的经验显示它会伤害到扩展性。

应用结点通过用户类库访问Sinfonia的数据。该类库提供基础读写一个内存结点的一个字节区域，还有minitransaction, 将在下面描述。

**3.3 Minitransaction**

Minitransaction允许应用在多个内存结点中原子性，一致性，隔离性 ，和持久性更新数据。原子性是指一个minitransaction全部执行或者全部不执行;一致性是指数据不会错误;隔离是指minitransaction可以并行;持久性是指提交的minitransactions不会丢失，即使发生错误。

我们调整minitransaction的能力致使在同样作用的情况下更有效率地执行。为了解释我们是怎样做到的，我们首先描述标准的分布式事务是怎样执行和提交的。粗略地说，一个协调者执行一个事务是通过请求参与者执行一个或多个事务，比如检索或修改数据项。在事务的最后，协调者会执行二阶段提交。在第一个阶段，协调者询问所有的参与者是否准备好提交，如果他们都投票成功，在第二个阶段协调者告诉他们提交或者中断。在Sinfonia,协议者是应用程序，参与者是内存结点。

我们观察到优化一些事务的执行是可行的，比如下面的例子。如果事务最后的操作不影响协调者做出的提交或中断的决定，那么协调者可以将最后的操作附带到第一阶段上（例如，一个操作是更新一项数据的例子）。这样的优化并不影响事务语义并且节省一个交流的往返。

即使事务最后的操作影响协调者作出提交或中断的决定，如果参与者知道协调者是怎样做出决定的，这样我们就可以将动作附带在提交协议中。比如，如果最后的动作是一个读并且参与者知道协调者在读返回0时会中断（或者提交），那么协调者可以附带这个动作到二阶段提交协议，参与者可以读这个数据项并在结果是0的时投票中断。

事实上，将整个事务操作附带到提交协议是可行的。我们设计·minitransactions因此。。。

更准确的说，一个minitransactions包含一系列的比较项，读项与写项。每个项都指定一个内存结点与内存结点的一个地址范围;比较与写项还包含了数据。项会在minitransactions执行之前选择。在执行时，minitransaction会做以下的事情：\(1\)比较那些比较项的位置，如果存在将数据紧跟在比较项上\(相同的比较\)。 \(2\)如果所有的比较成功，或者没有比较项，返回读项与写项的位置，\(3\)如果有比较失败，中断。因此，比较项控制minitransaction提交还是中断，然后读写项决定minitranstion的返回与更新。

minitranstion是一个用来控制分布式数据的强大底层。minitransactions样例包含在下面：

1.交换。一个读请求返回旧数据并且一个写请求覆盖它。

2.比较和交换。一个比较请求比较当值和一个固定值;如果相等一个写请求会覆盖它。

3.原子地读许多数据。完成多个读请求。

4.请求一个租约。一个比较项检查一个位置是否设为0，如果是的话写请求设置一个非0的租约拥有者的id，并且另一个写请求设置租约的时间。

5.原子性地请求多个租约。跟上面一样，除了有多个比较与写请求。注意每个租约可能会在不同的内存结点。

6.如果拥有租约则更改数据。一个比较读求检查是否拥有租约，如果有写请求更新数据。

一个频繁的minitransaction特色是使用比较请求来









