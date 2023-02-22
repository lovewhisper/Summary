《Advanced Chip Design》读书笔记 第一部分
================================================================

第一章 绪论
--------------------------------

能够设计任何数字系统的Verilog语言

数字电路设计技术，包括状态机的设置、FIFO的设计、仲裁机制、同步技术等所有数字系统中都会涉及的常用电路设计技术。

芯片设计专业知识（ASIC/SoC），包括综合、时序分析、可测性设计等。

各种接口技术和接口协议知识，包括广泛应用于数字系统的PCI Express(PCIe)、SATA、USB、DDR等接口。

系统知识，如硬件和软件之间的互操作（操作系统、驱动程序和应用程序）、中断和DMA操作等。

第二章 寄存器传输语言（RTL）
--------------------------------

RTL的优点：支持综合工具；文本编辑工具；不容易出错。

可综合语法结构主要用于电路设计；全部Verilog语法结构可用于对所设计的电路进行仿真验证。

第三章 可综合的Verilog–用于电路设计
--------------------------------

Verilog的结构：层次化、模块化设计。

在数字系统中，所有的逻辑门都在同时进行着逻辑运算，每个时钟有效触发边沿出现时，所有的触发器都将进行输出值的更新，因此硬件电路是全并行执行的。

使用always组合块描述，变量被声明为reg类型

使用assign语句描述，变量被声明为wire类型

Verilog操作符::

    逻辑与&&
    按位与&
    逻辑或||
    按位或|
    逻辑非！
    按位非~
    按位异或^
    按位异或非~^
    缩位与&：矢量每个比特都参与操作，结果为1比特位宽
    缩位或|
    缩位异或^
    缩位与非~&
    缩位或非~|
    缩位异或非~^
    逻辑等于==
    逻辑不等于！=
    大于>大于等于>=小于<小于等于<=
    加+减-乘*除
    并位{}
    向左移位<<
    向右移位>>
    条件运算符?
    求模%
    指数运算**

参数化设计
定义::

    module asynch_fifo #(parameter FIFO_PTR = 4,
                                   FIFO_WIDTH = 32)
                        (
                            //Declare input & output ports
                                   );
    endmodule

例化::

    modeule chip_top(//Declare input & output ports);
    parameter FIFO_PTR_CHAN0=6,
              FIFO_WIDTH_CHAN0=32;
    parameter FIFO_PTR_CHAN1=8,
              FIFO_WIDTH_CHAN1=32;
    asynch_fifo #(.FIFO_PTR(FIFO_PTR_CHAN1),
                  .FIFO_WIDTH(FIFO_WIDTH_CHAN1))
                asynch_fifo_chan0
                (
                //Declare input & output ports
                );
    endmodule

Verilog函数

函数具有一个函数名、一个或多个输入信号以及一个输出信号。函数内部所描述的逻辑功能是可综合的，综合后得到的是组合逻辑，其内部不能出现与时间控制相关的语句，如wait,@,#::

    function [7:0] function_name;
        //declare input port;
        begin
            //comnbinational logic;
        end
    endfunction

    function_name(//input);

generate结构

综合工具在对代码进行编译时，会将电路展开从而生成完整的设计代码，然后对其进行电路综合::

    module gray_to_binary #(parameter PTR=6)
           (gray_value,
           binary_value);
    input [PTR:0] gray_value;
    output [PTR:0] binary_value;
    wire [PTR:0] binary_value;
    assign binary_value[PTR]=gray_value[PTR];
    generate
        genvar i;
        for(i=0;i<PTR;i=i+1)
            begin
                assign binary_value[i]=binary_value[i+1]^gray_value[i];
            end
    endgenerate
    endmodule

`ifdef::

    注意`ifdef与mux的不同

    `ifdef
    `else if
    `else
    `endif

第四章 用于验证的Verilog语法
--------------------------------
initial 语句

Verilog系统任务::

    $finish
    $stop
    $display
    $monitor
    $time
    $realtime
    $random/$random(seed)
    $save
    $readmemh
    $writememh
    $fopen
    $fclose

任务可以定义输出和输入端口，任务中可以进行定时控制。

实际应用中，，芯片内存储器是通过定制设计的SRAM实现的，不是通过综合生成的。Verilog中可以通过定义二维寄存器数组的方式定义存储器，所定义的存储器可以综合并采用寄存器阵列加以实际实现，主要用于验证。

while循环内，建议加入与定时控制有关的语句，避免形成定时环路，造成死循环::

    repeat
    repeat(5)
        begin
            @(posedge clk)
            $display("time display",$time);
        end

force 强制reg/wire复制

release 强制不变

fork-join内部的语句是并发执行的。

项目1 寄存器读写操作设计与验证
--------------------------------

下面是综合了之前学习的verilog知识的设计案例和验证案例，实现了寄存器读写操作设计与验证。

//device_regs_nofunction.v::

    module device_regs_nofunction(
        address,write_en,data_in,read_en,read_data,clk,resetb
    );

    input  [3:0] address;
    input        write_en;
    input        read_en;
    input  [7:0] data_in;
    output [7:0] read_data;
    input        clk;
    input        resetb;

    wire         reg0_match,reg1_match,reg2_match,reg3_match;
    reg    [7:0] reg0,reg1,reg2,reg3;
    wire   [7:0] reg0_nxt,reg1_nxt,reg2_nxt,reg3_nxt;
    reg    [7:0] read_data,read_data_nxt;

    assign reg0_match =(address==4'b0000);
    assign reg1_match =(address==4'b0001);
    assign reg2_match =(address==4'b0010);
    assign reg3_match =(address==4'b0011);
    assign reg0_nxt =(reg0_match && write_en)?data_in:reg0;
    assign reg1_nxt =(reg1_match && write_en)?data_in:reg1;
    assign reg2_nxt =(reg2_match && write_en)?data_in:reg2;
    assign reg3_nxt =(reg3_match && write_en)?data_in:reg3;

    always @(posedge clk or negedge resetb)
    begin
        if(!resetb)
            begin
                reg0<='d0;
                reg1<='d0;
                reg2<='d0;
                reg3<='d0;
                read_data<='d0;
            end
        else
            begin
                reg0<=reg0_nxt;
                reg1<=reg1_nxt;
                reg2<=reg2_nxt;
                reg3<=reg3_nxt;
                read_data<=read_data_nxt;
            end
    end

    always @(*)
    begin
        read_data_nxt =read_data;
        if(read_en)begin
            case(1'b1)
                reg0_match:read_data_nxt=reg0;
                reg1_match:read_data_nxt=reg1;
                reg2_match:read_data_nxt=reg2;
                reg3_match:read_data_nxt=reg3;
            endcase
        end
    end
    endmodule

\\device_regs_withfunction.v::

    module device_regs_withfunction(
        address,write_en,data_in,read_en,read_data,clk,resetb
    );

    input  [3:0] address;
    input        write_en;
    input        read_en;
    input  [7:0] data_in;
    output [7:0] read_data;
    input        clk;
    input        resetb;

    reg    [7:0] reg0,reg1,reg2,reg3;
    reg    [7:0] read_data,read_data_nxt;

    function [7:0] reg_nxt;
        input [3:0] address;
        input write_en;
        input [7:0] data_in;
        input [7:0] reg_temp;
        input [3:0] reg_address;
        begin
            assign reg_nxt =((address==reg_address) && write_en)?data_in:reg_temp;
        end
    endfunction

    always @(posedge clk or negedge resetb)
    begin
        if(!resetb)
            begin
                reg0<='d0;
                reg1<='d0;
                reg2<='d0;
                reg3<='d0;
                read_data<='d0;
            end
        else
            begin
                reg0<=reg_nxt(address,write_en,data_in,reg0,4'b0000);
                reg1<=reg_nxt(address,write_en,data_in,reg1,4'b0001);
                reg2<=reg_nxt(address,write_en,data_in,reg2,4'b0010);
                reg3<=reg_nxt(address,write_en,data_in,reg3,4'b0011);
                read_data<=read_data_nxt;
            end
    end

    always @(*)
    begin
        read_data_nxt =read_data;
        if(read_en)begin
            case(1'b1)
                (address==4'b0000) :read_data_nxt=reg0;
                (address==4'b0001) :read_data_nxt=reg1;
                (address==4'b0010) :read_data_nxt=reg2;
                (address==4'b0011) :read_data_nxt=reg3;
                endcase
        end
    end
    endmodule

\\testbench_top.v::

    module testbench_top();
    reg  [3:0] address_tb;
    reg  [7:0] wrdata_tb;
    reg        write_en_tb,read_en_tb;
    reg        clk_tb,resetb_tb;
    wire [7:0] rddata_tb;

    parameter CLKTB_HALF_PERIOD = 5;
    parameter RST_DEASSERT_DLY  = 100;
    parameter REG0_OFFSET = 4'b0000;
    parameter REG1_OFFSET = 4'b0001;
    parameter REG2_OFFSET = 4'b0010;
    parameter REG3_OFFSET = 4'b0011;

    initial begin
                               clk_tb=1'b0;
        forever begin
            #CLKTB_HALF_PERIOD clk_tb=~clk_tb;
        end
    end
    initial begin
                          resetb_tb = 1'b0;
        #RST_DEASSERT_DLY resetb_tb = 1'b1;
    end

    initial begin
        address_tb ='h0;
        wrdata_tb  ='h0;
        write_en_tb=1'b0;
        read_en_tb =1'b0;
    end

    device_regs_nofunction device_regs_withfunction_0(
        .clk(clk_tb),
        .resetb(resetb_tb),
        .address(address_tb),
        .write_en(write_en_tb),
        .read_en(read_en_tb),
        .data_in(wrdata_tb),
        .read_data(rddata_tb)
    );

    task reg_write;
        input [3:0] address_in;
        input [7:0] data_in;

        begin
            @(posedge clk_tb);
                #1 address_tb =address_in;
            @(posedge clk_tb);
                #1 write_en_tb = 1'b1;
                wrdata_tb=data_in;
            @(posedge clk_tb);
                #1
                write_en_tb =1'b0;
                address_tb  =4'hF;
                wrdata_tb   =4'h0;
        end
    endtask

    task reg_read;
        input [3:0] address_in;
        input [7:0] expected_data;

        begin
            @(posedge clk_tb);
                #1 address_tb =address_in;
            @(posedge clk_tb);
                #1 read_en_tb = 1'b1;
            @(posedge clk_tb);
                #1 read_en_tb = 1'b0;
                address_tb=4'hF;
            @(posedge clk_tb);
            if(expected_data===rddata_tb)
                $display("data matches:expected_data=%h,actual_data=%h",expected_data,rddata_tb);
            else
                $display("Error:data mismatch:expected_data=%h,actual_data=%h",expected_data,rddata_tb);
        end
    endtask

    initial begin
        #1000;
            reg_write(REG0_OFFSET,8'hA5);
            reg_read(REG0_OFFSET,8'hA5);
            reg_write(REG1_OFFSET,8'hA6);
            reg_read(REG1_OFFSET,8'hA6);
            reg_write(REG2_OFFSET,8'hA7);
            reg_read(REG2_OFFSET,8'hA7);
            reg_write(REG3_OFFSET,8'hA8);
            reg_read(REG3_OFFSET,8'hA8);
            $finish;
    end

    `ifdef FSDB
    initial begin
        $fsdbDumpfile("tb_counter.fsdb");
        $fsdbDumpvars;
    end
    `endif

    endmodule
