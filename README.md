# Open-Source-Physical-Design
Physical design (RTL2GDSII) flow using OpenLANE open source flow 

## Open Source EDA, Open LANE and Sky130 PDK

### OpenLANE Directory Structure

<img width="877" alt="directory" src="https://user-images.githubusercontent.com/61197146/183237894-1c31b5cc-d8a4-4ae3-b3a8-0bb6b9072cef.png">

The directory that we use is /openlane_working_dir, which has openlane and pdks subdirectories of which we use openlane.

### Opening OpenLANE and Design Preparation 

Once we run the docker command, run the following:

`./flow.tcl -interactive`
`package require openlane 0.9`

OpenLANE is now ready to be used and the comman line prompt changes.

<img width="776" alt="openlane" src="https://user-images.githubusercontent.com/61197146/183237935-dbdee636-f9d6-4e32-8184-808fd11c1d95.png">

The design setup needs to be done in order to help fetch files from a specific location. Run the command 
`prep -design picorv32a`

<img width="1424" alt="prep complete" src="https://user-images.githubusercontent.com/61197146/183237915-7560b1b0-04a6-4d2b-865a-e5c444e0fcfb.png">

In designs/picorv32a, a runs folder should be created with a date folder, where results, reports, tmp, etc. are stored. (Results, reports, etc. will be empty pre-synthesis.)

### Synthesis and Post-Synthesis Characterisation

`run_synthesis`
The command runs synthesis using yosys and technology mapping using abc.

<img width="1420" alt="synth success" src="https://user-images.githubusercontent.com/61197146/183237948-08b86912-6405-4f45-bc71-d4c733fea8b9.png">

Once synthesis is complete, the chip area, no. of cells, flops, etc. can be seen. The flop ratio for given run is #flops/#cells=1613/18036=8.94%
<img width="469" alt="chip area" src="https://user-images.githubusercontent.com/61197146/183237953-c46b8774-3f33-4fd4-b10c-2e7aa1c0cab2.png">

<img width="863" alt="dff 1613" src="https://user-images.githubusercontent.com/61197146/183237962-2356848a-a53c-4122-a1c6-6e41720bbfb6.png">

After synthesis, runs/(date)/results gets populated. /(date)/synthesis gets the synthesised netlist (.v).

`less picorv32a.synthesis.v`
This can be used to open the netlist. It shows that abc mapping has been done.

In reports/synthesis, `less yosys_2_stat.rpt` gives the reports of the run. `opensta_main.timing.rpt` gives the prelayout timing response.

## Floorplan and Placement

### Setting floorplan switches, floorplanning and verifying parameters post floorplan

The floorplan stage is done post synthesis. During floorplan, areas of the die and core, aspect ratio of the chip, core utilisation factor are set. The I/O cells and macros are placed and power distribution network is laid. Standard cells are not placed during floorplan, this is done in the placement stage.

OpenLANE switches help us configure the floorplan settings, and the default parameters can be set in openlane/configurations/floorplan.tcl. The default values of each stage of the design flow can be seen in the README file in the configuration subdirectory. In openlane/designs/picorv32a, along with the runs folder, config.tcl and sky130A_sky130_fd_sc_hd_config.tcl (highest priority) files are present, where the floorplan parameters can be changed at a higher level of precedence above the defaults.

To run floorplan, the command is `run_floorplan`. Once floorplan is successful, "Running tapcell...Done" is displayed in the terminal. Successful floorplan results in a .def file (design exchange format) being created.

<img width="1349" alt="floorplan" src="https://user-images.githubusercontent.com/61197146/183240284-b5b42da1-6c27-4a05-a4fd-6bb28e36a255.png">

<img width="518" alt="Screenshot 2022-08-06 at 11 37 49" src="https://user-images.githubusercontent.com/61197146/183240272-dda16b06-b960-4eb8-9d26-7d09487ffa07.png">

To check the values taken up by the run in the floorplan log, we go to the path picorv32a/runs/(date folder)/logs/floorplan and give the command `less ioPlacer.log` to view the log file (shift+G to go to end of file and view vertical and horizontal metal layers). The value taken up in the run, which is present in the log file, is given input value+1. It can be verified that the input value given by user in .tcl file overrides system defaults and obeys the order of precedence. The die area can be calculated from the .def file die coordinates.

### Viewing floorplan using magic

Run in results/floorplan-

`magic -T  <path to .tech file: /home/..........pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &`

-T: tech file
lef read (merged lef file in date folder two levels above current)- since the tech lef and cell lef files were combined in previous stage 
& - free command line

Magic opens with the floorplan as shown-

<img width="1223" alt="Screenshot 2022-08-06 at 13 04 51" src="https://user-images.githubusercontent.com/61197146/183239489-9521a7cd-257c-4a5d-beee-4fd233a97ba0.png">

The tkcon window can help us get information such as which metal layer a given component belongs to, to help us verify this against our input parameter (such as metal layer set in io horiz metal). Decap cells and tap cells (to avoid latchup) are observed, with all standard cells being kept in the bottom left corner in layout.

### Placement
