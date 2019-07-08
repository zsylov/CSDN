AXI总线介绍
=====
 * AXI是ARM公司提出的AMBA（Advanced Microcontroller Bus Architecture）3.0协议中最重要的部分。AMBA4.0将其修改升级为AXI4.0。AMBA4.0 包括AXI4.0、AXI4.0-lite、ACE4.0、AXI4.0-stream  
      AXI4.0-lite是AXI的简化版本，ACE4.0 是AXI缓存一致性扩展接口，AXI4.0-stream是ARM公司和Xilinx公司一起提出，主要用在FPGA进行以数据为主导的大量数据的传输应用。  
      AXI是一种面向高性能，高带宽，低延迟的片内总线，AXI总线的特有的单向通道体系结构。单向通道体系结构使得片上的信息流只以单方向传输，减少了延时。从而使  
      SOC面积更小，功耗更低。
 ### 单向通道体系结构。信息流只以单方向传输，简化时钟域间的桥接，减少门数量。当信号经过复杂的片上系统时，减少延时。主要包括5个传输通道：  
 * 读地址通道 read address channel  
 * 读数据通道 read data channel  
 * 写地址通道 write address channel
 * 写数据通道 write data channe
 * 写响应通道 write response channel   
**其中由于读响应与读数据方向相同，因此将读响应与读数据通道合并**
`5条通道都包含信号信息与双路valid及ready握手信号`
+ __读事务结构__
 
  + 其中分为读地址通道和读数据通道。  
  * 读地址通道包含事务的必须的地址和控制信息。
  * 读数据地址包含从设备到主机传递的数据以及读响应。其中读数据通道包含一个last信号，表明传输结束。  
 
* **写事务结构**

  * 包含写数据通道，写地址通道以及写响应通道。
  * 写地址通道包含事务的必须的地址和控制信息。
  * 写数据通道包含主机向从机传递的数据信息。
  * 写响应通道每完成一个写突发方式产生一次。
  
__传输地址和数据都是在VALID和READY为高时有效。__
信号描述
====
全局信号
---
|信号名   | 源     |描述     |
|:-------:|:------:|:------:|
|ACLK     |时钟源  |全局时钟信号|
|ARESETn  |复位源  |全局复位信号，低电平有效|
写地址通道信号
----
|信号名  |源  |描述|
|:-------:|:------:|:------:|
|AWID  |Master  |写地址ID，用来标志一组写信号|
|AWADDR  |Master|写地址，一次突发写的首地址|
|AWLEN   |Master |突发长度，突发写传输数据的个数|
|AWSIZE   |Master |突发大小，每次突发传输的字节数|
|AWBURST   |Master |突发类型，FIXED，INCR，WRAP|
|AWLOCK   |Master |总线锁信号，normal, exclusive, locked|
|AWCACHE   |Master  |Cache类型，表明一次事务是怎样通过系统的bufferable, cacheable, read-allocate, write-allocate|
|AWPROT    |Master |保护类型，传输的特权级及安全等级|
|AWQOS    |Master |质量服务QoS，可作为安全级标志|
|AWREGION  |Master |域标记，可以实现一个物理地址与多个逻辑地址的映射，也可以对某些地址进行保护|
|AWUSER  |Master |用户自定义信号|
|AWVALID  |Master  |有效信号，表明此通道的地址控制信号有效|
|AWREADY  |Slave  |从设备已经准备好接受地址和控制信息|



      
      
