# NASSCOM_VSD_SOC_DESIGN_PROGRAM
Opensource RTL to GDS2 implementation on OpenLane

## Sky130 Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK (March/26/2025 - March/27/2025)

### SKY130_D1_SK1 - How to talk to computers

* Any embedded system will have chips. The chip is present in the **Package** to make it easy to mount on a embedded board and handling. Connections to the chip is done by various packaging methods, common among which is **Wire Bound** method
* Zooming into the chip itself, there is **I/o clearing** for placing I/O pads and **core area**. Both of these are sitting inside the **Die area**. The **wire Bound** connections attch to the i/o pad for the chip to communicate with outside.
* **Foundary** manufactures  the chips and can also provide **IP's (Intellectual Properties)** in the form of soft or hard macros to be used inside the design
* ISA is **Instruction Set Architecture** is asm code acting as last human readable code before its converted to machine code
* typical flow {C code > complied to ASM > assembler to machine code}
* The RTL code is used to model the hardware that will run the ISA.

### SKY130_D1_SK2 - SoC design and OpenLANE
* we require the following enablers to be readily available as open-source versions for open-source ASIC design implementation. They are as below
    1. RTL Designs
    2. EDA Tools
    3. PDK Data
* ASIC Implementation takes a lot of steps and openlane achieves this by pulling together the tools that enable each step
* Typical flow is as follows:
   1. Synthesis: Maps RTL to physical library from a PDK and gives us netlist defined with devices for that design. In other words during this stage the RTL (Register Transfer Level) code is converted to ( Verilog format) optimized Gate level netlist.
   2. Floorplan & Powerplan: Die size, pin placement and power scheme are set during this stage
   3. Place: Std cells are placed inside the core region
   4. CTS: All seq elements receive clock connection from clock tree generation
   5. Routing: Signal connections are done
   6. Signoff: The design is checked for DRC, LVS, timing issues. GDS is generated
 
## LAB : SKY130_D1_SK3 - Get familiar to open-source EDA tools

* Enter the dir `~/Desktop/work/tools/openlane_working_dir/openlane` all openlane runs must happen from this directory.
* the design folder `~/Desktop/work/tools/openlane_working_dir/openlane/design` holds the data for RTL2GDSII implementation, we focus on `picorv32a`. the `config.tcl` inside the designs have the highest priority and the cmds mentioned in them override the default commands in the tools
* To open the openlane instance:
  ```bash
  docker
  # calls the docker instance for the openlane
  ./flows.tcl -interactive
  # opens openlane terminal
  package require openlane 0.9
  # calls openlane 0.9
  prep -design picorv32a
  # prepares Picorv32a design for implemtation
  ```
  ![Screenshot from 2025-03-27 14-08-29](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-27%2014-08-29.png)

* after this `runs` dir willbe created in the `designs/picorv32a` folder
  ![Screenshot from 2025-03-27 14-10-38](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-27%2014-10-38.png)
* inside the run directory the folders will be created as the runs are done
  ![run dir](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-27%2014-51-16.png)
* there is one `config.tcl` which is diffrent from the other config file discussed before, this one shows which default parameters are being taken bt=y the runs
* Running synthesis:
  ```bash
  run_synthesis
  # runs synthesis and abc's
  ```
* after the run we can check the run dir for the results, logs and reports. `results` folder must have the synthesized netlsit. `reports` folder will have all the reports for synthesis step and `logs` will store all the run logs for synthesis. in the `reports` chronologically latest report is most accurate 
  ![runs folder content](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-27%2015-15-59.png)

* At this step we need to do DFF ratio calculation as follows:
```math
Flip Flop\ Ratio = \frac{Number\ of\ D\ Flip\ Flops}{Total\ Number\ of\ Cells}
```
```math
Percentage\ of\ DFF's = Flip Flop\ Ratio * 100
```
* checking for latest file we have `reports/synthesis/1-yosys_4.stat.rpt` from dir image above. Therefore Flip Flop ratio is:
```math
Percentage\ of\ DFF's = \frac{1613}{14876} * 100 = 10.84 %
```
* we can also analyse the sta reports along with the yosys usage summary
  


