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

For example, do you want it to output a gate-level netlist- essentially just a text-based representation of the logic gates that make it up and how they connect? Do you want it to use the SKY130 node, or any one of the several other process nodes it allows you to synthesize based on? Which one you pick is determined entirely by what commercial process you intend to make your chip using, and once you've got a synthesized output that's passing all tests, after a few cursory steps all you've got to do is send it to a foundry to get fabricated, and you've got a functioning chip.

## A2O OpenPOWER Chip
The chip I've synthesized here is the A2O chip (https://github.com/OpenPOWERFoundation/a2o).  It's a 2000s-era CPU open-sourced by IBM, that uses the POWER7 ISA, and it's written entirely in Verilog, which means it's possible for us to synthesize it with Yosys. The copy of it in this repo is modified to remove a few errors in its header files, and because the license it has is Apache 2.0, the license of this project is also necessarily that. For more info on the Apache 2.0 license, refer to https://www.apache.org/licenses/LICENSE-2.0.

## Dependencies
I'd recommend a Linux environment, or at least WSL if you're using Windows.
To install Yosys:
	sudo apt-get install yosys
Then:
	git clone https://github.com/rolypolytoy/synthesis

Unzip the a2o-master file, and remove the duplicate of it (it shouldn't be a2o-master inside a2o-master, remove the recursion if you want the synthesis to function correctly.) If you want to synthesize it yourself- refer to synth_netlist.ys, navigate to it in your command line, and then:

	yosys synth_netlist.ys

It should output, in the same repository, a file called netlist.v.


Or, if you don't want to have to even go through synthesis, refer to netlist.v.

You now have a gate-level netlist of an entire CPU on your computer. 
