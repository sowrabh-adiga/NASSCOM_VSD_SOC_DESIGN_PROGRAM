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

### SKY130_D3_SK3 - Sky130 Tech File Labs (March/31/2025)
#### ngspice sim for cell characterization

* After the cell layout is created we need to do characterization (timing , noise)
* timimng characterization is sone after extracting the device parasitics and R,C and running simulatioms
* here for inv cell we do transient analysis to find rise , fall transition, cell delay  

### LAB
* To create spice deck from spice file obtained from magic GUI , we need to add some extra commands tp spice
      1. add lib paths were the pmos,nmos are modeled
      2. add the proper grid dimension
      3. check if the correct name is used for pmos, nmos moidels
      4. Add signal, pwr, gnd connections to the circuit connections
      5. define analaysis type and specify the parameters for simulation
  ![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot-17.png) 
Commands for ngspice simulation

```bash
# Command to directly load spice file for simulation to ngspice
ngspice sky130_inv.spice

# ngsice terminal will open, inside that make sur ethere is no errors, warnings mentioned. if errors are present, solve them first, before proceeding any further
plot y vs time a
# plotting output vs time and input
```
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-31%2016-57-45.png)



Rise transition time calculation

```math
Rise\ transition\ time = Time\ taken\ for\ output\ to\ rise\ to\ 80\% - Time\ taken\ for\ output\ to\ rise\ to\ 20\%
```
```math
20\%\ of\ output = 0.66\ V
```
```math
80\%\ of\ output = 2.64\ V
```


```math
Rise\ transition\ time = 2.2468 - 2.1824 = 0.06396\ ns = 64.4\ ps
```
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-31%2018-07-10.png)

Fall transition time calculation

```math
Fall\ transition\ time = Time\ taken\ for\ output\ to\ fall\ to\ 20\% - Time\ taken\ for\ output\ to\ fall\ to\ 80\%
```
```math
20\%\ of\ output = 660\ mV
```
```math
80\%\ of\ output = 2.64\ V
```
```math
Fall\ transition\ time = 4.0955 - 4.0530 = 42.5\ ps
```

![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-31%2018-11-20.png)

Rise Cell Delay Calculation

```math
Rise\ Cell\ Delay = Time\ taken\ for\ output\ to\ rise\ to\ 50\% - Time\ taken\ for\ input\ to\ fall\ to\ 50\%
```
```math
50\%\ of\ 3.3\ V = 1.65\ V
```



```math
Rise\ Cell\ Delay = 2.21 - 2.15 = 0.06\ ns = 60\ ps

```
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-31%2018-18-42.png)

Fall Cell Delay Calculation


```math
Fall\ Cell\ Delay = Time\ taken\ for\ output\ to\ fall\ to\ 50\% - Time\ taken\ for\ input\ to\ rise\ to\ 50\%
```
```math
50\%\ of\ 3.3\ V = 1.65\ V
```



```math
Fall\ Cell\ Delay = 4.0780 - 4.0501 = 0.0279\ ns = 27.9\ ps
```
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-03-31%2018-23-20.png)


### APRIL/1/2025

download the drc_test folder in home directory
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-01%2013-21-44.png)

```bash
cd drc_tests

vi .magicrc
# to view the magic rc file

magic -d XR met3.mag&
```
TO draw a layer in magic
* create a bbox by left right click of mouse
* select the layer from the layer window by clicking it with mid mouse button
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-01%2018-52-22.png)

As example for drc rule fxing open poly.mag in magic and zoomin on poly.9 error, serch for the error defination in online pdk resource at [skywater pdk readthedocs](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#poly)
open the tech file `sky130A.tech` in the same folder and edit the missing rule: (the redbox in images below are the edited rules)
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot-25.png)
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot-27.png)
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-01%2020-02-47.png)


## Sky130 Day 4 - Pre-layout timing analysis and importance of good clock tree : SKY130_D4_SK1 - Timing modelling using delay tables

### April/2/2025
*Tmimng table is used to model delay of a cel wrt to input transition and output load
* To insert the custom design (`sky130_vsdinv` in this course )into openlane flow, we need the characterised lib and lef
* from `magic` GUI
```
# rename the cell design
save <new name>.mag

save sky130_vsdinv.mag

# open the design in new gui and write lef
write lef <optional name, default is the design name in the gui>
```
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-02%2013-09-04.png)
* copy `lib` and `lef` files to `design/<designanme>/src` folder
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-02%2013-24-05.png)
* edit the `design/config.tcl` file to include the paths to the new `lib`, `lef` in src folder. This ensures openlane flow picks the new lib and lef
```tcl
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot-29.png)

* open the Openlane terminal as before and open a design
```
# Open the openlane terminal
cd Desktop/work/tools/openlane_working_dir/openlane

docker

./flow.tcl -interactive

# Inside openlane terminal

# below cmd to be used to overwrite previous run data 
prep -design picorv32a -tag <prev run folder inside /runs> -overwrite 

# Cmds to include the new cell led in merged.lef in /tm[ folder
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

run_synthesis
```
* observe the reports for the default run of synthesis
The chip area info
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot-33.png)
The WNS and TNS for the design
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot-37.png)
The custom `sky130_vsdinv` in `temp/the merged.lef` indicating the cell was successfully used during synthesis
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-02%2020-15-58.png)

* To improve the timing of teh design during synthesis we can alter some of the variables (like SYNTH_SIZING,SYNTH_STRATEGY,SYNTH_BUFFERING from `configurations/README.md`) to get desired results
Variable information in `configurations/README.md`
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-02%2019-42-55.png)
Changing the variables in openlane terminal
```tcl
# Command to display current value of variable SYNTH_STRATEGY
echo $::env(SYNTH_STRATEGY)

# Command to set new value for SYNTH_STRATEGY
set ::env(SYNTH_STRATEGY) "DELAY 3"

# Command to display current value of variable SYNTH_BUFFERING to check whether it's enabled , if enabled =1
echo $::env(SYNTH_BUFFERING)

# Command to display current value of variable SYNTH_SIZING, if enabled =1
echo $::env(SYNTH_SIZING)

# Command to set new value for SYNTH_SIZING
set ::env(SYNTH_SIZING) 1

# Command to display current value of variable SYNTH_DRIVING_CELL to check whether it's the proper cell or not
echo $::env(SYNTH_DRIVING_CELL)
```
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot-35.png)

run synthesis to take all changes
```tcl
run_synthesis
```
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-02%2020-13-52.png)

the TNS and WNS seems has become 0.0, but the chip area has increased
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-02%2020-17-44.png)

* Next run the floorplan
```
run_floorplan
```
This gives an error in the run and error log in openroad shows this:
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-02%2020-54-22.png)
but there are no large macros. to circumvent this we can run each step in floorplan individually 

```
init_floorplan
place_io
tap_decap_or
```
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-02%2020-23-54.png)

```
run_placement
```
The overflow has to reduce or be closer to 0

* open the def in `magic` in anaother terminal
```bash
# enter the dir of design
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/24-03_10-03/results/placement/

# Command to load the placement def in magic tool
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-02%2021-09-16.png)

locating `sky130_vsdinv` instance in the implemented design
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-02%2021-12-52.png)

we can see the connectivity by selcting the `sky130_vsdinv` instance by keeping cursor  over it and pressing 's'. To see teh connections enter the cmd `expand` in tkconsole of `magic`

```tcl
expand
```
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-02%2021-15-59.png)

### SKY130_D4_SK2 - Timing analysis with ideal clocks using openSTA
#### April/03/2025

* *setup analysis*  refers to he minimum time a data signal must be stable before the active clock edge to ensure proper data capture by a flip-flop
* *Clock Jitter* refers to the deviation of a clock signal's edges from their supposed ideal clock edge
* *Clock uncertainity* encompasses all sources of timing variation, including jitter and other factors like skew and process variations.
* Uncertainity values are introduced in slack calculation of setup and hold slack to introduce more pessimisim to the design activity.

### Lab to config the tool to run opensta post synthesis
create `pre_sta.conf` to define the files to be loaded to the opensta for analysis in diffrent corners
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2014-46-10.png)
create `mybase.sdc` file to define timimng contraints info for the sta tool. place it in `designs/picorv32a/src`
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2014-39-17.png)
now run the opensta tool
```bash
sta pre_sta.conf
```
the analysis will be done and the results will be printed in the terminal

min (hold)
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2014-48-08.png)
max (setup)
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2014-48-45.png)

#### Lab:  Optimization
*Reopen the openlane to implemmemt any changes
```bash
docker

./flow.tcl -interactive

package require openlane 0.9

prep -design picorv32a -tag 03-04_09-49 -overwrite

# Adiitional commands to include custom lef to openlane flow
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

# Command to set new value for SYNTH_STRATEGY
set ::env(SYNTH_STRATEGY) "DELAY 3" 

# Command to set new value for SYNTH_SIZING
set ::env(SYNTH_SIZING) 1

# Command to set new value for SYNTH_MAX_FANOUT
set ::env(SYNTH_MAX_FANOUT) 4

# Now that the design is prepped and ready, we can run synthesis using following command
run_synthesis
```
open new sta cmdline in new terminal
```bash
sta pre_sta.conf
```
screenshots from the run
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2015-51-34.png)
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2015-52-34.png)
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2015-52-52.png)

### Eco implementation

* identify problematic cells from the design 
The oai cell seems to be drving 4 cells
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2016-11-33.png)
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2016-11-45.png)
```tcl
# Reports all the connections to a net
report_net -connections _04373_

# Checking command syntax
help replace_cell

# Replacing cell
replace_cell _22324_ sky130_fd_sc_hd__o21ai_4

# Generating custom timing report
report_checks -fields {net cap slew input_pins} -digits 4
```
The slack improved from -4.62 to -4.5886
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2016-26-41.png)

Nand gate of strength 2 is driving three cells
* changing drive stenght to 3 returns the message "Warning: liberty cell 'sky130_fd_sc_hd__nand2_3' not found."
* changing the drive strenght to 4
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2016-34-05.png)
the slack reduced to -4.5555 from -4.5886
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2016-35-09.png)

OAI cell of drive strenght 2 has more delay:
* changing drive stenght to 3 returns the message "Warning: liberty cell 'sky130_fd_sc_hd__o21bai_3' not found."
* changing the drive strenght to 4
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2016-43-17.png)
The slack reduced from -4.5555 to -4.5060
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2016-43-30.png)

to check the timimg through a cell we changed
```tcl
report_checks -from _35312_ -to _35239_ -through _22380_
# report_checks -from _start_pont_net_id -to end_point_net_id -through cell_id
```

### writin the ecos to new netlist

keep the old netlist in different name
```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/03-04_09-49/results/synthesis/

mv picorv32a.synthesis.v picorv32a.synthesis_old.v
```
 
Now write the new netlist in the opensta terminal

```tcl
write_verilog  write_verilog ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/03-04_09-49/results/synthesis/picorv32a.synthesis.v
```
After this stage rerunning synthesis will erase all the good changes done in opensta tool. so after `write_verilog` cmd we can return to openalne cmdline and run

```tcl

#if "run_floorplan" command gives error, following cmds do the same
init_floorplan
place_io
tap_decap_or

# Now we are ready to run placement
run_placement

# With placement finished , run CTS
run_cts
```
After the cts , new .v file is creted in the synthesis results folder 
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-03%2017-26-48.png)

### SKY130_D4_SK4 - Timing analysis with real clocks using openSTA
### April/04/2025

* openroad is is the tool around which openlane was built, openroad has inbuit sta tool which we will utilze to analyse timimg.
```bash
docker

./flow.tcl -interactive

package require openlane 0.9

#to start with previous data without overwrite tag
prep -design picorv32a -tag 03-04_09-49 
```

```tcl
# INit Openroad terminal inside openlane terminal
openroad

# Reading lef file
read_lef /openLANE_flow/designs/picorv32a/runs/03-04_09-49 /tmp/merged.lef

# Reading def file
read_def /openLANE_flow/designs/picorv32a/runs/03-04_09-49/results/cts/picorv32a.cts.def

# Creating an OpenROAD database to work with
write_db pico_cts.db

# Loading the created database in OpenROAD
read_db pico_cts.db

# Read netlist post CTS
read_verilog /openLANE_flow/designs/picorv32a/runs/03-04_09-49/results/synthesis/picorv32a.synthesis_cts.v

# Read library for design
read_liberty $::env(LIB_SYNTH_COMPLETE)

# Read in the custom sdc we created
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc

# Setting all cloks as propagated clocks
set_propagated_clock [all_clocks]

# Generating custom timing report
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

# Exit to OpenLANE flow into openRoad
exit
```
cmd runs:
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-04%2014-09-21.png)
hold slack report
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-04%2014-09-30.png)
setup slack report
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-04%2014-09-41.png)

##### removing "sky130_fd_sc_hd__clkbuf_1'" from CTS_CLK_BUFFER_LIST and reruning the cts to see the results


```tcl
# Checking current value of 'CTS_CLK_BUFFER_LIST'
echo $::env(CTS_CLK_BUFFER_LIST)

# Removing 'sky130_fd_sc_hd__clkbuf_1' from the list
set ::env(CTS_CLK_BUFFER_LIST) [lreplace $::env(CTS_CLK_BUFFER_LIST) 0 0]

# Checking current value of 'CTS_CLK_BUFFER_LIST'
echo $::env(CTS_CLK_BUFFER_LIST)

# Setting def as placement def otherwise, we get clock not found error
set ::env(CURRENT_DEF) /openLANE_flow/designs/picorv32a/runs/04-04_09-02//results/placement/picorv32a.placement.def

# Run CTS again
run_cts

# Command to run OpenROAD tool
openroad

# Reading lef file
read_lef /openLANE_flow/designs/picorv32a/runs/04-04_09-02//tmp/merged.lef

# Reading def file
read_def /openLANE_flow/designs/picorv32a/runs/04-04_09-02//results/cts/picorv32a.cts.def

# Creating an OpenROAD database to work with
write_db pico_cts.db

# Loading the created database in OpenROAD
read_db pico_cts.db

# Read netlist post CTS
read_verilog /openLANE_flow/designs/picorv32a/runs/04-04_09-02//results/synthesis/picorv32a.synthesis_cts.v

# Read library for design
read_liberty $::env(LIB_SYNTH_COMPLETE)

# Read in the custom sdc we created
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc

# Setting all cloks as propagated clocks
set_propagated_clock [all_clocks]

# Generating custom timing report
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

# Report hold skew
report_clock_skew -hold

# Report setup skew
report_clock_skew -setup

# Exit to OpenLANE flow
exit


# Checking current value of 'CTS_CLK_BUFFER_LIST'
echo $::env(CTS_CLK_BUFFER_LIST)

# Inserting 'sky130_fd_sc_hd__clkbuf_1' to first index of list
set ::env(CTS_CLK_BUFFER_LIST) [linsert $::env(CTS_CLK_BUFFER_LIST) 0 sky130_fd_sc_hd__clkbuf_1]

# Checking current value of 'CTS_CLK_BUFFER_LIST'
echo $::env(CTS_CLK_BUFFER_LIST)

```
hold slack
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-04%2015-24-35.png)
setup slack and skew values
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-04%2016-04-37.png)

before removal of "sky130_fd_sc_hd__clkbuf_1":
|Slack|Value|
|----|-----|
|hold | 0.0637   slack (MET)|
|setup | 4.5985   slack (MET)|

After removal of "sky130_fd_sc_hd__clkbuf_1":
|Slack|Value|
|----|-----|
|hold |  0.2351   slack (MET)|
|setup | 4.6943   slack (MET)|

### Sky130 Day 5 - Final steps for RTL2GDS using tritonRoute and openSTA
3 steps are done here:
* PWR Routing : due to how openlane is structured pwr routing is done at this stage, normally PDN is created during floorplan stage
* signal routing withh triton route
* Rc extraction
* OpenSTA for post route timing analysis

PWR Routing:
```tcl

# check which def file openlane is currently pointing at , it should be pointing to CTS def
echo $::env(CURRENT_DEF)

# create PDN
gen_pdn
```
pdn.def generated
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot-45.png)
view the pdn with 
```bash

cd ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/04-04_09-02/tmp/floorplan

magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../merged.lef def read 12-pdn.def &
```
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-04%2018-33-39.png)
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-04%2018-34-34.png)


Signal routing
ROUTING_STRATEGY variable isnt available, routing optimization iterations (ROUTING_OPT_ITERS) has a default value of 64
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-04%2017-47-51.png)
```tcl
run_routing
```
files genrated after the routing

![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-04%2017-20-13.png)
to visualize the def with magic
```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/04-04_09-02/results/routing/

magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.def &

```
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-04%2018-49-56.png)
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-04%2018-51-57.png)

Post route the SPEF extraction and STA happens with `run_routing` command
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot-43.png)

to verify the results open the OpenRoad tool

```tcl
openroad

# Reading lef file
read_lef /openLANE_flow/designs/picorv32a/runs/04-04_09-02/tmp/merged.lef

# Reading def file
read_def /openLANE_flow/designs/picorv32a/runs/04-04_09-02/results/routing/picorv32a.def

# Creating an OpenROAD database to work with
write_db pico_route.db

# Loading the created database in OpenROAD
read_db pico_route.db

# Read netlist post CTS
read_verilog /openLANE_flow/designs/picorv32a/runs/04-04_09-02/results/synthesis/picorv32a.synthesis_preroute.v

# Read library for design
read_liberty $::env(LIB_SYNTH_COMPLETE)

# Read in the custom sdc we created
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc

# Setting all cloks as propagated clocks
set_propagated_clock [all_clocks]

# Read SPEF
read_spef /openLANE_flow/designs/picorv32a/runs/04-04_09-02/results/routing/picorv32a.spef

# Generating custom timing report
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

# Exit to OpenLANE flow
exit
```
cmds
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-04%2017-51-50.png)
hold
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-04%2017-52-16.png)
setup
![](https://github.com/sowrabh-adiga/NASSCOM_VSD_SOC_DESIGN_PROGRAM/blob/main/files/Screenshot%20from%202025-04-04%2017-52-25.png)

Final Slack values

|Slack|Value|
|----|-----|
|hold |  0.1079   slack (MET)|
|setup | 4.3674   slack (MET)|






