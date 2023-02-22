IC后端设计学习总结
================================

.. toctree::

    IC_Summary_DesignFlow
    IC_Definition_Summary
    IC_Instruction_Summary
    IC_QuickOperate_Summary
    IC_Question

2022年12月26日开始学习 Innovus

Innovus 学习笔记
--------------------------------

启动innovus终端输入指令::


    innovus -log project_name

文件夹会生成project_name.cmd、project_name.log、project_name.logv文件。

- cmd文件记录指令；

- log文件记录详细运行过程与指令；

- logv文件在log文件基础上记录了时间。

------------------------------------------------

::

    innovus -log import_design

操作方式：GUI界面操作以及终端指令操作

设计导入：File→import design→点击下方Load选择.globals文件→点击OK完成导入。

.. note::

    ！！！.global文件

界面切换：physical view(place、CTS、route)/floorplan view(floorplan、powerplan)

**快捷键 F 设计显示在中间**

查看Cell的Pin：physical view→Cell→Pin Shapes勾选

放大和缩小界面：工具栏、鼠标滚轮、快捷键（z放大shift+z缩小）

查看设计子模块：选中模块→点击右键→选择Ungroup

蓝色飞线：module与module、IO、marco之间的连接关系，取消显示：Miscellameous→Flight Line

Design Browser工具栏启动、菜单栏Tools→Design Browser，设计规模

**快捷键K尺子模式、复制选中后按C、选中对象按Q弹出属性、移动（shift+R）、快捷键A Selected By Box模式**

------------------------------------------------

固定显示菜单栏：打开菜单栏，单击最上面虚线固定菜单栏

清除floorplan:菜单栏floorplan→clear floorplan。不建议使用，推荐重新导入设计。

------------------------------------------------

定义芯片尺寸：floorplan→specify floorplan→Aspect Ratio、Core to IO Boundary

::

    checkDesign -netlist

检查结果存放在./checkDesign文件夹下，文件.ascii记录了详细信息

指令帮助::

    man checkDesign(按q键推出)
    checkDesgin -help(精简版)

------------------------------------------------

导入参考布局：菜单栏File→Load→Floorplan→dtmf_blocks.fp→open

选中Marco按Q可以查看Placement Halo属性

选中Marco右击弹出菜单，选择Edit Halo

添加Routing blockage：工具栏或者快捷键（shift+B）进入（Create Routing Blockage）模式→按F3弹出菜单。

添加Placemen Blockage：工具栏或者快捷键（shift+Y）进入（Create Placement Blockage）模式→按F3弹出菜单。Hard区域任何cell不可放置，Soft区域可放置opt design的cell。

保存floorplan:菜单栏File→Save→Floorplan→Save

------------------------------------------------

加入Power Ring：菜单栏Power→Power Planning→Add Ring→配置(Marco Power Ring的延申)→OK

加入Power Stripe：菜单栏Power→Power Planning→Add Stripe→配置→OK

GUI摆放cell(Design Browser找到选择位置摆放)。代码摆放一个cell::

    placeInstance DTMF_INST/TDSP_CORE_INST/TDSP_CORE_GLUE_INST/i_09007 600 700

打follow pin：Route→Special Route→配置（Basic和Via Generation）→OK

------------------------------------------------

获取Place所有操作::

    getPlaceMode

读取def文件::

    defIn scan_input.def

.. note::

        def文件作用

优化设计::

    place_opt_design

跑完之后切换到Physical View分析拥塞程度

生成scan.def文件::

    defOutBySection -noNets -noComps -scanChains scan.def

显示扫描链：Place→Display→Scan Chain，工具栏取消勾选route

取消显示：Place→Display→Scan Chain→Clear

显示Congestion：工具栏Overlay→Congestion

place_opt_design之后查看Routing Overflow H和V均小于1%走线没有问题

代码保存设计::

    saveDesign project_name.inn .inv .enc

GUI保存：File→Save Design→Innovus→OK

------------------------------------------------

Early Global Route：route→Early Global Route→勾选Routing Layer→设置Min、Max

保存设计指令::

    saveDesign ../saved/project_name.enc

恢复设计指令::

    restoreDesign ../saved/project_name.enc.dat/ DTMF_CHIP （顶层设计名）

GUI界面：File→Restore Design→Innovus→选择保存的文件→OK

选中一条走线selectNet DTMF_INST/RESULTS_CONV_INST/n_1310

取消Route勾选后可以看见走线。●代表走线的起点，×代表走线的终点。

取消选中走线::

    deselectAll

------------------------------------------------

提取RC参数：菜单栏Timing→Extract RC

输出延时文件：菜单栏Timing→Write SDF→勾选Ideal Clock以及Active View并选择dtmf_view_setup(未做CTS的默认选项)→OK

生成Timing报告：菜单栏Timing→Report Timing

Debug Timing：Timing→Debug Timing

展示hold time的情况：终端运行::

    set_analysis_view -setup {dtmf_view_setup} -hold {dtmf_view_hold}→Timing→Report Timing→设置成Hold→Timing→Debug Timing→Report File切换成hold

------------------------------------------------

CTS时钟树综合

读取文件::

    source dtmf.ccopt

生成spec文件::

    create_ccopt_clock_tree_spec

时钟树综合::

    ccopt_design

分析时钟树质量：Clock→Ccopt Clock Tree Debugger→OK→显示CTD界面

CTD界面介绍：红色方块、刻度、Visibility→Clock tree选择需要分析的时钟树

时序分析：Timing→Report Timing→Post-CTS→hold、setup分别报告

时序分析后opt界面操作：菜单栏ECO→Optimization Design→Post-CTS

------------------------------------------------

（了解）产生合适的RC系数::

    generateRCFactor -preroute true -postroute medium -reference externalSpef -spefMapFile spef.map

------------------------------------------------

给net设置特殊的space和shielding::

    setAttribute -net DTMF_INST/TDSP_CORE_INST/read_data -shield_net VDD

查看是否设计成功::

    getAttribute -net DTMF_INST/TDSP_CORE_INST/read_data

设置两倍space::

    setAttribute -net DTMF_INST/clk -preferred_extra_space 2

选择线布：Route→NanoRoute→Route→勾选Timing Driven以及SI Driven

自动布线：Route→NanoRoute→Route→勾选Timing Driven以及SI Driven，建议勾选Optimize Via以及Optimize Wire

DRC检查：工具栏Violation Browser

设置OCV：setAnalysisMode -analysisType onChipVariation

Timing→Report Timing→PostRoute→setup/hold→OK

opt：ECO→Optimize Design→Setup/Hold→Max Fanout→opt直至符合要求→保存设计

设计流程：floorplan→powerplan→place→CTS→route

------------------------------------------------

LAB16-1操作步骤（修复DRC的小技巧，了解）

工作目录：FPR/work/EDIT_ROUTE

restoreDesign EditRoute.dat/ DTMF_CHIP

切换到physical view

选中IO cell：select_obj IOPADS_INST/Prefclkip

工具栏Cell→Pin Shapes勾选（v:可见s:可选）

选中Pin终端输入：dbGet selected.net.name获取net名字

点击画线图标，键入E，弹出窗口中输入net名

单击Pin开始画线，至目标Pin后鼠标双击完成画线。

更换通孔：Layer只显示通孔，选中通孔，Replace Via

切线：菜单栏cut wire by line，鼠标切割需要切线位置，菜单栏选择Move可以移动切线，复制切线，旋转切线（左侧工具栏），选中旋转后的切线，键入S，延长切线。

更换走线层次：选中线，终端输入：editChangeLayer -layer_vertical M3

修改线宽：选中线，终端输入：editChangeWidth -width_vertical 0.44

------------------------------------------------

LAB19(DRC检查)

工作目录：FPR/work/VERIFY

导入globals文件，导入def文件，

检查设计环路：verifyConnectivity -geomLoop

定位错误，删除错误

锁定区域：zoomTo 300 610

查看锁定区域DRC：clearDrc，verify drc -view window

全局DRC：verify drc