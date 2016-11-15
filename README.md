# Sinfonia 一种构建可扩展分布式系统的范式

摘要

我们提出一种构建可扩展分布式系统的范式。我们要达到不需要处理消息传递协议-现存的分布式系统中主要的复杂点。相反，开发者只要设计和控制我们提供并称之为Sinfonia的数据结构。Sinfonia将应用的数据记录在一个内存结点的集合中，每个结点提供一个线性的地址空间。Sinfonia的核心是一种创新的mini-transaction原语，通过它能高效、一致地访问数据，并且屏蔽了并发与故障带来的复杂。通过使用Sinafonia，我们这几个月来实现两种相差巨大且复杂的应用：一个文件系统集群与一个群通信服务。我们的实现运转良好，规模达数百台机器。



