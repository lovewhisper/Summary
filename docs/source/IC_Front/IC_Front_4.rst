《Advanced Chip Design》读书笔记 第二部分
================================================================

第五章 数字电路设计——初级篇
--------------------------------

n型晶体管生长在p型衬底上；p型晶体管生长在n型衬底上。

反相器是一个p型晶体管与一个n型晶体管串联而成的。输出端：p型晶体管的漏极和n型晶体管的源极连接在一起。输入端：栅极连接在一起。p型晶体管的源极被连接到Vdd，n型晶体管的漏极连接到GND。

在实际应用中，一般使用与非门、或非门。主要原因是采用CMOS工艺实现的与非门、或非门工作速度比与门和或门快得多。与非门和或非门被称为通用逻辑门，任何逻辑门都可以通过这两种门来实现。一般输入端口最多为4个或5个。

缓冲门的输出逻辑值与输入相同，其功能是提供更大的驱动能力驱动多个负载。

复杂门电路有助于减少芯片面积，提高电路工作速度。

边沿检测
>>>>>>>>>>>>>>>>>>>>>>>>>>>

//1 同步上升沿检测::

    input sig_1;

    reg   sig_a_d1;
    wire  sig_a_risedge;

    always@(posedge clk or negedge rstb)
    begin
        if(!rstb)
            sig_a_d1<=1'b0;
        else
            sig_a_d1<=sig_a;
    end

    assign sig_a_risedge = sig_a & !sig_a_d1;

//2 同步下降沿检测::

    input sig_1;

    reg   sig_a_d1;
    wire  sig_a_faledge;

    always@(posedge clk or negedge rstb)
    begin
        if(!rstb)
            sig_a_d1<=1'b0;
        else
            sig_a_d1<=sig_a;
    end

    assign sig_a_faledge = !sig_a & sig_a_d1;

//3 同步上升下降沿检测::

    input sig_1;

    reg   sig_a_d1;
    wire  sig_a_faledge;

    always@(posedge clk or negedge rstb)
    begin
        if(!rstb)
            sig_a_d1<=1'b0;
        else
            sig_a_d1<=sig_a;
    end

    assign sig_a_faledge = sig_a ^ sig_a_d1;

行波进位加法器
>>>>>>>>>>>>>>>>>>>>>>>>>>>

//无进位::

    sum_out   = a^b;
    carry_out = a&b;

//有进位::

    sum_out   = a^b^c;
    carry_out = (a & b)|((a ^ b) & c);

超前进位加法器
>>>>>>>>>>>>>>>>>>>>>>>>>>>

仅根据当前的输入得到每一级的进位输出，速度更快，资源消耗更少。

乘法可以通过移位相加或移位相减来实现。
