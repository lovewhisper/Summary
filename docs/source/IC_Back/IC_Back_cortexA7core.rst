cortexA7core 实战笔记
================================

一、设计导入
------------------------------------------------------

1.启动innnovus

    >>> innnovus -log import_design

2.加载设计文件

    具体操作：GUI界面菜单栏File-Import Design

    golbals文件详述：

        - Netlist

        - LEF Files

        - Power Nets

        - Ground Nets

        - MMMC View Definition Files

    globals文件::

        缺

脚本执行:

    >>> source -echo -verbose ../scripts/cortexa7core.globals

    >>> init_design

    >>> saveDesign ../save/init_design.enc

Tips:

    - 关闭图形界面

        >>> win off

    - 快捷键F适中显示

    - 快捷键shift+z 和 z 缩放显示

二、FloorPlan
------------------------------------------------------

1.重新打开innovus，恢复设计

2.定义floorplan大小

3.摆放memory

    GUI界面菜单栏-Floorplan-Floorplan Toolbox

4.摆放port

    GUI界面菜单栏-Edit-Pin Editor

5.加halo

6.加EndCap/TapCell



脚本执行:

    >>> restoreDesign ../save/init_design.enc.dat cortexa7core

    >>> floorplan -site core -s 700 840 1 1 1 1

    >>> defIn ../input/cortexa7core.def

    >>> deselectAll

    >>> select_obj [dbGet top.insts.cell.subclass block -p2]

    >>> writeFPlanScript -selected -fileName marco.tcl

    >>> saveIoFile io_port.loc

    >>> loadIoFile -noAdjustDieSize io_port.loc



Tips:

    - 快捷键k进入尺子模式

    - 快捷键shift+k清楚尺子

    - 飞线显示：右侧Flight Line显示，菜单栏View-Set Preferences-Flightline-Show Both input Pins Connection

    - 保存历史命令行

        >>> history keep 100000

三、PowerPlan
------------------------------------------------------

1.逻辑链接

2.加PowerStripe

脚本执行：

Tips：