《Advanced Chip Design》读书笔记 第二部分
================================================================

第五章 数字电路设计——初级篇
--------------------------------

n型晶体管生长在p型衬底上；p型晶体管生长在n型衬底上。

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
