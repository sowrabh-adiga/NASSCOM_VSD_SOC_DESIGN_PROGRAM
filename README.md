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
 
## LAB 

* Enter the dir `~/Desktop/work/tools/openlane_working_dir/openlane` all openlane runs must happen from this directory.
* the design folder `~/Desktop/work/tools/openlane_working_dir/openlane/design` holds the data for RTL2GDSII implementation, we focus on `picorv32a`. the `config.tcl` inside the designs have highest priprity and the cmds mentioned in them override the defualt commands in the tools
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
  ![Screenshot from 2025-03-27 14-08-29](https://github.com/user-attachments/assets/86af7bab-5f48-4078-8016-4a62847cbd75)

* after this `runs` dir willbe created in the `designs/picorv32a` folder
  ![Screenshot from 2025-03-27 14-10-38](https://github.com/user-attachments/assets/552dadfd-3a1d-4420-988c-5764904a0760)


