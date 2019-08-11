异步fifo设计，仿真
====

前面研究了同步fifo，并进行了仿真验证，有关同步fifo问题可以参考本人上一篇博客，本次主要对异步fifo进行设计仿真验证。  
与同步fifo相比，异步fifo主要不同之处在于读写时钟不同，因此异步fifo需要处理的问题较为复杂，通常需要处理注意的问题点有以下几点：  

* 1. 不同时钟域之间信号的同步化处理。
* 2. 异步fifo的空状态与满状态的判断。
* 
异步fifo框图
----
![异步fifo](https://img-blog.csdnimg.cn/20190811103609124.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDI2MDAx,size_16,color_FFFFFF,t_70)
---------------------------------------------------
不同时钟域之间信号的同步化处理。
-----
  针对读写时钟域的不同采用寄存器打两拍来实现同步化处理。另外由于二进制在两个时钟域之间传递，考虑到这个问题将二进制转换为gray(格雷码)，由于gray  
  相邻两位之间只有一位不同，因此在不同域之间传递减小了出错的概率。（两种方式结合都是为了较小亚稳态）
  
  二进制转换为gray方法
  --- 
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190811103859430.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDI2MDAx,size_16,color_FFFFFF,t_70)
verilog 实现二进制转gray
---
//---------------gray----------
assign r_gray = (r_reg2 >> 1) ^r_reg2;
assign w_gray = (w_reg2 >> 1) ^w_reg2;	

判断fifo的空满状态
---
判断fifo的状态需要考虑几个问题：
* 1当读时钟快于写时钟的时候如何考虑；
* 2当读时钟慢于写时钟的时候如何考虑；
* 3何时将不同的时钟同步到另一个时钟域；
* 4如何用格雷码进行fifo状态的判断；
以上几个问题可以总结为，需要判断哪个状态就将另一个时钟同步过来，例如判断fifo的满状态，因为fifo的状态影响到fifo的写功能，因此我们需要在写时钟域进行判断，即将读始终同步到写时钟域。另外在判断fifo状态时会出现虚空虚满的状态，但是不影响实际的功能。

verilog代码
---
````verilog
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: zhangsy
// 
// Create Date: 2019/08/05 11:58:12
// Design Name: 
// Module Name: asynfifo
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


module asynfifo#(
									parameter data_depth = 4,
									parameter data_width = 4,
									parameter ptr_width  = 2
								)
								(
								
									input w_clk,r_clk,
									input write,read,
									input rst_n,
									input [data_width-1:0] data_in,
									output [data_width-1:0] data_out,
									output  valid,
									output full,
									output emty
								);
///////////////////////////////////////////////////////////////////
//name              enable
//write               H
//read                H
//rst_n               L
//valid               H
//full                H
//emty                H
/////////////////////////////////////////////////////////////////////
reg [data_width-1:0]fifo [data_depth-1:0];
reg [ptr_width:0] r_ptr;
wire [ptr_width:0]r_ptr_g;
reg [ptr_width:0] w_ptr;
wire [ptr_width:0] w_ptr_g;
//****************update r_ptr************
always @ (posedge r_clk)
	if(!rst_n)
		r_ptr <= 2'd0;
	else if(read && !emty )
		r_ptr <= r_ptr + 1'b1;
	assign r_ptr_g = (r_ptr >>1) ^r_ptr;
//****************update w_ptr************
always @ (posedge w_clk)
	if(!rst_n)
		w_ptr <= 2'd0;
	else if(write && !full )
		w_ptr <= w_ptr + 1'b1;	
		
assign w_ptr_g = (w_ptr >>1) ^w_ptr;

//---------------------------------read-------------
//******************synchronization w_ptr***********
reg[ptr_width:0] w_reg1,w_reg2;
wire [ptr_width:0] w_gray;
reg [data_width-1:0] dout;
reg valid_r;
always @ (posedge r_clk)
	if(!rst_n)
		{w_reg1,w_reg2} <= 2'b00;
	else
		{w_reg1,w_reg2} <= {w_ptr,w_reg1};
		
always @(posedge r_clk)
	if(!rst_n)
		begin
			dout <= 'h0;
			valid_r <= 1'b0;
		end
	else if(read && !emty)
		begin
			dout <= fifo[r_ptr[ptr_width-1-:ptr_width]];
			valid_r <= 1'b1;
		end 
		else
		begin
			dout <= 'h0;
			valid_r <= 1'b0;
		end
//---------------------------------write-------------
//******************synchronization r_ptr***********
reg[ptr_width:0] r_reg1,r_reg2;
wire [ptr_width:0] r_gray;

always @ (posedge w_clk)
	if(!rst_n)
		{r_reg1,r_reg2} <= 2'b00;
	else
		{r_reg1,r_reg2} <= {r_ptr,r_reg1};

always @ (posedge w_clk)
	 if(write && !full)
		begin
			fifo[w_ptr[ptr_width-1-:ptr_width]] <= data_in;
		end 

//---------------gray----------
assign r_gray = (r_reg2 >> 1) ^r_reg2;
assign w_gray = (w_reg2 >> 1) ^w_reg2;	

assign full =  ({~w_ptr_g[ptr_width:ptr_width-1],w_ptr_g[ptr_width-2:0]}== r_gray);	
assign emty =  (w_gray == r_ptr_g);
assign data_out = dout;
assign valid = valid_r;
endmodule


````

仿真结果
---
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019081111055682.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDI2MDAx,size_16,color_FFFFFF,t_70)

激励文件
---
````verilog
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: zhangsy
// 
// Create Date: 2019/08/05 13:03:14
// Design Name: 
// Module Name: simtxt
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


module simtxt();
reg w_clk,r_clk;
reg write,read;
reg rst_n;
reg [3:0] data_in;
wire  [3:0] data_out;
wire  valid;
wire  full;
wire  emty;
initial
begin
    w_clk = 0;
    r_clk = 0;
    rst_n = 0;
     #80
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
            write =0;
            
          read =1;
          #200;
          read = 0;
          #40
         write = 1;
             // #20
              data_in = 4'd6;
               #20
                 data_in = 4'd7;
                // read = 1;
                  #20
                    data_in = 4'd8;
                     #20
                       data_in = 4'd9;
                        read = 1;
           #200
                      //    write = 1;
                         
                         // #20read
                          data_in = 4'd5;
                           #20
                             data_in = 4'd5;
                              #20
                               read =0;
                                data_in = 4'd4;
                                 #20
                                   data_in = 4'd4;
                            
                             #20
                               write =0;
                             read =1;  
end			
always #10 w_clk = ~w_clk;
always #20 r_clk = ~r_clk;						
asynfifo asynfifo_inst(                                          
                .w_clk,
                .r_clk,
                .write,
                .read,
                .rst_n,
                .data_in,
                .data_out,
                .valid,
                .full,
                .emty
            );
endmodule

````
实际设计中需要注意的问题
---
>* 1考虑格雷码的对称性，在进行读写指针读取数据时应该去除最高位否则会影响fifo的读写功能。
>* 2在进行fifo状态判断时要注意时钟域的转换
>* 3fifo满状态的判断是将写指针的最高位与次高位取反，在与读指针进行判断，同理将读指针高两位取反判断亦可，具体参考博主程序设计
>* 4fifo空状态的判断为读写指针完全相等时即为空。
