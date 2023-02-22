从零基础入门vcs编译工具
================================

1 前言
--------------------------------

作为一个零基础项目，电路结构特别简单，就是一个按键控制一个灯，按下按键灯亮，松开按键灯灭。
主要包括以下文件：

- led.v
- tb_led.v
- file.f
- Makefile

2 RTL设计代码编写
--------------------------------

首先完成RTL设计，即输入控制输出，简单的逻辑组合即可实现::

    module led(
        input wire key_in,
        output wire led_out
    );
    assign led_out = key_in;
    endmodule

3 TestBench测试代码编写
--------------------------------

测试脚本的编写，测试脚本作为RTL设计的激励::

    module tb_led();
    reg key_in;
    wire led_out;
    #初始化信号
    initial begin
        key_in<=1'b0;
        repeat(20) begin
        #10 key_in<={$random}%2;
        end
    end
    #实例化RTL设计连线
    led led_inst(
        .key_in(key_in),
        .led_out(led_out)
    );
    # 用于产生fsdb波形
    `ifdef FSDB
        initial begin
            $fsdbDumpfile("tb.fsdb");
            $fsdbDumpvars;
        end
    `endif
    endmodule

4 file.f文件集合
--------------------------------

- led.v
- tb_led.v

5 Makefile编写
--------------------------------
::

    filelist  = file.f
    fsdnbfile = tb.fsdb

    comp : clean vcs

    vcs  :
        vcs -R -full64 +v2k\
        -fsdb +define+FSDB\
        -sverilog\
        -f $(filelist)\
        -l run.log

    ## verdi:
        verdi -f $(filelist)\
        -ssf $(fsdbfile) &

    clean:
        rm -rf *~\ *.log csrc\
        simv*\
        ucli.key\
        *.fsdb*

具体参照《Makefile文件编写案例》(:doc:`IC_Summary_Verilog_1`)

6 vcs编译仿真
--------------------------------

终端输入进行编译仿真::

    make

7 Verdi波形查看
--------------------------------

终端输入进入verdi,点击左侧顶层设计，按ctrl+4加入所有波形，波形区域点击f查看全部波形::

    make verdi

8 总结
--------------------------------

如果不编写Makefile,亦可以在终端输入以下命令直接进行仿真编译、查看波形::

    vcs -R -full64 +v2k -fsdb +define+FSDB -sverilog -f file.f -l run.log
    vcs -R -full64 +v2k -fsdb +define+FSDB -sverilog -v led.v tb_led.v -l run.log
    verdi -f file.f -ssf $(fsdbfile) &
    verdi -v led.v tb_led.v -ssf $(fsdbfile) &