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
 
