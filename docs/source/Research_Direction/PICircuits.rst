可编程光子集成链路PhotoCAD实践
========================================

设计思路：

从下至上，从左至右

基础模块分为横向和纵向

M*N网络构成基础模块

- 横向模块(X_ij)：M行N个
- 纵向模块(Y_ij)：（M-1）行（N+1）个

M*N网络单个网络连接方式

- 左下：X_ij     1  与  Y_ij     4
- 左上：X_(i+1)j 2  与  Y_ij     1
- 右下：X_ij     3  与  Y_i(j+1) 3
- 右上：X_(i+1)j 4  与  Y_i(j+1) 2

具体代码如下::

备注::

    progcuit[f"{0},{i},{j}"] 横向模块

    progcuit[f"{1},{i},{j}"] 纵向模块

    progcuit[f"{2},{0},{j}"] 纵向模块右上直波导

    progcuit[f"{2},{1},{j}"] 纵向模块右下直波导

    progcuit[f"{3},{0},{j}"] 纵向模块左上直波导

    progcuit[f"{3},{1},{j}"] 纵向模块左下直波导

    progcuit[f"{4},{0},{j}"] 横向模块左下直波导

    progcuit[f"{4},{1},{j}"] 横向模块右下直波导

    progcuit[f"{5},{0},{j}"] 横向模块左上直波导

    progcuit[f"{5},{1},{j}"] 横向模块右下直波导

PICircuits.py::

    from dataclasses import dataclass
    from typing import Mapping, cast

    from fnpcell import all as fp
    from gpdk.technology import get_technology
    from ring_modulator import RingModulator
    from h_fanout import HFanout
    from gpdk.technology.waveguide_factory import EulerBendFactory
    from gpdk.technology.waveguide_factory import StraightFactory
    from gpdk import all as pdk


    def bend_factories(waveguide_type: fp.IWaveguideType):
        if waveguide_type == TECH.WG.FWG.C.WIRE:
            return EulerBendFactory(radius_min=16, l_max=18, waveguide_type=waveguide_type)
        return waveguide_type.bend_factory

    @dataclass(eq=False)
    class PICircuit(fp.PCell):
        def build(self):
            global spacing, row_number, column_number
            insts, elems, ports = super().build()
            TECH = get_technology()

            spacing =500
            row_number = 10
            column_number = 12

            basic_comp = RingModulator()
            basic_comp_y = RingModulator(transform=fp.rotate(degrees=90))

            xy_spacing = []
            xx_spacing = []
            yy_spacing = []
            yx_spacing = []

            for i in range(row_number + 1):
                xy_spacing.append(spacing * i)

            for i in range(column_number):
                xx_spacing.append(spacing * (i + 0.5))

            for i in range(row_number):
                yy_spacing.append(spacing * (i + 0.5))

            for i in range(column_number + 1):
                yx_spacing.append(spacing * i)

            x_straight_top_right = []
            x_straight_btm_right = []
            x_straight_top_left = []
            x_straight_btm_left = []
            straight_index_min_x = 0
            straight_index_max_x = 0
            straight_index_min_xt = 0
            straight_index_max_xt = 0

            for i in range(row_number + 1):
                for j in range(column_number):
                    x = xx_spacing[j]
                    y = xy_spacing[i]
                    x_dcoupler = basic_comp["op_0"].repositioned(at=(x, y)).owner
                    insts += x_dcoupler, f"{0},{i},{j}"
                    if (i == 0) and (j == 0):
                        get_position_y = basic_comp["op_1"].position[y]
                        x_straight_btm_left.append(pdk.Straight(name = "xbl_straight"+str(straight_index_min_x) ,length=20, waveguide_type=TECH.WG.FWG.C.WIRE)["op_0"].repositioned(at=(x-30-0.5*spacing, get_position_y)).owner)
                        insts += x_straight_btm_left[straight_index_min_x], f"{4},{0},{straight_index_min_x}"
                        straight_index_min_x = straight_index_min_x+1
                    if (i == 0) and (j == (column_number-1)):
                        get_position_y = basic_comp["op_1"].position[y]
                        x_straight_btm_right.append(pdk.Straight(name = "xbr_straight"+str(straight_index_max_x) ,length=20, waveguide_type=TECH.WG.FWG.C.WIRE)["op_0"].repositioned(at=(x+60+0.5*spacing, get_position_y)).owner)
                        insts += x_straight_btm_right[straight_index_max_x], f"{4},{1},{straight_index_max_x}"
                        straight_index_max_x = straight_index_max_x+1
                    if (i == row_number) and (j == 0):
                        x_straight_top_left.append(pdk.Straight(name = "xtl_straight"+str(straight_index_min_xt) ,length=20, waveguide_type=TECH.WG.FWG.C.WIRE)["op_0"].repositioned(at=(x-30-0.5*spacing, y)).owner)
                        insts += x_straight_top_left[straight_index_min_xt], f"{5},{0},{straight_index_min_xt}"
                        straight_index_min_xt = straight_index_min_xt+1
                    if (i == row_number) and (j == (column_number-1)):
                        x_straight_top_right.append(pdk.Straight(name = "xtr_straight"+str(straight_index_max_xt) ,length=20, waveguide_type=TECH.WG.FWG.C.WIRE)["op_0"].repositioned(at=(x+60+0.5*spacing,y)).owner)
                        insts += x_straight_top_right[straight_index_max_xt], f"{5},{1},{straight_index_max_xt}"
                        straight_index_max_xt = straight_index_max_xt+1


            y_straight_top_right = []
            y_straight_btm_right = []
            y_straight_top_left = []
            y_straight_btm_left = []
            straight_index_min = 0
            straight_index_max = 0

            for i in range(row_number):
                for j in range(column_number + 1):
                    x = yx_spacing[j]
                    y = yy_spacing[i]
                    y_dcoupler = basic_comp_y["op_0"].repositioned(at=(x, y)).owner
                    insts += y_dcoupler, f"{1},{i},{j}"
                    if (j == 0) :
                        y_straight_top_right.append(pdk.Straight(name = "tr_straight"+str(straight_index_min) ,length=20, waveguide_type=TECH.WG.FWG.C.WIRE)["op_0"].repositioned(at=(x-30, y+60)).owner)
                        y_straight_btm_right.append(pdk.Straight(name = "br_straight"+str(straight_index_min) ,length=20, waveguide_type=TECH.WG.FWG.C.WIRE)["op_0"].repositioned(at=(x-30, y-10)).owner)
                        insts += y_straight_top_right[straight_index_min], f"{2},{0},{straight_index_min}"
                        insts += y_straight_btm_right[straight_index_min], f"{2},{1},{straight_index_min}"
                        straight_index_min = straight_index_min+1
                    if (j == (column_number)) :
                        y_straight_top_left.append(pdk.Straight(name = "tl_straight"+str(straight_index_max) ,length=10, waveguide_type=TECH.WG.FWG.C.WIRE)["op_0"].repositioned(at=(x+60, y+60)).owner)
                        y_straight_btm_left.append(pdk.Straight(name = "bl_straight"+str(straight_index_max) ,length=10, waveguide_type=TECH.WG.FWG.C.WIRE)["op_0"].repositioned(at=(x+60, y-10)).owner)
                        insts += y_straight_top_left[straight_index_max], f"{3},{0},{straight_index_max}"
                        insts += y_straight_btm_left[straight_index_max], f"{3},{1},{straight_index_max}"
                        straight_index_max = straight_index_max+1

            progcuit = cast(Mapping[str, fp.ICellRef], insts)
            # p = progcuit["0,1,3"]["op_2"].position[0]

            for i in range(row_number + 1):
                for j in range(column_number):
                    if i == 0 and (j < column_number - 1):
                        link1 = fp.LinkBetween(start=progcuit[f"{0},{i},{j}"]["op_2"],
                                               end=progcuit[f"{0},{i},{j + 1}"]["op_1"],
                                               bend_factory=EulerBendFactory(radius_min=5, l_max=5, waveguide_type=TECH.WG.FWG.C.WIRE))
                        insts += link1
                    if i == row_number and (j < column_number - 1):
                        link2 = fp.LinkBetween(start=progcuit[f"{0},{i},{j}"]["op_3"],
                                               end=progcuit[f"{0},{i},{j + 1}"]["op_0"],
                                               bend_factory=EulerBendFactory(radius_min=5, l_max=15, waveguide_type=TECH.WG.FWG.C.WIRE))
                        insts += link2
                    if (i < row_number) and (j < column_number):
                        link3 = fp.LinkBetween(start=progcuit[f"{0},{i},{j}"]["op_0"],
                                               end=progcuit[f"{1},{i},{j}"]["op_1"],
                                               bend_factory=EulerBendFactory(radius_min=5, l_max=15, waveguide_type=TECH.WG.FWG.C.WIRE))
                        insts += link3
                        link4 = fp.LinkBetween(start=progcuit[f"{0},{i},{j}"]["op_3"],
                                               end=progcuit[f"{1},{i},{j + 1}"]["op_0"],
                                               bend_factory=EulerBendFactory(radius_min=15, l_max=15, waveguide_type=TECH.WG.FWG.C.WIRE))
                        insts += link4
                        link5 = fp.LinkBetween(start=progcuit[f"{0},{i + 1},{j}"]["op_1"],
                                               end=progcuit[f"{1},{i},{j}"]["op_2"],
                                               bend_factory=EulerBendFactory(radius_min=15, l_max=15, waveguide_type=TECH.WG.FWG.C.WIRE))
                        insts += link5
                        link6 = fp.LinkBetween(start=progcuit[f"{0},{i + 1},{j}"]["op_2"],
                                               end=progcuit[f"{1},{i},{j + 1}"]["op_3"],
                                               bend_factory=EulerBendFactory(radius_min=15, l_max=15, waveguide_type=TECH.WG.FWG.C.WIRE))
                        insts += link6
            n = 0
            link7 = fp.LinkBetween(start=progcuit[f"{0},{0},{column_number-1}"]["op_2"],
                                   end=progcuit[f"{4},{1},{0}"]["op_0"],
                                   bend_factory=TECH.WG.FWG.C.WIRE.BEND_CIRCULAR)
            insts += link7
            ports += progcuit[f"{4},{1},{0}"]["op_1"].with_name("op_"+str(n))

            for i in range(row_number):
                link8 = fp.LinkBetween(start=progcuit[f"1,{i},{column_number}"]["op_1"],
                                       end=progcuit[f"{3},{1},{i}"]["op_0"],
                                       bend_factory=EulerBendFactory(radius_min=15, l_max=15, waveguide_type=TECH.WG.FWG.C.WIRE))
                insts += link8
                n = n + 1
                ports += progcuit[f"{3},{1},{i}"]["op_1"].with_name("op_"+str(n))
                link9 = fp.LinkBetween(start=progcuit[f"1,{i},{column_number}"]["op_2"],
                                       end=progcuit[f"{3},{0},{i}"]["op_0"],
                                       bend_factory=EulerBendFactory(radius_min=15, l_max=15, waveguide_type=TECH.WG.FWG.C.WIRE))
                insts += link9
                n = n + 1
                ports += progcuit[f"{3},{0},{i}"]["op_1"].with_name("op_"+str(n))

            n = n + 1
            link10 = fp.LinkBetween(start=progcuit[f"{0},{row_number},{column_number-1}"]["op_3"],
                                   end=progcuit[f"{5},{1},{0}"]["op_0"],
                                   bend_factory=EulerBendFactory(radius_min=15, l_max=15, waveguide_type=TECH.WG.FWG.C.WIRE))
            insts += link10
            ports += progcuit[f"{5},{1},{0}"]["op_1"].with_name("op_"+str(n))

            n = n+1
            link10 = fp.LinkBetween(start=progcuit[f"{0},{row_number},0"]["op_0"],
                                   end=progcuit[f"{5},{0},{0}"]["op_1"],
                                   bend_factory=EulerBendFactory(radius_min=15, l_max=15, waveguide_type=TECH.WG.FWG.C.WIRE))
            insts += link10
            ports += progcuit[f"{5},{0},{0}"]["op_0"].with_name("op_"+str(n))

            for i in range(row_number-1,-1,-1):
                n = n + 1
                link11 = fp.LinkBetween(start=progcuit[f"1,{i},0"]["op_3"],
                                        end=progcuit[f"{2},{0},{i}"]["op_1"],
                                        bend_factory=EulerBendFactory(radius_min=15, l_max=15,
                                                                      waveguide_type=TECH.WG.FWG.C.WIRE))
                insts += link11
                ports += progcuit[f"{2},{0},{i}"]["op_0"].with_name("op_"+str(n))
                n = n + 1
                link12 = fp.LinkBetween(start=progcuit[f"1,{i},0"]["op_0"],
                                        end=progcuit[f"{2},{1},{i}"]["op_1"],
                                        bend_factory=EulerBendFactory(radius_min=15, l_max=15,
                                                                      waveguide_type=TECH.WG.FWG.C.WIRE))
                insts += link12
                ports += progcuit[f"{2},{1},{i}"]["op_0"].with_name("op_"+str(n))

            n = n+1
            link13 = fp.LinkBetween(start=progcuit["0,0,0"]["op_1"],
                                   end=progcuit[f"{4},{0},{0}"]["op_1"],
                                   bend_factory=TECH.WG.FWG.C.WIRE.BEND_CIRCULAR)
            insts += link13
            ports += progcuit[f"{4},{0},{0}"]["op_0"].with_name("op_"+str(n))

            # fmt: on
            return insts, elems, ports


    if __name__ == "__main__":
        from pathlib import Path

        gds_file = Path(__file__).parent / "local" / Path(__file__).with_suffix(".gds").name
        library = fp.Library()

        TECH = get_technology()
        # =============================================================
        # fmt: off
        library += [
                HFanout(name="PIC",
                        device=PICircuit(),
                        left_spacing=spacing/2,
                        right_spacing=spacing/2,
                        left_distance =spacing/3,
                        right_distance=spacing/3,
                        bend_factories=bend_factories,
                        left_waveguide_type=TECH.WG.SWG.C.WIRE,
                        right_waveguide_type=TECH.WG.SWG.C.WIRE)
            ]
        # library = PICircuit()

        # fmt: on
        # =============================================================
        fp.export_gds(library, file=gds_file)
        fp.plot(library)

