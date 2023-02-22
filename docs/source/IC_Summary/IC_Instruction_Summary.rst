IC后端设计指令总结
================================


启动innovus终端输入指令::

    innovus -log project_name

指令帮助::

    man checkDesign(按q键推出)
    checkDesgin -help(精简版)

GUI摆放cell(Design Browser找到选择位置摆放)。代码摆放一个cell::

    placeInstance DTMF_INST/TDSP_CORE_INST/TDSP_CORE_GLUE_INST/i_09007 600 700

获取Place所有操作::

    getPlaceMode

读取def文件::

    defIn scan_input.def

优化设计::

    place_opt_design

生成scan.def文件::

    defOutBySection -noNets -noComps -scanChains scan.def

保存设计指令::

    saveDesign ../saved/project_name.enc

恢复设计指令::

    restoreDesign ../saved/project_name.enc.dat/ DTMF_CHIP （顶层设计名）

取消选中走线::

    deselectAll

展示hold time的情况：终端运行::

    set_analysis_view -setup {dtmf_view_setup} -hold {dtmf_view_hold}→Timing→Report Timing→设置成Hold→Timing→Debug Timing→Report File切换成hold

读取文件::

    source dtmf.ccopt

生成spec文件::

    create_ccopt_clock_tree_spec

时钟树综合::

    ccopt_design

产生合适的RC系数::

    generateRCFactor -preroute true -postroute medium -reference externalSpef -spefMapFile spef.map

------------------------------------------------

给net设置特殊的space和shielding::

    setAttribute -net DTMF_INST/TDSP_CORE_INST/read_data -shield_net VDD

查看是否设计成功::

    getAttribute -net DTMF_INST/TDSP_CORE_INST/read_data

设置两倍space::

    setAttribute -net DTMF_INST/clk -preferred_extra_space 2