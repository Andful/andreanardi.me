---
title: Adding an External Clock to the KV260 Dev Board
slug: adding-an-external-clock-to-the-kv260-dev-board
description: The KV260 dev board is one of the cheepest FPGA dev board for the
  hardware-resources it offers, but it is deemed not user-friendly. In this blog
  post, I describe, somewhat of a satirical hack, to alleviate the
  user-unfriendliness of the board.
date: 2026-05-07T22:49:00.000+02:00
author: Andrea Nardi
thumbnail: /assets/media/2026-05-07-adding-an-external-clock-to-the-kv260-dev-board/installation.jpeg
draft: false
showToc: true
UseHugoToc: true
---
## Introduction

The  Kria KV260 Vision AI Starter Kit has great specs, given that the board only costs $249.00. The board has 117120 LUTs, 234240 FFs, and 23.06 Mb of on-chip memory (combining BRAM and URAM).

This makes the board, arguably, the most cost-effective board. For example, if we compare the LUTs per dollar, FFs per dollar, and on-chip memory per dollar, it is one of the cheapest boards out there, as it can be seen on {{< ref id="tab-cost-effective">}}.

| Board           | Cost\[$] | LUTs per dollar | FFs per dollar | memory per dollar\[Kb/$] |
| --------------- | -------- | --------------- | -------------- | ------------------------ |
| Kria KV260      | 249.00   | **470.36**      | **940.72**     | **94.84**                |
| Kria KR260      | 349.00   | 335.59          | 671.17         | 67.67                    |
| ULX3S(ECP5 85F) | 250.00   | 344.06          |                | 14.98                    |
| Tang Nano 9K    | 25.92    | 333.33          | 250.00         | 18.67                    |
{ id="tab-cost-effective" caption="Per board cost and hardware-resource per dollar" }

While it is one of the cheapest boards out there for what it offers, and while it offers, generally, a large amount of hardware resources, it is not a very beginner-friendly dev board[^not-beg1]. It is alleged that Xilinx did not envision the use of Vivado(the main FPGA development tool for Xilinx's FPGAs) with this board[^no-vivado1]. One of the reasons why the dev board is not user-friendly is that the programmable logic does not have direct access to a clock signal[^no-clock1][^no-clock3]. A clock signal is essential for an FPGA and the RTL(the design) that lies within it to function.

I fixed that by designing a PCB that directly provides a clock signal to the programmable logic. The PCB exploits the Raspberry Pi camera interface connector to provide the clock signal, as can be seen in {{< ref id="fig-installed" >}}. In this blog, I detail the [Design](#design), [Validation](#validation), and [Cost](#cost) of this PCB, to then provide my concluding remarks in the [Conclusion](#conclusion).

![Installed external clock PCB](/assets/media/2026-05-07-adding-an-external-clock-to-the-kv260-dev-board/installed.png)
{ id="fig-installed" }

## Design

The KV260 dev board does not have many peripherals accessible by the programmable logic. The only high-speed peripherals are the 3 MIPI camera interfaces. One of them has a Raspberry Pi camera interface connector. That connector is the largest, making it easier (but not easy) to work with by hand. Further, that connector is an edge-connector, for which the interfacing PCB would be slotted in. This saves a connector-type component from the PCB, reducing the components required, easing soldering, and reducing the bill of materials. That is why I chose the Raspberry Pi camera interface connector to interface the PCB and provide a clock signal to the programmable logic. The Raspberry Pi camera interface uses power and logic levels of 3.3V. Because we want to feed a clock signal, it is only fitting that we use the 2 signal traces for the differential clock signal. The power and the clock signal traces are shown in {{< ref id="fig-mipi-pins" >}}, for a Raspberry Pi Camera Module 2. The PCB design will require the same power and clock signal traces to interface with the KV260 dev board. 

![A Raspberry Pi Camera Module 2, and its power traces and differential clock traces](/assets/media/2026-05-07-adding-an-external-clock-to-the-kv260-dev-board/mipi_pins.png)
{ id="fig-mipi-pins" }

To send a differential clock signal, I decided to use a crystal oscillator, whose output is connected to an LVDS(low-voltage differential signaling) transceiver. The components I decided to use are respectively [EPSON X1G0048010002 (Crystal Oscillator)](https://www.lcsc.com/product-detail/C881723.html) and [RUNIC RS90LV011YF5 (LVDS Transceiver)](https://www.lcsc.com/product-detail/C22390261.html), for which I included a 10nF decoupling capacitor for each of the previously mentioned components.

The KiCad PCB design can be found on [GitHub](https://github.com/Andful/KV260-PL-External-Clock-PCB). A KiCad view of the PCB can be seen in {{< ref id="fig-kicad">}}.

![KiCad view of the PCB design file, with dimensions annotations](/assets/media/2026-05-07-adding-an-external-clock-to-the-kv260-dev-board/kicad.png)
{ id="fig-kicad" }

Manufacturing was done by JLCPCB, while the soldering was done by me, by hand, and reenacted in {{< ref id="fig-reflow">}}. The manufactured PCB can be found in {{< ref id="fig-pcb">}}. 

![PCB delivered by the manufacturer. On the left, an unpopulated PCB, and on the right, a populated PCB](/assets/media/2026-05-07-adding-an-external-clock-to-the-kv260-dev-board/pcb.jpeg)
{ id="fig-pcb" }

The PCB itself is very small and requires a steady hand to place the components. To give an idea of the size of the PCB, {{< ref id="fig-reflow">}} shows a reenactment of the reflow process, on a G3061 mini hot plate, which does not look so "mini" in comparison. For another point of size comparison, {{< ref id="size-comparison">}} shows the PCB next to a €2 coin.

![Reenactment of the reflow soldering process on a G3061 mini hot plate](/assets/media/2026-05-07-adding-an-external-clock-to-the-kv260-dev-board/reflow.jpg)
{ id="fig-reflow" }

![Size comparison of the PCB next to a €2 coin](/assets/media/2026-05-07-adding-an-external-clock-to-the-kv260-dev-board/size_comparison.jpg)
{ id="size-comparison" }

Anyway, given the assembled PCB, it would be interesting to see if it actually works. The next section validates whether the PCB performs its intended function.

## Validation

I do not think testing the PCB directly on the KV260 board would be a good idea, as I could damage something. Given the size of the PCB, ideally, I would have made a breakout board or some testing jig. Because I did not want to wait for the lead time of PCB manufacturing, I employed 4 hands (2 of my hands and 2 of my girlfriend's hands) to test the PCB. My girlfriend applied power with her two hands (ground with one hand and 3.3V with the other hand). I "measured" the clock signal with my two hands (ground with one hand and the measuring probe with the other). I saw what resembled a square wave, at roughly the expected frequency (of 10MHz), roughly at the expected voltage levels (of 0V and 3.3V). This made me confident enough to test the PCB on the KV260 dev board, knowing that it would most probably not break anything.

To ultimately find out if the design works correctly, I inserted the PCB in its intended connector, as shown in {{< ref id="fig-installation">}}.

![Installation of the external clock PCB on the Raspberry Pi camera interface connector of the KV260 Dev Board](/assets/media/2026-05-07-adding-an-external-clock-to-the-kv260-dev-board/installation.jpeg)
{ id="fig-installation" }

To test whether the PCB works, I made a simple FPGA design, comprised of the files `top.v` and `constraints.xdc`, to test the correct functioning of the design. The files can be found in the [`reference_design`](https://github.com/Andful/KV260-PL-External-Clock-PCB/tree/master/reference_design) folder of [the GitHub repo of the PCB](<>). But, for the convenience of the reader, the contents of `top.v` and `constraints.xdc` are included here:
{{% details summary="`top.v`" %}}

```verilog
module top(
    input clk_p,
    input clk_n,
    output [7:0] o
    );
    
    localparam Count = 10_000_000;
    
    wire clk;
    reg [31:0] count = 0;
    reg b = 1;
    
    assign o = {7'b0, b};
    
    IBUFDS #(
       .DIFF_TERM("TRUE"),
       .IBUF_LOW_PWR("TRUE"),
       .IOSTANDARD("DEFAULT")
    ) IBUFDS_inst (
       .O(clk),
       .I(clk_p),
       .IB(clk_n)
    );
    
    always @(posedge clk) begin
        if (count > Count - 1) begin
            count <= 0;
            b <= ~b;
        end else begin
            count <= count + 1;
        end
    end
endmodule
```

{{% /details %}}
{{% details summary="`constraints.xdc`" %}}

```text
set_property PACKAGE_PIN B11 [get_ports {o[7]}]
set_property PACKAGE_PIN D11 [get_ports {o[6]}]
set_property PACKAGE_PIN E12 [get_ports {o[5]}]
set_property PACKAGE_PIN B10 [get_ports {o[4]}]
set_property PACKAGE_PIN C11 [get_ports {o[3]}]
set_property PACKAGE_PIN D10 [get_ports {o[2]}]
set_property PACKAGE_PIN E10 [get_ports {o[1]}]
set_property PACKAGE_PIN H12 [get_ports {o[0]}]
set_property IOSTANDARD LVCMOS33 [get_ports {o[*]}]

create_clock -name SysClk -period 10 -waveform {0 5} [get_ports "clk_p"];
set_property PACKAGE_PIN D7      [get_ports "clk_p"];
set_property PACKAGE_PIN D6      [get_ports "clk_n"];

set_property IOSTANDARD LVDS [get_ports "clk_p"]
set_property IOSTANDARD LVDS [get_ports "clk_n"]
```

{{% /details %}}

The PCB, does indeed seem to work as intended. The system outputs the expected square wave with a period of 2 seconds, and pulse widths of 1 second, as shown in {{< ref id="fig-validation">}}.

![Validation of the clock PCB. The oscilloscope is measuring a square wave with a 2 second period, and a pulse width of 1 second. The expected, frequency divided, clock signal](/assets/media/2026-05-07-adding-an-external-clock-to-the-kv260-dev-board/validation.jpeg)
{ id="fig-validation"}

## Cost

The cost of the board is composed of the cost of its components, which I bought on LCSC, and the manufacturing of the PCB, done by JLCPCB. The manufacturing of the PCB does not include the cost of soldering, as I soldered the components by hand.

During the manufacturing of the PCB, I received an additional charge of $6.90, on top of the original cost of $16.00(discounted to $2 by coupon) for 5 PCBs(without shipping included). The following justification was given for the additional charge of $6.90:

> When the stiffener area is over 100% of the FPC dimension or independent stiffeners no less than 4 pcs per board, there will be an extra cost.

I don't fully understand the justification of the extra charge, as I would consider the singular stiffener's area to be 100%, but not over, the area of the FPC. Regardless, I will not count it over an idealized, amortized cost of the assembled PCB, presented in {{< ref id="tab-total-cost">}}. Overall, this price makes me appreciate the cheaper and more complex, assembled PCBs sold on AliExpress and Amazon.

| Item                                                                                         | Cost\[$]  |
| -------------------------------------------------------------------------------------------- | --------- |
| [Crystal Oscillator (EPSON X1G0048010002)](https://www.lcsc.com/product-detail/C881723.html) | 1.05      |
| [LVDS Transceiver (RUNIC RS90LV011YF5)](https://www.lcsc.com/product-detail/C22390261.html)  | 0.57      |
| [10uF capacitors](https://www.lcsc.com/product-detail/C315248.html) x2                       | $0.01 x2  |
| PCB fabrication (JLCPCB)                                                                     | $16.00 ÷5 |
| **Total**                                                                                    | $4.84     |
{ id="tab-total-cost" caption="Amortized itemized cost of the PCB" }

## Conclusion

The development experience, while still not great, I believe, is made much better with this PCB. What would have originally required the long and careful configuration and engineering of the Zynq through Vitis, just to obtain a clock signal, can now be done through Vivado, or through simple TCL scripts. This allows me to invest my time in the more interesting hardware design.

When I started this project, I believed I could make the KV260 dev board more user-friendly and justify my possibly rash expense of $249.00, for what, on paper, seemed like a great deal. I am glad I was partially right. That said, I don't think I would be recommending the board to a friend. This PCB alleviates some of the problems of the dev board, but I consider this work somewhat of a satirical hack, and not a serious solution.

[^not-beg1]: <https://www.reddit.com/r/FPGA/comments/1sp6hyb/comment/oh0hedy/>
[^not-beg2]: <https://www.reddit.com/r/embedded/comments/1qyx1q3/comment/o46wq7g/>
[^no-vivado1]: https://www.reddit.com/r/FPGA/comments/1sp6hyb/comment/ogzc2kh/
[^no-vivado2]: https://www.reddit.com/r/FPGA/comments/1oeskvp/comment/nl41vfs/
[^no-clock1]: <https://adaptivesupport.amd.com/s/question/0D54U00008sOBURSA4/-info-post-pl-and-ps-clock-source-in-kria-boards-kv260kr260kd240>
[^no-clock2]: <https://adaptivesupport.amd.com/s/question/0D54U00007rYseGSAS/kv260-pl-clock-pin-%E5%95%8F%E9%A1%8C>
[^no-clock3]: <https://adaptivesupport.amd.com/s/question/0D54U00008vA7uaSAC/kria-kv260-fpga-io-and-clock>
