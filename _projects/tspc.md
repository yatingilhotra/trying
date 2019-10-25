---
layout: project
title: Controlling TSPC Parameters
nav: projects
importance: 80
description: controlling the parameters of a tspc flip-flop
img: /assets/img/tspc.svg
---


TSPC(True Single-Phase Clock) Logic, as the name suggests is a family of logic circuits which uses just one clock phase to implement an operation, and doesn't require multiple clock phases, or sets of non-overlapping clocks. This inherent property of TSPC logic eases the distribution of multiple phases of clocks, which is otherwise a challenging task in high speed systems. Another reason it is widely popular is because it is less bulky, and hence suitable for high speed, low power/area applications.      

The maximum frequency of the clock on which a system can run is limited by the latency of individual elements. For example, the maximum frequency of clock, \\(f_{clk}\\) for a two-flop system, as shown in Figure 8, is decided by the time \\(t_1\\), the clock to output delay of the flop, \\(t_2\\), latency of the logic, and \\(t_{setup}\\), the setup time of the flop. For reliable operation, the two flop system must follow this relation:

$$ {\bf t}_1 + {\bf t}_2 + {\bf t}_{setup} < {\bf T}$$

Where, \\(T\\) is the time period of the clock, related to the frequency by the relation \\(f_{clk} = 1/T\\).

<div class="container-fluid text-center mt-4 p-0">
  <img class="img-responsive col-12 col-sm-10 col-md-6 ml-auto mr-auto" src="{{ '/assets/img/clock_system.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center mt-2" style="color: #78909c;"><u>Figure 8:</u> A two flop clock system</h6>
</div>

Clearly, having lower setup times, and lower latency for the circuits is the key to higher speed operation. It is instrumental to understand how to get desired values for these parameters with minimal cost. Here, the setup time, hold time, and the clock-out delay is explained for a conventional TSPC Flop, as shown in Figure 9.

<div class="container-fluid text-center mt-4 p-0">
  <img class="img-responsive col-12 col-sm-10 col-md-6 ml-auto mr-auto" src="{{ '/assets/img/tspc.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center mt-2" style="color: #78909c;"><u>Figure 9:</u> TSPC D Flip-Flop</h6>
</div>

Figure 9 shows a D-FF with an inverted output \\(Q_{bar}\\), which means, sampling a 1 gives a 0, and vice versa. \\(t_{setup}\\), \\(t_{hold}\\), and \\(t_{clk-q}\\) can be mapped to the delays of specific elements in Figure 9, but before that, it is important to define these quantities. So, \\(t_{setup}\\) is defined as the minimum amount of time the in data must be stable, before the clock's active edge comes, for reliable latching. \\(t_{hold}\\) is defined as the minimum amount of time for which data must remain stable after the clock's active edge, and \\(t_{clk-q}\\) is the amount of time the circuit takes to give the output after the sampling clock edge arrives. Any input data transition between \\(t_{setup}\\) +\\(t_{hold}\\) window may result in incorrect data at the output, which is also known as the metastability window.

<h4 class="title mt-4 p-0 text-left">Sampling a 1</h4>

Let's see the mechanism by which the circuit shown in Figure 9 samples a 1. Before the positive(active) edge of the clock comes, N1 must pull down the node X to the ground, since in the time region \\(T_1\\), \\(D = 1\\), N1 would be ON and P1 would be OFF. Other than this, Y must be pre-charged to 1 by P3 since the \\(CLK = 0\\). X and Y nodes should be stable before active clock edge comes for proper operation. Since the circuit is obligated to perform these tasks before the arrival of the clock, we can say:

$$ t_{setup} \propto \max{[\Delta_{D-X(N1)}, \Delta_{CLK-Y(P3)}]}, $$

where \\(\Delta_{D-X(N1)}\\) is the time taken by N1 to discharge the node X, and \\(\Delta_{CLK-Y(P3)}\\) is the time taken by P3 to charge node Y, upon the onset of \\(D\\) and \\(CLK\\) respectively. If the \\(\Delta_{CLK-Y(P3)}\\) is small enough, then  

$$ t_{setup} \propto \Delta_{D-X(N1)}$$

since P3 starts to charge the node Y before the N1 starts to discharge the node X, because CLK becomes 0 before the D becomes 1, as shown in the Figure. Now, when the positive clock edge finally arrives after the node X and Y have settled, the output is supposed to become 0. \\(\Delta_{CLK-Q(D=1)}\\) is the time it takes for the stacked N4 and N5 to discharge the output node after the clock'e positive edge comes.

$$ \Delta_{CLK-Q(D=1)} = \Delta_{Y-Q(N4-N5)} $$

So, making N4 and N5 faster is an effective knob for decreasing the clock-to-output delay for sampling a 1.

<h4 class="title mt-4 p-0 text-left">Sampling a 0</h4>
When D = 0, X would be Pulled up to 1 by the P1-P2 PMOS stack. Y would still have to be charged to 1 by P3, because CLOCK = 0. Since D transition happens after CLK goes to 0, if \\(\Delta_{CLK-Y(P3)}\\) is small enough, then it can be easily said that(using the similar argument made above for the 'sampling a 1' case):

$$ t_{setup} \propto \Delta_{D-X(P1-P2 Stack)}$$

And it can be easily analyzed that the clock-output delay in this case would be:

$$ \Delta_{CLK-Q(D=0)} = \Delta_{X-Y(N2-N3)} + \Delta_{Y-Q(P4)} $$
