# RISCV-IM-5-Stage-pipelined-Processor
This repository describes the information needed to build a five-stage RISC-V pipelined core, which has the upport of base interger RV32I instruction format and Multiplication instruction format using Nmigen. 

# Introduction 
RISC-V has been designed to support extensive customization and specialization. The base integer ISA can be extended with one or more optional instruction-set extensions, but the base integer instructions cannot be redefined. We have implemented the Multiplication extension of the same, taking into consideration stalling, forwarding and branching for optimized utilisation of the space and clock cycles.\

**RISC-V base instruction format is shown below** : 
![Image](https://github.com/Asmita-Zjigyasu/RISCV-IM-5-Stage-pipelined-Processor/blob/main/Images/riscv%20instruction%20description.gif "RISC-V base instruction formats")

![Image](https://github.com/Asmita-Zjigyasu/RISCV-IM-5-Stage-pipelined-Processor/blob/main/Images/riscv%20instructions%20set.gif "RISC-V base instruction set") 

# Design Description
1. This is a 5 stage pipelined processor based on RISCV architecture, which supports Integer arithematic operations and has been extended to accomodate Multiplication set of instructions as well.
2. The processor has been coded in Nmigen library of Python. This has been done because Python is very easy to understand. The Nmigen code can be converted to Verilog by 2 commands, which are stated ahead.
3. The 5 stages in the processor are - fetch, decode, execute, memory, writeback. All the stages are integrated using a Wrapper module.
4. This is a 32 bit processor with 1024 memory locations of both instruction memory and data memory.
5. It includes implementation of stall, forwarding and branching conditions while taking care of majority of the corner cases by performing extensive program testing. Several C programs have been converted to RISCV assembly using the RISCV GNU Toolchain and have been tested on the processor code with successful results. The C programs range from a simple ADD program (c=a+b) to a merge sort program for sorting 7 numbers.
6. Few of the many RV32I base instruction set descriptions can be found in the screenshot below:
//image of the riscv instruction set//

# Application
We plan to use the processor to build a full-fleged System on Chip design. This can be used to build a RISCduino if the correct and compatible peripherals are integrated with the processor. The RISCV procossor will be the core of the micro-controller.

# Block Diagram
![Image](https://github.com/Asmita-Zjigyasu/RISCV-IM-5-Stage-pipelined-Processor/blob/main/Images/block%20diagram.gif)

# Tool installation Details
## iVerilog : 
Icarus Verilog is an implementation of the Verilog hardware description language.

## GTKWave : 
GTKWave is a VCD waveform viewer based on the GTK library. This viewer support VCD and LXT formats for signal dumps. Waveform dumps are written by the Icarus Verilog runtime program vvp. The user uses $dumpfile and $dumpvars system tasks to enable waveform dumping, then the vvp runtime takes care of the rest. 

To download iVerilog and GTKWave in Ubuntu, use the following commands:
```
sudo apt-get update
sudo apt-get install iverilog gtkwave
```

## Nmigen : 
Despite being faster than schematics entry, hardware design with Verilog and VHDL remains tedious and inefficient for several reasons. Nmigen enables hardware designers to take advantage of the richness of the Python language—object oriented programming, function parameters, generators, operator overloading, libraries, etc.—to build well organized, reusable and elegant designs.\
To install Nmigen, follow the steps in the [Robert Baruch Nmigen Installation repository](https://github.com/RobertBaruch/nmigen-tutorial/blob/master/1_install.md).

## GNU Toolchain : 
The GNU Toolchain is a set of programming tools in Linux systems that programmers can use to make and compile their code to produce a program or library. So, here we use the RISCV GNU Toolchain to generate the RISCV Assembly for any C code.\
RISCV GNU Toolchain can be installed from [this repository](https://github.com/shivanishah269/risc-v-core#overview-of-gnu-compiler-toolchain).

# Functional Simulation
Run the following commands on terminal to run the verilog file with testbench to get a .vcd file output:

```
1. python3 Wrapper_class.py>wrapper.v  
2. iverilog wrapper.v test.v 
3. ./a.out 
4. gtkwave test.vcd
```

## Screenshot
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/gtkwave_simualtion.png)

# Functional characteristics
![Image](https://github.com/Asmita-Zjigyasu/RISCV-IM-5-Stage-pipelined-Processor/blob/main/Images/stages.gif)
## STAGES:
1. **Instruction Fetch**: IF module takes Program Counter as Input from Wrapper module and gives out the 32-bit Instruction which will be forwarded to the Decode stage for processing.
2. **Decode (ID)**: RV32IM has 7 types of instructions namely R, I, S, B, U, J, and M types differentiated by opcodes. According to each type, the source register, destination register and/or immediate field are extracted from the instruction. The extracted data is passed to Execute Stage.
3. **Execute**: Execute stage performs arithmetic and logical operations based on the opcode. Ra, Rb and/or sign-extended immediate field will be the parameters of the EX stage.
4. **Memory File**: Memory is accessed in this stage if required. This stage is accessed only by the load and the store instruction to extract or stare data into the memory. For load instruction, it loads an operand from the memory and for the store instruction, it would store an operand into the memory. 
5. **Register File**: The register file stores the data of all the registers of the processor. It is accessed in the ID stage and in the Write-back stage to extract data from the registers and to write to them. Data is written in the positive half of the cycle so that we read the updated data in the negative half of the clock cycle. We write data in the positive half of the cycle so that we read the updated data in the negative half of the clock cycle. 
## WRAPPER MODULE:
The wrapper module connects the 5 stages and makes the
processor work. It can be majorly divided into 4 units.
1. **Control Unit**: The control unit interlinks all 5 stages by connecting the outputs of one stage to the inputs of the other stage. It manages the communication of control signals to all other stages. The control signals determine how to process input data.
2. **Forwarding Unit**: Forwarding unit helps to reduce the overall CPI (clock cycles per instruction) of the processor. To accommodate this dependencies, the forwarding unit passes the data from the stage where it is available to the stage where it is required.
3. **Stalling Unit**: The functionality of this unit is to wait when there are dependencies between the instructions. It introduces NOP instruction between the instructions which have dependencies.
4. **Flushing Unit**: As this is a pipelined processor, by the time when the branching condition is checked in the EX stage, 2 instructions are already fetched. If the branch is taken, which is decided at the start of the MEM stage, the 3 fetched instructions need to be flushed out by the Flushing unit so that unnecessary changes are not made to the memory or register file.

The processor has been tested on several C programs, providing the correct output. The complete testing process has been expalined in [this document](https://docs.google.com/document/d/1JXD6lvziDR5GNDnnr6U30ztoxr11s2_hYlZX54NLtvs/edit?usp=sharing).

# Synthesis:
## Yosys installation for Ubuntu
1. Clone the [this](https://github.com/YosysHQ/yosys) GitHub repository.
2. Execute the following set of commands
```
1. make
2. sudo apt install make
3. sudo apt-get install build-essential clang bison flex \
    libreadline-dev gawk tcl-dev libffi-dev git \
    graphviz xdot pkg-config python3 libboost-system-dev \
    libboost-python-dev libboost-filesystem-dev zlib1g-dev
4 . make
5. sudo make install.

```
## Steps for generating the netlist
1. Clone [this](https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop) GitHub repository
2. Change the yosys_run.sh script with your design. 
3. Check the reference script in this repo.
4. Type yosys in the terminal which opens an interactive command shell.
5. Execute the following command which synthesizes your design and generates a gate-level netlist.
```
script yosys_run.sh
```
6. We can view the statistics of the netlist by typing "stat".
## Screenshots of the statistics of our design iiitb_riscv32im5
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/yosys_stats1.png)
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/yosys_stats2.png)

7. To test the post synthesis design execute the following commands
```
iverilog -DFUNCTIONAL -DUNIT_DELAY=#1 synth_riscv.v iiitb_riscv_tb.v primitives.v sky130_fd_sc_hd.v
./a.out
gtkwave test.vcd
```
## Post synthesis simulation
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/post_simulation_gtkwave.png)
* We can observe that the simulation results are same as before synthesis.


# Advanced Physical Design using OpenLane/Sky130 Workshop simulations and Results 

## Docker Installation
```
$ sudo apt-get remove docker docker-engine docker.io containerd runc (removes older version of docker if installed)
$ sudo apt-get update
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null  
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
$ apt-cache madison docker-ce (copy the version string you want to install)
$ sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io docker-compose-plugin (paste the version string copies in place of <VERSION_STRING>)
$ sudo docker run hello-world (If the docker is successfully installed u will get a success message here)
```

## OpenLane Installation
```
$ git clone https://github.com/The-OpenROAD-Project/OpenLane.git
$ cd OpenLane/
$ sudo make
$ sudo make test
```

## Magic Installation

For proper installation and workig of Magic, the following softwares have to be installed first:

### Installing csh
```
$ sudo apt-get install csh
```

### Installing x11/xorg
```
$ sudo apt-get install x11
$ sudo apt-get install xorg
$ sudo apt-get install xorg openbox
```

### Installing GCC
```
$ sudo apt-get install gcc
```

### Installing build-essential
```
$ sudo apt-get install build-essential
```

### Installing OpenGL
```
$ sudo apt-get install freeglut3-dev
```

### Installing tcl/tk
```
$ sudo apt-get install tcl-dev tk-dev
```
### Installing magic
After all the softwares are installed, run the following commands for installing magic:

```
$ git clone https://github.com/RTimothyEdwards/magic
$ cd magic
$ ./configure
$ sudo make
$ sudo make install
```

## Klayout Installation

```
$ sudo apt-get install klayout
```

## ngspice Installation

```
$ sudo apt-get install ngspice
```


# Creating a Custom Inverter Cell

Open Terminal in the folder you want to create the custom inverter cell and run the following commands:

```
$ git clone https://github.com/nickson-jose/vsdstdcelldesign.git
$ cd vsdstdcelldesign
$  cp ./libs/sky130A.tech sky130A.tech
$ magic -T sky130A.tech sky130_inv.mag &
```
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/custom%20inverter%20cell.png)


To extract Spice netlist, Type the following commands in tcl window.

```
% extract all
% ext2spice cthresh 0 rthresh 0
% ext2spice
```
"cthresh 0 rthresh 0" is used to extract parasitic capacitances from the cell.
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/cthresh_rthresh.png)

Copy the below code into sky130_inv.spice.
### Spice Netlist:
```
* SPICE3 file created from sky130_inv.ext - technology: sky130A

.option scale=0.01u
.include ./libs/pshort.lib
.include ./libs/nshort.lib


M1001 Y A VGND VGND nshort_model.0 ad=1435 pd=152 as=1365 ps=148 w=35 l=23
M1000 Y A VPWR VPWR pshort_model.0 ad=1443 pd=152 as=1517 ps=156 w=37 l=23
VDD VPWR 0 3.3V
VSS VGND 0 0V
Va A VGND PULSE(0V 3.3V 0 0.1ns 0.1ns 2ns 4ns)
C0 Y VPWR 0.08fF
C1 A Y 0.02fF
C2 A VPWR 0.08fF
C3 Y VGND 0.18fF
C4 VPWR VGND 0.74fF


.tran 1n 20n
.control
run
.endc
.end
```

Open the terminal in the directory where ngspice is stored and type the following command to open the ngspice console:
```
$ ngspice sky130_inv.spice 
```
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/ngspice.png)

Now plot the graphs for the designed inverter model using the following command:
```
plot y vs time a
```
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/inverter%20cell.png)


## Rise time and Fall time
Four timing parameters are used to characterize the inverter standard cell:
1. Rise time: Time taken for the output to rise from 20% of max value to 80% of max value
```
Rise time = (2.2 - 2.1) = 100ps
```

2. Fall time- Time taken for the output to fall from 80% of max value to 20% of max value
```
Fall time = (4.066 - 4.0) = 66ps
```

3. Cell rise delay = time(50% output rise) - time(50% input fall)
```
Cell rise delay = (2.150 - 2.076) = 74ps
```

4. Cell fall delay = time(50% output fall) - time(50% input rise)
```
Cell fall delay = (4.0 - 3.983) = 17ps
```

To save the file with a different name, use the folllowing command in tcl window
```
% save sky130_vsdinv.mag
```

Now open the sky130_vsdinv.mag using the magic command in terminal
```
$ magic -T sky130A.tech sky130_vsdinv.mag
```
In the tcl command type the following command to generate sky130_vsdinv.lef
```
$ lef write
```
A sky130_vsdinv.lef file will be created.

# Layout using OpenLane
The layout is generated using OpenLane. To run a custom design on OpenLane, navigate to the openlane folder and run the following commands:
```
$ cd designs
$ mkdir iiitb_riscv32im
$ cd iiitb_riscv32im
$ mkdir src
$ touch config.json
$ cd src
$ touch iiitb_riscv32im.v
```

Final src folder of the iiitb_riscv32im design should look like this:

![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/src_folder.png)

The iiitb_riscv32im.v file should contain the verilog RTL code you have used and got the post synthesis simulation for.

The contents of the config.json are as follows:
```
{
    "DESIGN_NAME": "top",
    "VERILOG_FILES": "dir::src/iiitb_riscv.v",
    "CLOCK_PORT": "clk",
    "CLOCK_NET": "clk",
    "GLB_RESIZER_TIMING_OPTIMIZATIONS": true,
    "CLOCK_PERIOD": 10,
    "PL_TARGET_DENSITY": 0.4,
    "FP_SIZING" : "relative",
    "pdk::sky130*": {
        "FP_CORE_UTIL": 30,
        "scl::sky130_fd_sc_hd": {
            "FP_CORE_UTIL": 20
        }
    },
    
    "LIB_SYNTH": "dir::src/sky130_fd_sc_hd__typical.lib",
    "LIB_FASTEST": "dir::src/sky130_fd_sc_hd__fast.lib",
    "LIB_SLOWEST": "dir::src/sky130_fd_sc_hd__slow.lib",
    "LIB_TYPICAL": "dir::src/sky130_fd_sc_hd__typical.lib",  
    "TEST_EXTERNAL_GLOB": "dir::../iiitb_riscv32im/src/*"
}
```

Copy "sky130_fd_sc_hd__fast.lib", "sky130_fd_sc_hd__slow.lib", "sky130_fd_sc_hd__typical.lib" and "sky130_vsdinv.lef" files to src folder in your design.

Navigate to the openlane folder in terminal and give the following command :
```
$ make mount (or use sudo as prefix)
```
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/openlane-3.png)


After entering the openlane container give the following command:
```
$ ./flow.tcl -interactive
```
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/openlane-2.png)


This command will take you into the tcl console. In the tcl console type the following commands:
```
% package require openlane 0.9
% prep -design iiitb_riscv32im
```
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/openlane-1.png)


The following commands are to merge external the lef files to the merged.nom.lef. In our case sky130_vsdiat is getting merged to the lef file

```
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
```

# Synthesis
```
% run_synthesis
```
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/openlane0.png)


## Synthesis Reports

Details of all the gates used:

![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/image.png)

Clock skew and Worst Slack:

![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/clock_skew.png)
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/worst%20slack.png)

# Floorplan
```
% run_floorplan
```
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/openlane1.png)

## Floorplan Reports

Die Area:

![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/floor_planning_1.png)

Core Area:

![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/floorplan%20area.png)

# Placement
```
% run_placement
```
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/openlane2.png)

The sky130_vsdinv should also reflect in your netlist after placement

![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/vsdinv_placement.png)
# Clock Tree Synthesis
```
% run_cts
```

# Routing
```
% run_routing
```
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/routing%20vsdinv.png)


we can open the def file and view the layout after the routing process by the following command, you can follow the path as per the image.<br>

```
$ magic -T /home/Desktop/ASIC/OpenLane/pdks/sky130A/libs.tech/magic/sky130A.tech read ../../tmp/merged.nom.lef def read top.def &
```
<br>

### NOTE
We can also run the whole flow at once instead of step by step process by giving the following command in openlane container<br>
```
$ ./flow.tcl -design iiitb_riscv32im
```
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/flow1.png)
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/flow2.png)
All the steps will be automated and all the files will be generated.<br>





we can open the mag file and view the layout after the whole process by the following command, you can follow the path as per the image.<br>

```
$ magic -T /home/Desktop/ASIC/OpenLane/pdks/sky130A/libs.tech/magic/sky130A.tech iiitb_riscv32im.mag &
```
<br>

![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/terminal_layout.png)
![Image](https://github.com/SakethGajawada/iiitb_riscv32im/blob/master/Images/layout1.png)

# Contibutors
* Mayank Kabra, Student, IIIT Bangalore
* Asmita Zjigyasu, Student, IIIT Bangalore
* Saketh Gajawada, Student, IIIT Bangalore
* Yathin Kumar, Student, IIIT Bangalore
* Oishi Seth, Student, IIIT Bangalore

# Acknowledgements
* Kunal Ghosh, Director, VSD Corp. Pvt. Ltd.

# Contact Information
* [Mayank Kabra](mailto:Mayank.Kabra@iiitb.ac.in?subject=[GitHub]%20ASIC%20Course)
* [Asmita Zjigyasu](mailto:Asmita.Zjigyasu@iiitb.ac.in?subject=[GitHub]%20ASIC%20Course)
* [Saketh Gajawada](mailto:Saketh.Gajawada@iiitb.ac.in?subject=[GitHub]%20ASIC%20Course)
* [Yathin Kumar](mailto:Yathin.Kumar@iiitb.ac.in?subject=[GitHub]%20ASIC%20Course)
* [Oishi Seth](mailto:Oishi.Seth@iiitb.ac.in?subject=[GitHub]%20ASIC%20Course)
* [Kunal Ghosh](kunalghosh@gmail.com)

# References
* [GNU Toolchain and RISCV overview](https://github.com/shivanishah269/risc-v-core)
* [RISCV Manual](https://riscv.org/wp-content/uploads/2017/05/riscv-spec-v2.2.pdf)
* [Robert Baruch Nmigen Tutorial](https://github.com/RobertBaruch/nmigen-tutorial)
* [Function calls](https://github.com/riscv-collab/riscv-gnu-toolchain/issues/1088)








