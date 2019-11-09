---
layout: project
title: Controlling TSPC Parameters
nav: projects
importance: 90
description: controlling the setup, hold and clock-to-q of a tspc flip-flop
img: /assets/img/tspc.svg
---


TSPC(True Single-Phase Clock) Logic, as the name suggests is a family of logic circuits which uses just one clock phase to implement an operation, and doesn't require multiple clock phases, or sets of non-overlapping clocks. This inherent property of TSPC logic eases the distribution of multiple phases of clocks, which is otherwise a challenging task in high speed systems. Another reason it is widely popular is because it is less bulky, and hence suitable for high speed, low power/area applications.      

The maximum frequency of the clock on which a system can run is limited by the latency of individual elements within that system. For example, the maximum frequency of clock, \\(f_{clk}\\) for a two-flop system, as shown in Figure 1, is decided by the time \\(t_1\\), the clock to output delay of the flop, \\(t_2\\), latency of the logic, and \\(t_{setup}\\), the setup time of the flop. For reliable operation, the two flop system must follow this relation:

$$ {\bf t}_1 + {\bf t}_2 + {\bf t}_{setup} < {\bf T}$$

Where, \\(T\\) is the time period of the clock related to the frequency by the relation \\(f_{clk} = 1/T\\).

<div class="container-fluid text-center mt-4 p-0">
  <img class="img-responsive col-12 col-sm-10 col-md-6 ml-auto mr-auto" src="{{ '/assets/img/clock_system.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center mt-2" style="color: #78909c;"><u>Figure 1:</u> A two flop clock system</h6>
</div>

{% comment %}
The other condition for reliable operation is that data must remain stable at the input of a certain flop, after the clock's active edge samples the data, which is known as the hold time. If both the flops in the system shown are identical then this condition appears:

$$ {\bf t}_1 + {\bf t}_2 < {\bf t_{hold}}$$
{% endcomment %}

Clearly, having lower setup time and delay for the TSPC flop is the key to higher speed operation. It can be instrumental to understand how to get desired values for these parameters with minimal power and area cost. This article is about understanding these trade-offs and minimizing the cost for a conventional TSPC flip-flop as shown in Figure 2.

<div class="container-fluid text-center mt-4 p-0">
  <img class="img-responsive col-12 col-sm-10 col-md-6 ml-auto mr-auto" src="{{ '/assets/img/tspc.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center mt-2" style="color: #78909c;"><u>Figure 2:</u> TSPC D Flip-Flop</h6>
</div>

Figure 2 shows a D-FF with an inverted output \\(Q_{bar}\\), which means, sampling a 1 gives a 0, and vice versa. \\(t_{setup}\\), \\(t_{hold}\\), and \\(t_{clk-q}\\), (which is \\(t_1\\) in Figure 1), can be mapped to the delays and sizes of specific elements(MOSFETs) in Figure 2. \\(t_{setup}\\) is defined as the minimum amount of time for which the input data must stay stable before the clock's active edge comes for reliable latching. \\(t_{hold}\\) is defined as the minimum amount of time for which data must remain stable after the clock's active edge, and \\(t_{clk-q}\\) is the amount of time the circuit takes to give the output after the active clock edge arrives. Any input data transition between \\(t_{setup}\\) + \\(t_{hold}\\) window may result in incorrect data at the output. This time region is also known as the metastability window. Again, mimizing this metastability window is desirable. Understanding the sampling mechanism by this flop, and the origin of \\(t_{setup}\\), \\(t_{hold}\\), and \\(t_{clk-q}\\) within this flop gives insights which help in reducing these times. These 3 parameters are different when sampling a one, and when sampling a zero.

<h4 class="title mt-4 p-0 text-left">Sampling a 1</h4>

<div class="container-fluid text-center mt-4 p-0">
  <img class="img-responsive col-12 col-sm-10 col-md-6 ml-auto mr-auto" src="{{ '/assets/img/sampling1.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center mt-2" style="color: #78909c;"><u>Figure 3:</u>Timing Diagram describing setup, hold and delay</h6>
</div>

The flip-flop in figure 2 is a positive edge triggered flop. Before the clock's positive edge comes, N1 must pull down the node X to the ground<b>(Action 1)</b>, if the flop is sampling a one (\\(D=1\\)). Other than this, Y must be pre-charged to 1 by P3<b>(Action 2)</b> since \\(CLK = 0\\). X and Y nodes should be stable before postive clock edge comes for the circuit to operate properly. Since the circuit is obligated to perform Action 1 and 2 before the arrival of the clock, this dictates the setup time for sampling a 1. We can say:

$$ t_{setup(D=1)} \propto \max{[\Delta_{D-X(N1)}, \Delta_{CLK-Y(P3)}]}, $$

where \\(\Delta_{D-X(N1)}\\) is the time taken by N1 to discharge the node X, and \\(\Delta_{CLK-Y(P3)}\\) is the time taken by P3 to charge node Y, upon the onset of \\(D=1\\) and \\(CLK=0\\) respectively. If the \\(\Delta_{CLK-Y(P3)}\\) is small enough, then  

$$ t_{setup(D=1)} \propto \Delta_{D-X(N1)}$$

<!-- since P3 starts to charge the node Y before the N1 starts to discharge the node X, because CLK becomes 0 before the D becomes 1, as shown in the Figure 3.-->

 Based on which delay is larger, making the Action 1 or Action 2 faster could be the knobs to effectively reduce the setup time. The time it takes for Action 1 or 2 could be derived using MOS current equations, but it might be easier to take the help of a simulator to determine them.

 When the positive clock edge finally arrives after the node X and Y have settled, the output is supposed to become 0. Since the node Y is pre-charged to 1, \\(\Delta_{CLK-Q(D=1)}\\) is the time it takes for the stacked N4 and N5 to discharge the output node when clock's positive edge arrives.

$$ \Delta_{CLK-Q(D=1)} = \Delta_{Y-Q(N4-N5)} $$

Making N4 and N5 faster is an effective knob for decreasing the clock-to-output delay for sampling a 1.

If the node Y changes its state, and becomes 0 before the N4-N5 stack discharges the output \\(Q_{bar}=0\\), then the circuit operation fails. So, the node Y should stay in the pre-charge state till N4-N5 stack passes the zero to the output. The factors which could drive the change of state of Y after the positive clock edge are:

- (a) D becoming 0 from 1, and/or the leakage from P1 and P2 which could make X=1, which will make Y=0.
- (b) The leakage from N3 and N2 which could make Y=0.

These inherent leakages hinder with the TSPC flop's operation at lower frequencies. Even if the leakages aren't enough to flip the node Y, they could still cause an increase in the clock-to-q delay. Since the hold time is usually characterized by measuring the clock-to-q delay for various relative clock-data edge positions, this would cause an increase in the hold time of the flop. Placing weak keeper circuits generally helps in reducing the effect of leakages by holding the internal node states.   

$$ t_{hold(D=1)} \propto leakages (a) and (b)$$

<h4 class="title mt-4 p-0 text-left">Sampling a 0</h4>
Using the same arguments as used above, and analysing the setup and clock-to-q for sampling a 0, the following relations can be established:


$$ t_{setup(D=0)} \propto \max{[\Delta_{D-X(P1-P2)}, \Delta_{CLK-Y(P3)}]}, $$

if \\(\Delta_{CLK-Y(P3)}\\) is sufficiently small, then

$$ t_{setup(D=0)} \propto \Delta_{D-X(P1-P2)}$$

where \\(\Delta_{CLK-Y(P3)}\\) is the time taken by the P3 transistor to charge Y when clock's negative edge comes (it becomes 0), \\(\Delta_{D-X(P1-P2)}\\) is the amount of time P1-P2 stack takes to charge \\(X\\)to 1 when \\(D=0\\). The clock-to-q for sampling a 0 directly relates to the discharging of node Y by N2-N3 stack, and charging of \\(Q_{bar}\\) by P4 when clock's positive edge comes.

$$ \Delta_{CLK-Q(D=0)} \propto \Delta_{X-Y(N2-N3)} + \Delta_{Y-Q(P4)} $$

In this case, the hold violation arises when Y doesn't achieve fully discharged state. This happens when before the flop passes a 1 to the output, input \\(D\\) changes its state to 1, which in turn tries to make \\(X=0\\), and \\(Y\\) is not strongly discharged. This increases the time it takes for P4 to make \\(Q_{bar} = 1\\), and in extreme cases, it may entirely contradict with the flop's correct operation. This is the reason as to why D must remain stable for a certain time after the positive edge of the clock.

<b>By understanding these mechanisms which dictate setup, hold, and delay parameters, minimal and meticulous changes in the sizes of specific MOSFETs could be made to achieve desirable flop characteristics. This keeps in check the area and power of a single TSPC instance. Since these flops are instantiated probably hundreds of times in high-speed transceivers, their efficient design could result in significant area and power savings for the whole chip.</b>
