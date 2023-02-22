Makefile文件编写案例
================================

1 Makefile案例
--------------------------------

执行make默认执行comp::

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

2 文件详细解读
--------------------------------

2.1 Makefile的基本格式
>>>>>>>>>>>>>>>>>>>>>>>>>>>

Makefile的基本格式::

    target1 ...: prerequisites ...
        command   # 注意在命令前面需要添加一个Tab键
            ...
    target2 ...: prerequisites ...
        command
            ...

2.2 Makefile变量引用
>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

    # 定义变量1
    变量1 = 文件或者

    # 命令行中引用变量1
    target1 :
        command $(变量1)

2.2 vcs编译仿真命令解读
>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

        vcs\
    # -R表示自动运行仿真
        -R\
        -full64\
    # +v2k表示支持 verilog 2001 标准
        +v2k\
    # 文件前加FSDB定义并产生fsdb波形文件
        -fsdb +define+FSDB\
    # 支持system Verilog
        -sverilog\
    # 文件列表亦可以 -v列出所有v文件
        -f $(filelist)\
    # 生成编译仿真log文件
        -l run.log

2.3 verdi波形查看命令解读
>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

        verdi\
    #文件列表
        -f $(filelist)\
    #加载波形
        -ssf $(fsdbfile) &
