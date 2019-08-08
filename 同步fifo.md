基于verilog的同步FIFO设计
===
工具 vivado2016.2 
--
同步fifo
----
FIFO的英文全称为first in first out,顾名思义就是先进先出的意思。fifo又分为同步fifo与异步fifo，fifo通常作为不同时钟域之间的数据传递，以及不同数据接口之间数据匹配。这次主要进行同步FIFO的研究。
* 同步fifo原理框图
![fifo 框图](https://img-blog.csdnimg.cn/20190804114642220.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDI2MDAx,size_16,color_FFFFFF,t_70)
由上图可知fifo主要有输入：clk,rst_n,read,write,data_in.输出：full,emty,data_out。与普通存储器不同的是fifo没有外部读写地址的输入，因此fifo使用较为简单，同样这也造成无法无法随意访问任意地址，只能顺序输出。

FIFO设计难点以及主要参数
---
fifo主要参数有，fifo深度，fifo宽度，空标志，满状态。
fifo设计的主要难点：
	fifo设计主要难点在于如何判断fifo的满状态以及空标志。如果不能准确判断fifo的满状态可能造成写的数据被覆盖。同样如果不能准确判断fifo的空可能造成读书的数据错误。博主在设计这两个状态是就出现了相应的问题，最后进行修改才能正确进行读写。
	以下是博主第一次进行设计的同步fifo，代码如下：
	
```verilog
`timescale 1ns / 1ps

//////////////////////////////////////////////////////////////////////////////////
module snfifo1(
									input clk,
									input rst_n,
									input[3:0] data_in,
									input write,
									input read,
									output[3:0] data_out,
									output  full,
									output   emty
								);
								
reg[3:0] fifo_reg[3:0];//depth4
reg[1:0] w_head,R_tail;
reg[1:0] count;
reg [3:0] dout;
reg f,e;
parameter MAX = 3;
//*************update w_head*************		
always @ (posedge clk)
	if(!rst_n)
		w_head <= 2'd0;
	else if(write && !full)
		w_head <= w_head +  1'b1;
		//else if(full) w_head <= 3'd0;
//*************update R_tail*************		
always @ (posedge clk)
	if(!rst_n)
		R_tail <= 2'd0;
	else if(read && !emty)
		R_tail <= R_tail +  1'b1;
		//else if(emty) R_tail <= 3'd0;
		
//**************count************
always @(posedge clk)
	if(!rst_n)
	begin
		count <= 2'd0;
		
    end
	else
	 case({read,write})
	 		2'b00:
	 			count <= count;
	 		2'b01:
               if(count != MAX)
                  
                   count <= count + 1'b1;
                   
	 		2'b10:
	 			if(count != 0)
	 			
	 				count <= count - 1'b1;
	 		2'b11:  
	 		      ; // count <= count;
	 endcase
//************read**********
always @(posedge clk)
	if(!rst_n)
		dout <= 4'd0;
	else if(read && !emty)
		dout <= fifo_reg[R_tail];
///***************write**********		
always @(posedge clk)
	if(write && !full)
		fifo_reg[w_head] <= data_in;

////-----------------------------
assign full = (count  == MAX); //? 1'b1:1'b0;
assign emty = (count  == 0 );//	? 1'b1:1'b0;


assign data_out = dout;
endmodule
```
 仿真结果如图
![仿真结果](https://img-blog.csdnimg.cn/20190804160611756.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDI2MDAx,size_16,color_FFFFFF,t_70)
还设计中的emty,full信号采用组合逻辑实现，读写以及读写指针采用时序逻辑设计，结果发现最后一位数据总是无法写入，通过观察结果发现当count为3时此时full信号拉高，同时显示fifo为满，因此无法再次写入。分析full 应该在count为3时延迟一个始终输出因此采用如下设计。
````verilog
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: zhangsy
// 
// Create Date: 2019/08/01 10:19:25
// Design Name: 
// Module Name: snfifo1
// Project Name: 
// Target Devices: 
// Tool Versions: 
// Description: 
// 
// Dependencies: 
// 
// Revision:
// Revision 0.01 - File Created
// Additional Comments:
// 
//////////////////////////////////////////////////////////////////////////////////


module snfifo1(
									input clk,
									input rst_n,
									input[3:0] data_in,
									input write,
									input read,
									output[3:0] data_out,
									output  full,
									output   emty
								);
								
reg[3:0] fifo_reg[3:0];//depth4
reg[1:0] w_head,R_tail;
reg[1:0] count;
reg [3:0] dout;
reg f,e;
parameter MAX = 3;
//*************update w_head*************		
always @ (posedge clk)
	if(!rst_n)
		w_head <= 2'd0;
	else if(write && !full)
		w_head <= w_head +  1'b1;
		//else if(full) w_head <= 3'd0;
//*************update R_tail*************		
always @ (posedge clk)
	if(!rst_n)
		R_tail <= 2'd0;
	else if(read && !emty)
		R_tail <= R_tail +  1'b1;
		//else if(emty) R_tail <= 3'd0;
		
//**************count************
always @(posedge clk)
	if(!rst_n)
	begin
		count <= 2'd0;
		
    end
	else
	 case({read,write})
	 		2'b00:
	 			count <= count;
	 		2'b01:
               if(count != MAX)
                  
                   count <= count + 1'b1;
                   
	 		2'b10:
	 			if(count != 0)
	 			
	 				count <= count - 1'b1;
	 		2'b11:  
	 		      ; // count <= count;
	 endcase
//************read**********
always @(posedge clk)
	if(!rst_n)
		dout <= 4'd0;
	else if(read && !emty)
		dout <= fifo_reg[R_tail];
///***************write**********		
always @(posedge clk)
	if(write && !full)
		fifo_reg[w_head] <= data_in;
		
always @ (posedge clk)
            if(count==0)
                e <= 1'b1;
              else e <= 1'b0;
          always @ (posedge clk)
                  if(count==MAX)
                      f <= 1'b1;
                    else f <= 1'b0;
  assign full = f;
  assign emty = e;
                   
////-----------------------------
//assign full = (count  == MAX); //? 1'b1:1'b0;
//assign emty = (count  == 0 );//	? 1'b1:1'b0;


/*always @ (count)
    if(count==0)
        emty <= 1'b1;
      else emty <= 1'b0;
  always @ (count)
          if(count==MAX)
              full <= 1'b1;
            else full <= 1'b0;    
        
*/
assign data_out = dout;
endmodule
````
仿真结果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019080416134548.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDI2MDAx,size_16,color_FFFFFF,t_70)
仿真结果显示能够实现正常数据输入以及输出。
激励文件如下：
````verilog
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: zhangsy
// 
// Create Date: 2019/08/01 10:32:56
// Design Name: 
// Module Name: txt
// Project Name: 
// Target Devices: 
// Tool Versions: 
// Description: 
// 
// Dependencies: 
// 
// Revision:
// Revision 0.01 - File Created
// Additional Comments:
// 
//////////////////////////////////////////////////////////////////////////////////


module txt();
reg clk;
reg rst_n;
reg write,read;
reg [3:0] data_in;
wire [3:0] data_out;
wire full,emty;
initial 
begin
    clk =0;
    rst_n =0;
    #100
    rst_n = 1;
    write =0;
    read = 0;
    #20
    write = 1;
   // #20
    data_in = 4'd1;
     #20
       data_in = 4'd2;
        #20
          data_in = 4'd3;
           #20
             data_in = 4'd4;
       #20
       write =1;
       #20
         write =0;
       read =1;
       #100;
       read = 0;
      write = 1;
          // #20
           data_in = 4'd6;
            #20
              data_in = 4'd7;
              read = 1;
               #20
                 data_in = 4'd8;
                  #20
                    data_in = 4'd9;
        #20
                       write = 1;
                      
                      // #20read
                       data_in = 4'd5;
                        #20
                          data_in = 4'd2;
                           #20
                            read =0;
                             data_in = 4'd3;
                              #20
                                data_in = 4'd4;
                         
                          #20
                            write =0;
                          read =1;
       
end
always #10 clk =~clk;
 snfifo1 snfifo1_inst(
               .clk,
               .rst_n,
               .write,
               .read,
               .data_in,
               .data_out,
               .full,
               .emty
   );
endmodule

````
另一种写法如下仅供参考：
````verilog
module  synfifo1(
					input clk,
					input rst_n,
					input write,
					input read,
					input data_in,
					output data_out,
					output full,
					output emty
				);
parameter MAXDeepth = 4;
	reg dout;
	reg [1:0] W_head,R_tail;
	reg [1:0] count;
	reg [3:0] fifo_reg;
//***************write and read **************8
always @ (posedge clk)
	if(!rst_n)
		begin
			W_head <= 2'b00;
			R_tail <= 2'b00;
			fifo_reg <= 4'd0;
			count <= 2'b00;
		end
	else if(write && (!read) && (!full))
			begin
				fifo_reg[W_head] <= data_in;
				W_head <= W_head + 1'b1;
				count <= count + 1'b1;
			end
	else if (!write && read && !emty)
			begin
				dout <= fifo_reg[R_tail];
				R_tail <= R_tail + 1'b1;
				count <= count -1'b1;
			end
	else if (write && read && !emty && !full)
			begin
				fifo_reg[W_head] <= data_in;
				W_head <= W_head + 1'b1;
				dout <= fifo_reg[R_tail];
				R_tail <= R_tail + 1'b1;
			end
			always @ (posedge clk)
            if(count==0)
                e <= 1'b1;
              else e <= 1'b0;
          always @ (posedge clk)
                  if(count==MAX)
                      f <= 1'b1;
                    else f <= 1'b0;
  assign full = f;
  assign emty = e;
	//assign full = (count == MAXDeepth) ? 1'b1: 1'b0;
	//assign emty = (count == 0) ? 1'b1:1'b0;
	assign data_out = dout;
	endmodule
	````
  [csdn](https://blog.csdn.net/qq_15026001/article/details/98451457)
