---
layout: post
title: "从新认识网卡"
date: 2014-06-14 10:26:55 +0800
comments: true
categories:
---

最近有个 `case` 由于网卡的软中断几乎被打满导致作为七层负载均衡的 `Nginx` 机器出现大量丢包，
`Upstream Proxy` 出现大量重试，业务响应耗时明显增加。当时的主要问题是 `tcpflow` 导致的。但这次不说 `tcpflow`, 了解下网卡以及这方面的软硬件技术。
	
## 网卡相关的数据


`$ lspci -v | grep Ethernet -A 1`

	01:00.0 Ethernet controller: Intel Corporation I350 Gigabit Network Connection (rev 01)
	Subsystem: Dell Intel GbE 4P I350crNDC

以上便是使用的 `Intel I350` 的网卡，官方 Datasheet [下载](<http://www.intel.com/content/dam/www/public/us/en/documents/datasheets/ethernet-controller-i350-datasheet.pdf>)。

既然是说到调优，必然需要关注下相关的 `Performance Features`

* TCP segmentation offload Up to 256 KB 
* Fragmented UDP checksum offload for packet reassembly
* Message Signaled Interrupts (MSI)
* Message Signaled Interrupts (MSI-X) number of vectors (25)
* Packet interrupt coalescing timers (packet timers) and absolute-delay interrupt timers for both transmit and receive operation
* Interrupt throttling control to limit maximum interrupt rate and improve CPU utilization
* Rx packet split header
* Receive Side Scaling (RSS) number of queues per port (up to 8)
* RX header replication
* Low latency interrupt
* DCA support
* TCP timer interrupts
* No snoop
* Relax ordering
* TSO interleaving for reduced latency
* SCTP receive and transmit checksum offload
* UDP TSO

一堆 TSO,GSO,LRO,GRO,DCA,RSS,MSI,MSI-X 看似很高级的名词，其实也无非是一些对应场景的优化，跟《葵花宝典》类似，不是什么人都可以练的。尝试前最好了解清楚具体的含义，参考资料 `1` 讲解的很清晰明了。

## 测试相关的工具

任何 Performance 都需要进行测试以验证所做的修改是否合理，是否达到预期。测试方面没有很多的经验，这方面的水也够深的，下次详细了解下。很多时候大家都使用以下工具做些测试，官方应该有不少好东西可以扒扒。

* [NetPerf](<http://www.netperf.org/>)
* [Iperf](<http://iperf.fr/>)

## 数据的监控

作为一个 sa，我通常把监控放在很高的位置，我相信不能测量也就不能优化这句话，都不了解你想要的数据，怎么可能做好优化呢。

列几点我想到的可以做的监控，常规的 cpu，mem，io，app 不提。

* 网卡统计信息，ip/tcp 数据; 可以通过 ifconfig 或者 ethtool -S, netstat -s 或者 /proc/net/netstat 获取
* app 响应时间，通过 access_log 获取 response time。
* tcp stream 数据可以通过 tcpflow 或者 pt-tcp-model 工具来获取 

## 参考资料

以下都是非常好的学习和实践资料

1. [Effective Gigabit Ethernet Adapters-Intel千兆网卡8257X性能调优](<http://blog.csdn.net/dog250/article/details/6462389>)
2. [10g82599eb-测试优化中断处理](<http://jaseywang.me/2013/10/15/10g82599eb-%E7%BD%91%E5%8D%A1%E6%B5%8B%E8%AF%95%E4%BC%98%E5%8C%96%E6%80%BB/>)
