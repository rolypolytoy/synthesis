# Synthesis of the A2O OpenPOWER Chip
## An Ode to Yosys

Modern CPUs are incredibly useful tools, that can run several different kinds of software at various levels of abstraction- ranging from interacting with software via button-press GUIs to running programs in assembly on bare metal. 

## HDL
HDLs- or Hardware Description Languages, are programming languages that specifically describe the structure and behavior of electronic circuits. What does that mean?

They're software that describes hardware.

The two biggest ones are VHDL and Verilog- although SystemVerilog, a superset of Verilog, is replacing it in many modern applications. An example of Verilog code (taken from https://www.chipverify.com/tutorials/verilog) is below:


    module ctr (input  				up_down,
								clk,
								rstn,
            output reg [2:0] 	out);

	always @ (posedge clk)
		if (!rstn)
			out <= 0;
		else begin
			if (up_down)
				out <= out + 1;
			else
				out <= out - 1;
		end
    endmodule

This Verilog module describes a counter's behavior. More complex modules might describe the behaviors of entire hardware units- for example a major part of an ALU, or a component of the instruction decoder. 

## Conversion to Hardware
Verilog/VHDL can describe hardware behaviorally, but how do we turn, say, a repository of code in these languages into an actual CPU?
There are two main steps. One is to synthesize the code, the other is to actually manufacture the product of the synthesis.
Identifying how to actually manufacture hardware from a complete description of it is beyond the scope of this project, so we'll figure out how to do synthesis instead.

## What is Synthesis?
Synthesis is taking the HDL code and converting it into a manufacturable representation of the circuit you're trying to make.
For example- an HDL might in effect say 'there are four CPU cores' and references where you can find their design. When you check inside a CPU core, it describes how the parts connect. If you check, say, the FPU, it tells you how many parts there are, how they connect, and where to check to find them. It's not so much a blueprint as a blueprint for a blueprint. 

To do synthesis, we use a synthesis tool (aptly named). Some commercial ones include Synopsys Design Compiler, Genesys Cadence, or FPGA-supplier specific tools, but these are either extremely expensive, or very limited in what they can do. One open-source option is Yosys, but it only works on Verilog as intended.

Synthesis basically converts the he says-she says that's the reality of a repository of HDL code into a hardware-manufacturable representation, and you can specify what hardware process you want it to output when you do synthesis. 

For example, do you want it to output a gate-level netlist- essentially just a text-based representation of the logic gates that make it up and how they connect? Do you want it to use the SKY130 node, or any one of the several other process nodes it allows you to synthesize based on? Just pop over to synth.ys, modify one line of code (there's a comment next to it which tells you which), find the inbuilt library you want to use that Yosys already has, or find a .lib file for the one you want online and use that. The rest is handled. Which one you pick is determined entirely by what commercial process you intend to make your chip using, and once you've got a synthesized output that's passing all tests, after a few cursory steps all you've got to do is send it to a foundry to get fabricated, and you've got a functioning chip.

## Why a netlist? Why not actual production-ready GDSII files on an actually functional process?
Helps with debugging. A gate-level netlist is plaintext, so it's a lot easier to visually check if Yosys did synthesis correctly. Modify the synth.ys file if you want to synthesize it with a different library.

## A2O OpenPOWER Chip
The chip I've synthesized here is the A2O chip (https://github.com/OpenPOWERFoundation/a2o).  It's a 2000s-era CPU open-sourced by IBM, that uses the POWER7 ISA, and it's written entirely in Verilog, which means it's possible for us to synthesize it with Yosys. The copy of it in this repo is modified to remove a few errors in its header files and its synthesis code is heavily modified. Because the license it has is Apache 2.0, the license of this project is also necessarily that. For more info on the Apache 2.0 license, refer to https://www.apache.org/licenses/LICENSE-2.0.

I've also synthesized GDSII files for it using OpenROAD (I found the sky130 liberty files at https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/blob/master/flow/platforms/sky130hd/lib/sky130_fd_sc_hd__tt_025C_1v80.lib after a lot of perusing, and have included it in this project for your ease), so as to make this closer in style and quality to production-level tapeout projects in backend VLSI.

## Dependencies
I'd recommend a Linux environment, or at least WSL if you're using Windows.
To install Yosys:
	sudo apt-get install yosys
Then:
	git clone https://github.com/rolypolytoy/synthesis

If you want to synthesize it yourself- refer to synth_netlist.ys, navigate to it in your command line, and then:

	yosys synth_netlist.ys

Or, use yosys synth_sky130.ys if you want to synthesize a netlist compatible with the SKY130 node (an open-source 130-180 nm node that's closer to the kind of work required by actual tapeout). Make sure to change the locations referenced in the synthesis files, depending on your OS and what location you have your files in.

It should output, in the same repository, a file called netlist.v, or sky130.v

Or, if you don't want to have to even go through synthesis, refer to netlist.v or sky130.v.

## Netlist to GDSII

To take our gate-level netlist sky130.v and have production-ready GDSII files, we have further steps to do. The next steps of this project requires going from a netlist to a floorplan, doing placement, CTS, routing, and then generating GDSII files from this. Future projects in this vein will implement these for the sky130 nm node. Keep in mind, due to the open-source nature of OpenROAD, several other nodes can be made with this pipeline including Globalfoundries 180nm, Nangate45, ASAP7, and more. With commercial tools such as Synopsys and Cadence, virtually all modern semiconductor nodes can be made with processes even more streamlined than OpenROAD uses, which is less sophisticated than commercial solutions due to its open-source nature.

For now- you now have a gate-level netlist of an entire CPU on your computer. Download netlist.v or sky130.v for a gate-level netlist of the A2O core, or synthesize it yourself for your own enjoyment.

## Bug Documentation

Whenever I run the following [synthesis script](https://github.com/rolypolytoy/synthesis/blob/main/synth_sky130.ys) I run into the error message: /mnt/c/Users/rishi/Desktop/a2o/dev/verilog/work/c_wrapper.v:252: ERROR: Unimplemented compiler directive or undefined macro `NCLK_WIDTH. NCLK_WIDTH is the issue. If we examine [c_wrapper.v](https://github.com/OpenPOWERFoundation/a2o/blob/master/dev/verilog/work/c_wrapper.v) (the culprit) we find that it includes [tri_a2o.vh](https://github.com/OpenPOWERFoundation/a2o/blob/d8d9fb6a9967ee51398db03f10dea8f6fbd662be/dev/verilog/trilib/tri_a2o.vh#L4). If you look at a commit message you can find [this](https://github.com/OpenPOWERFoundation/a2o/commit/3a65b3deaf595f6add1d8641d5c24b44b3f7d8d0) which includes commenting out an NCLK_WIDTH-related piece of logic at [tri.vh](https://github.com/OpenPOWERFoundation/a2o/blob/master/dev/verilog/trilib/tri.vh) (changes [here](https://github.com/OpenPOWERFoundation/a2o/commit/3a65b3deaf595f6add1d8641d5c24b44b3f7d8d0#diff-4930dd4d9b015fa840b43bf64f2cf6d7d0255ca5326886b049fbbcdc5703cd55).

Simply uncommenting the \\ behind the critical piece of logic enables full synthesis with the aforementioned script, or just using my [fork](https://github.com/rolypolytoy/a2o) of the repo fixes this.
