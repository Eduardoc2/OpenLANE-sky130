# OpenSource Physical Design
  This repository contains all the information studied and created during the Advanced Physical Design Using OpenLANE / SKY130 workshop. It is primarily foucused on a complete RTL2GDS flow using the open-soucre flow named OpenLANE.
  
# Introduction To RTL to GDSII Flow
  RTL to GDSII Flow refers to the all the steps involved in converting a logical Register Transfer Level(RTL) Design to a fabrication ready GDSII format. GDSII is a database file format which is an industry standard for data exchange of IC layout artwork.
  The RTL to GSDII flow consists of following steps:
  - RTL Synthesis
  - Static Timing Analysis(STA)
  - Design for Testability(DFT)
  - Floorplanning
  - Placement
  - Clock Tree Synthesis(CTS)
  - Routing
  - GDSII Streaming
  ### What is OpenLANE
   [OpenLANE](https://github.com/efabless/openlane) is an automated RTL to GDSII flow which includes various open-source components such as OpenROAD, Yosys, Magic, Fault, Netgen, SPEF-Extractor. It also facilitates to add custom design exploration and optimization scripts.
   The detailed diagram of the OpenLANE architecture is shown below:
   
   <img src="Images/openlane_flow.png">
   
   OpenLANE flow consists of several stages. By default all flow steps are run in sequence. Each stage may consist of multiple sub-stages. OpenLANE can also be run interactively as shown here.

  1. Synthesis
      1. `yosys` - Performs RTL synthesis
      2. `abc` - Performs technology mapping
      3. `OpenSTA` - Pefroms static timing analysis on the resulting netlist to generate timing reports
  2. Floorplan and PDN
      1. `init_fp` - Defines the core area for the macro as well as the rows (used for placement) and the tracks (used for routing)
      2. `ioplacer` - Places the macro input and output ports
      3. `pdn` - Generates the power distribution network
      4. `tapcell` - Inserts welltap and decap cells in the floorplan
  3. Placement
      1. `RePLace` - Performs global placement
      2. `Resizer` - Performs optional optimizations on the design
      3. `OpenPhySyn` - Performs timing optimizations on the design
      4. `OpenDP` - Perfroms detailed placement to legalize the globally placed components
  4. CTS
      1. `TritonCTS` - Synthesizes the clock distribution network (the clock tree)
5. Routing *
      1. `FastRoute` - Performs global routing to generate a guide file for the detailed router
      2. `TritonRoute` - Performs detailed routing
      3. `SPEF-Extractor` - Performs SPEF extraction
  6. GDSII Generation
      1. `Magic` - Streams out the final GDSII layout file from the routed def
  7. Checks
      1. `Magic` - Performs DRC Checks & Antenna Checks
      2. `Netgen` - Performs LVS Checks
  
# Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK
    
 ## Open-Source EDA Tools
 ### OpenLANE Initialization
   First we need to make sure we have the following files inside the openlane folder
   
   <img src="Images/D1_pdk_directory.png">

   For invoking OpenLANE we should first run the docker everytime we use OpenLANE. This is done by using the following script:
    
    docker 
    
   A custom shell script or commands will be generated.
   - To invoke OpenLANE run the `./flow.tcl` script.
    
    ./flow.tcl -interactive
   
   The first step after invoking OpenLANE is to import the openlane package of required version. 
   
    package require openlane 0.9
    
   The next step is to prepare our design for the OpenLANE flow. This is done using following command
       
    prep -design picorv32a
   
   <img src="Images/openlane_design_prep.png"> 
   
   During the design preparation the technology LEF and cell LEF files are merged together to obtain a `merged.lef` file. The LEF file contains information like the layer information, set of design rules, information about each standard cell which is required for place and route. 
    
 ### Design Synthesis and Results
   The first step in OpenLANE flow is RTL Synthesis of the design loaded. This is done using the following command.
   
    run_synthesis
   
   After running the synthesis we can see some statistics
   
   <img src="Images/synthesis_detail.png">
   
# Day 2 - Good floorplan vs bad floorplan and introduction to library cells
 ## Chip Floorplanning
   Chip Floorplanning is the arrangement of logical block, library cells, pins on silicon chip. It makes sure that every module has been assigned an appropriate area and aspect ratio, every pin of the module has connection with other modules or periphery of the chip and modules are arranged in a way such that it consumes lesser area on a chip.
 
 ### Floorplan using OpenLANE
   Floorplanning in OpenLANE is done using the following command. 
    
    run_floorplan
   
   Successful floorplanning gives a `def` file as output. This file contains the die area and placement of standard cells.
   
   <img src="Images/d2_floorplan.png">
 
 ### Review Floorplan Layout in Magic
   Magic Layout Tool is used for visualizing the layout after floorplan. In order to view floorplan in Magic, following three files are required:
    1. Technology File (`sky130A.tech`)
    2. Merged LEF file (`merged.lef`)
    3. DEF File
    
        magic -T <file path/sky130A.tech> lef read <file path/merged.lef> def read <file path/picorv32a.floorplan.def>
    
   <img src="Images/d2_floorplan_magic.png">
   <img src="Images/d2_floorplan_magic_zoom.png">
   <img src="Images/d2_floorplan_magic_zoom2.png">
 
 ## Placement
 ### Placement and Optimization
   The next step after floorplanning is placement. Placement determines location of each of the components on the die. Placement does not just place the standard cells available in the synthesized netlist. It also optimizes the design, thereby removing any timing violations created due to the relative placement on die.
   
   Placement in OpenLANE is done using the following command. 
    
    run_placement
   
   The DEF file created during floorplan is used as an input to placement. Placement in OpenLANE occurs in two stages:t
   
   Placement is carried out as an iterative process till the value of overflow converges to 0.
   
   <img src="Images/d2_placement_finish.png">
   <img src="Images/d2_plac_magic.png">
   <table border="0"><tr><td><img src="Images/d2_plac_mag2.png"> </td><td> <img src="Images/d2_plac_mag3.png"> </td></tr></table>
 
# Day 3 - Design library cell using Magic Layout and ngspice characterization
  Every Design is represented by equivalent cell design. All the standard cell designs are available in the Cell Library. A fully custom cell design that meets all rules can be added to the library. To begin with, a CMOS Inverter is designed in Magic Layout Tool and analysis is carried out using NGSPICE tool.
  
 ## CMOS Inverter Design using Magic
  The inverter design is done using Magic Layout Tool. It takes the technology file as an input (`sky130A.tech` in this case). 
  The snippet below shows a layout for CMOS Inverter with and without design rule violations.
  
  <img src="Images/d3_inv_magic.png">
  <img src="Images/d3_inv_drc.png">
  <img src="Images/d3_inv_drc_nice.png">
  
 ## Extract SPICE Netlist from Standard Cell Layout
  To simulate and verify the functionality of the standard cell layout designed, there is a need of SPICE netlist of a given layout.
  Extraction of SPICE model for a given layout is done in two stages.
  1. Extract the circuit from the layout design.
  
    extract all
  
  2. Convert the extracted circuit to SPICE model.
    
    ext2spice cthresh 0 rthresh 0
    ext2spice
   
  <img src="Images/d3_extraction.png">   
   
  The extracted SPICE model like the first snippet shown below. Some modification are done to the SPICE netlist for the purpose of simulations, which is shown in the second snippet below.
  
  <img src="Images/d3_spice.png">
  <img src="Images/d3_modi_spice.png">
  
 ## Transient Analysis using NGSPICE
  The SPICE netlist generated in previous step is simulated using the NGSPICE tool. NGSPICE is an open-source mixed-level/mixed-signal electronic spice circuit simulator.
  The command used to invoke NGSPICE is shown below.
  
    ngspice <name-of-SPICE-netlist-file>
    
   <img src="Images/d3_ngspice.png">
    
  Following command is used to plot waveform in ngspice tool.
    
    ngspice 1 -> plot Y vs time a
   
   Below figure shows the waveform of Inverter output vs input w.r.t. time. Many timing parameters like rise time delay, fall time delay, propagation delay are calculated using this waveform.
   
   <img src="Images/d3_plot.png">
   <img src="Images/d3_ext_file.png">
  
# Day 4 - Pre-layout timing analysis and importance of good clock tree
 ## Magic Layout to Standard Cell LEF
  Before creating the LEF file we require some details about the layers in the designs. This details are available in a `tracks.info` as shown below. 
    
  <img src="Images/d4_tracks.png">
  
  It gives information about the `offset` and `pitch` of a track in a given layer both in horizontal and vertical direction. The track information is given in below mentioned format.
  
    <layer-name> <X-or-Y> <track-offset> <track-pitch>
  
  To create a standard cell LEF from an existing layout, some important aspects need to be taken into consideration.
  1. The height of cell be appropriate, so that the `VPWR` and `VGND` properly fall on the power distribution network.
  2. The width of cell should be an odd multiple of the minimum permissible grid size.
  3. The input and ouptut of the cell fall on intersection of the vertical and horizontal grid line.
  
  Standard cell in Magic
  
  <img src="Images/d4_mag.png">  
  <img src="Images/d4_lef_write.png">
  
  Then we can see the .lef file
  
  <img src="Images/d4_lef.png">
  
 ## Timing Analysis using OpenSTA
  The Static Timing Analysis(STA) of the design is carried out using the OpenSTA tool.
   
  OpenSTA is invoked using the below mentioned command.
  
    sta <conf-file-if-required>
  
  The above command gives an Timing Analysis Report which contains:
   1. Hold Time Slack
   2. Setup Time Slack
   3. Total Negative Slack (= 0.00, if no negative slack)
   4. Worst Negative Slack (= 0.00, if no negative slack)
   
   First we look at the file configuration
  
  <img src="Images/d4_config_tcl.png">
  <img src="Images/d4_p3.png">
  
  Then we overwrite the existing directory
  
  <img src="Images/d4_prep_inv.png">
  <img src="Images/d4_prep_complete.png">
  <img src="Images/d4_syn_complete.png">
  
  We need to make some adjustments
  
  <img src="Images/d4_changes.png">
  <img src="Images/d4_floor_plac_complete.png">
  <img src="Images/d4_chip_mag.png">
  <img src="Images/d4_chip_mag2.png">
  
 ## Clock Tree Synthesis using TritonCTS
  Clock Tree Synthesis(CTS) is a process which makes sure that the clock gets distributed evenly to all sequential elements in a design. The goal of CTS is to minimize the clock latency and skew.
  
  In OpenLANE, clock tree synthesis is carried out using TritonCTS tool. CTS should always be done after the floorplanning and placement as the CTS is carried out on a `placement.def` file that is created during placement stage.
  
  Checking the files
    
   <img src="Images/d4_mybase.png">
   <img src="Images/d4_p1.png">
   <img src="Images/d4_p2.png">
 
   
   The command used for running CTS in OpenLANE is given below.
  
    run_cts
   
   <img src="Images/d4_p5.png">
   <img src="Images/d4_p6.png">

# Day 5 - Final steps for RTL2GDS
 ## Generation of Power Distribution Network
   In a normal RTL to GDSII flow the generation of power distribution network is done before the placement step, but in the OpenLANE flow generation of PDN is carried out after the Clock Tree Synthesis(CTS). This step generates all the tracks, rails required for routing power to entire chip.
   Generation of power distribution network is done using following command.
   
    gen_pdn
    
   <img src="Images/d5_run_gen_pdn.png">
   <img src="Images/d5_pnd.png">
   <img src="Images/d5_1.png">
   
 ## Routing using TritonRoute
   OpenLANE uses TritonRoute, an open source router for modern industrial designs. The router consists of several main building blocks, including pin access analysis, track assignment, initial detailed routing, search and repair, and a DRC engine.
   
   The following command is used for routing.
   
    run_routing
    
   <img src="Images/d5_run_routing.png">
   
   Checking the files
   
   <img src="Images/d5_routing_2.png">
   <img src="Images/d5_routing_3.png">
    
 ## SPEF File Generation
   Standard Parasitic Exchange Format (SPEF) is an IEEE standard for representing parasitic data of wires in a chip in ASCII format. Non-ideal wires have parasitic resistance and capacitance that are captured by SPEF. 
    
   The below snippet shows a small part of the `.spef` file.
   
   <img src="Images/d5_spef.png">
   
 ## Magic
 
 After finishing the routing we generate the .mag and .gds files.
   
    run_magic
 
 <table border="0"><tr><td><img src="Images/d5_picorv32a.png"> </td><td> <img src="Images/d5_gnd.png"> </td></tr></table>
 <table border="0"><tr><td><img src="Images/d5_zoom2.png"> </td><td> <img src="Images/d5_zoom3.png"> </td></tr></table>
 
