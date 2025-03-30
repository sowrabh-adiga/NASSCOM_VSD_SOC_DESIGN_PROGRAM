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
  

## Sky130 Day 2 - Good floorplan vs bad floorplan and introduction to library cells

### SKY130_D2_SK1 - Chip Floor planning considerations

* Floorplanning: defining the dimesions of a chip
* assume unit width and height of std cells and FF's then taking all of them together, gives us the abs min area required to place the cells inside a core area
* below are the most important concepts for floorplan definition: 
```math
Utilization = \frac{Area\ utilized\ by\ netlist}{Total\ core\ area}
```
and 

```math
Aspect\ Ratio= \frac{Heigth of core}{Width of core}
```
* with abs min area known, we need to account for area occupied by routing metals.
## (March/28/2025)
* **Preplaced cells concept** : top level logic is broken into mutiple cuts and are implemented separately
* The circuits implemented in such a way can be black boxed and the design can be used multiple times, like an IP block
* Memory, complex clock gating cells, mux, comparators, ALU’s can be designed in such a way
* These cells are placed and fixed before the std cell placement and the tool doent alter the placement of these pre-placed cells
* we need to surround the preplaced cells with decoupling  cells
  
* **decoupling  cells** help maintain stable supply voltage across the logic they are placed near , by provding capacitive charging effect in the event of supply voltage drop due to reistance on the pwr supply path
* **pwr planning**: Having single pwr supply point results in voltage drop in larger circuits, this results in signal integrity issues. To solve this pwr meshes are spread throughout the design to have a very small path from pwr supply to logic devices.
* **Pin Placement** requires knoledege of the design, how pins interact with cels inside the design.
## LAB : floorplan and placement
* `:~/Desktop/work/tools/openlane_working_dir/openlane/configuration` contains the tool default configurations and `README.md` filde inside that folder lists all the diffrent config variables avaible for the designer
* priority : `:~/Desktop/work/tools/openlane_working_dir/openlane/configuration` < `design/runs/config.tcl` < `design/PDK.tcl`
```bash
  run_floorplan
  # run floorplan
```
floorplan run in terminal
 ![runs folder content](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-28%2014-37-46.png)
The run outputs structure in runs folder

![runs folder content](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-28%2014-36-49.png)

floorplan def
![ ](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-28%2014-45-16.png)

Floorplan def gives us the actual arae of design as follows:
```math
1000\ Unit\ Distance = 1\ Micron
```
```math
Width\ in\ unit\ distance = 660685 - 0 = 660685
```
```math
Height\ in\ unit\ distance = 671405 - 0 = 671405
```
```math
Area\ of\ die\ in\ microns = \frac{660.685 * 671.405}{1000 * 1000} = 443587.21\ Microns^2
```

* we can view the floorplan generated with the help of `magic` tool
```bash
# in a seperate terminal enter the directory with def filr (results folder in recent run stage : floorplan)
magic -T ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &

```
magic Gui
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-28%2015-13-37.png)
* in `magic` gui to center the design for better viewing when in  the gui press "s" and "v", "s" is to select an object (entire design in this case)
* to zoom into a location: left click mouse for lower (x1,y1) and right mouse click for setting (x2,y2)
* keep the cursor on the object whose info is needed, then press "s" to select and then in tkconsole write cmd "what"
Pin info in the GUI
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-28%2015-21-49.png)

diffrent elements in the floorplan
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot-9.png)

### SKY130_D2_SK2 - Library Binding and Placement

* **Library**: contains diffrent flavours of cells, timing, functionality informations
* Placing the netlist elements in core area from floorplan stage is **placement**
* When the placement is done its wrt the pin placement, estimated resistance, capacitances of the interconeects . signal Transition analysis is done to see if the placed cells are recievin the signals on time, if not buffers are added to improve the transition.

### LAB 
```bash
run_placement
```
output on terminal;
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-28%2016-27-11.png)

files generated in `results/placement` dir. 
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-28%2016-33-04.png)

see the def generated in `magic`

```bash
# in a seperate terminal enter the directory with def file (results folder in recent run stage : placement)
magic -T ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &

```

viewing the def generated in `magic`
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-28%2016-41-08.png)

![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-28%2016-39-58.png)


## Sky130 Day 3 - Design library cell using Magic Layout and ngspice characterization
### Lab (MArch/30/2025)

* Clone custom inverter standard cell design from github repository: [nickson-jose vsdstdcelldesign](https://github.com/nickson-jose/vsdstdcelldesign.git).
* Load the custom inverter layout in magic and explore.
```bash
# Enter the Openlane work dir 
# Clone the repo with custom inverter design
git clone https://github.com/nickson-jose/vsdstdcelldesign.git

# Enter repo directory
cd vsdstdcelldesign

# Copy magic tech file to the repo directory for easy access
cp ../../pdks/sky130A/libs.tech/magic/sky130A.tech .

# Command to open custom inverter layout in magic
magic -T sky130A.tech sky130_inv.mag &

# Do "s" to select the element under the cursor. multiple "s" selects all elements connected to the element under the cursor
# after selection in tkconsole  of magic enter the command "what" to get info on elements selected
```
folder created by git clone
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-30%2016-25-42.png)
magic gui for std cell
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-30%2016-26-21.png)
PMOS identification
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-30%2017-32-20.png)
NMOS identification
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-30%2017-32-42.png)
Drain to drain connection of nmos and pmos
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-30%2017-34-28.png)
Pwr to Pmos connection
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-30%2017-34-46.png)
Gnd to Nmos connection
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-30%2017-35-01.png)

* Next extract the inv cell layout
```
#inside the magic tkconsole
extract all
# creates cell.ext file
ext2spice
# creates spice file
```
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-30%2018-20-39.png)
