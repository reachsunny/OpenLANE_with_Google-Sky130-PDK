# Project

## Experimentation and Learning on fully open-sourced RTL2GDS OpenLANE flow inspired by VSD Workshop

**RTL2GDS flow steps with OpenLANE**

**Invertor cell design using Magic**

**Parasitic extraction and spice files**

**Spice Simulation using ngspice**


# Table of Contents

1. [OpenLANE tools and Flows](#openlane_with_google-sky130-pdk)
2. [Useful GIT_REPOS](#useful-git-repos)
3. [Setting Up Virtual Machine](#setting-up-virtual-machine-and-openlane-flows)
4. [Design Prep](#design-prep)
5. [Synthesis](#synthesis)
6. [Floorplan](#floorplanning)
7. [Placement](#placement)
8. [Design Library Cell](#design-library-cell)
   * [View in magic layout](#design-library-cell)  
   * [Parasitic extraction & spice deck](#extraction-with-magic)
   * [Lib cell characteristics](#cell-characterization)
   * [LEF generation](#lef-generation-from-magic-layout)
9. [Clock Tree Synthesis](#clock-tree-synthesis-cts) 

# OpenLANE_with_Google-Sky130-PDK

OpenLANE is an automated RTL2GDS flows that incorporates multiple open source tools to perform ASIC design activities. Tools used in the OpenLANE flows are listed below:

![tools](/Images/OpenLane_tools.PNG "OpenLANE-tools")

**Overall RTL2GDS OpenLane Flow**

![Flow](/Images/openlane.flow.1.png "OpenLANE-flow")

# Useful git-repos

https://github.com/efabless/openlane

https://github.com/google/skywater-pdk

https://github.com/kunalg123/vsdflow

https://github.com/nickson-jose/openlane_build_script


#  Setting up Virtual Machine and OpenLANE flows
A good disk-space to start with would be ~40G with 4GB RAM. Additional space and memory should be alloted based on your project needs

*  To setup VM on your local machine [Setting up VirtualBox](https://www.youtube.com/watch?v=x5MhydijWmc)
*  To setup OpenLANE flows [Installing OpenLANE flow](https://www.udemy.com/course/vsd-a-complete-guide-to-install-openlane-and-sky130nm-pdk/learn/lecture/21989274#overview)

# Design Prep
If you are using the VM, then you need to setup PDK_ROOT and docker before invoking the OpenLANE flow as your branch version and local path.
<pre>export PDK_ROOT=&apos;absolute-path-to-your-pdks&apos;
alias docker=&apos;docker run -it -v $(pwd):/openLANE_flow -v $PDK_ROOT:$PDK_ROOT -e PDK_ROOT=$PDK_ROOT -u $(id -u $USER):$(id -g $USER) openlane:rc2&apos;

</pre>


![Design Prep](/Images/OL-prep_stage.png)

# Synthesis
For Synthesis OpenLANE uses following tools
1. yosys for RTL synthesis optmization without technology mapping
2. ABC for doing technology mapping as per libraries provided
3. OpenSTA for doing timing analysis

Command to use: *run_synthesis*
Once completed, it will report out the design statistics. Here is an example for design picorv32a

![Synthesis](/Images/synth-stats.png "Synthesis")

You can do some simple analysis on the design:

Flop Ratio   = Number of Flops / Total number of cells

Buffer Ratio = Total number of buffers / Total number of cells

Chip Design Area , Timing reports etc.

# Floorplanning 

Floorplanning is the step to experiment on establishing the die and core size of your chip. IO ports locations, pre-place cells planning, power-distribution networks. Its and iterative process to understand a good-floorplan and bad-floorplan.

Command to use: *run_floorplan*
It invokes multiple openroad commands: ioplacer, pdn, tapcell insersion and defined core and die areas

Based on the config varaibles for floorplan it will perform the IO placement, create power-distribution network, Insert tap-cells 

Some of the floorplan config variables:
Here the default values for **FP_IO_VMETAL and FP_IO_HMETAL** has been changed to one-layer up. 

![FP-config-vars](/Images/FP-var_settings.png "FP-vars")

Also the **FP_IO_MODE is 1** that means all the IO pins even though placed randomly, but will be at an equidistant.

![FP-magic](/Images/FP-magic-view1.png "Floorplan magic layout view")

![FP-IO-place](/Images/FP-magic-view2.png "IO-Placement")

Another important factor during floorplanning is the design **ASPECT RATIO = Height / Width**

![FP-AR](/Images/FP-AR.png "FP Aspect Ratio")




# Placement

Once the floorplanning stage is satisfactory, we can now legally place the standard cells on the pre-defined site-rows. This will ensure the standard-cell power will line up to the pgn network grids. OpenLANE invokes multiple tools to provide optmized legal placed design:

Command to use: *run_placement*

**RePlace** will perform coarse global placement

**Resize** will size up cells to meet the timing, power and area targets

**OpenPhySyn** will perform optimization to improve the quality of placement

**OpenDP** will complete the placement process with legalized and optimized design


Here is the snapshot of the design after floorplanning and after placement.

![FP-and-Place](/Images/FP_and_Place-magic.png "FP and Place")

# Design Library Cell

[Magic](http://opencircuitdesign.com/magic/index.html) will be used as a layout tool for simple invertor design. Extracting its parasitics and writing out spice file that can be used with ngspice for simulation. The magic invertor was taken from [vsdstdcelldesign](https://github.com/nickson-jose/vsdstdcelldesign)

To invoke the layout of designed invertor in Magic: Use relative paths to the .tech and .mag files.

<pre>magic -T sky130A.tech sky130_inv.mag &amp;</pre>

![Invertor](/Images/sky130A-inv-design.PNG "Invertor")


### Extraction with Magic**

Before extracting and dumping the spice files, ensure there are no DRC errors in the layout.

![Invertor-Parasitic-Extraction](/Images/extraction_from_magic_layout1.PNG "inv-pext from magic")


Once the spice file is dumped from the magic layout. We can now edit the spice file to incorporate spice commands for transient analysis. See the highlighted sections.

![Spice-deck-updates](/Images/inv-ngspice-read_spice-deck.PNG "Spice-deck-updates")

Check the Voltages and capacitance values, once satisfactory 

we can now plot the **waveform {output(y) vs time with sweeping input(a)}** as shown below. With the plots, we can now identify library cell output characteristics. Some of the important stdcell parameters are:

### Cell Characterization

1. Transition time (rise and fall) usually measured 20% - 80%
2. Propagation delay (rise and fall) usually measured at 50% ouput to input

![spice-plot-sweep](/Images/inv-spice-plot.PNG  "Spice-plot-sweep")

Here are example of calculating the rise and fall propagation delay for our invertor.

**Rise prop dly = @50% (out rise time -input fall time)**

**Fall prop dly = @50% (out fall time - in rise time)**

![inv-rise-dly](/Images/inv-rise-prop-dly.PNG "inv-rise-dly")
![inv-fall-dly](/Images/inv_fall_prop-dly.PNG "inv-fall-dly")

### LEF generation from magic layout**

LEF can be directly generated from the Magic layout tool using *write lef* and this lef can be used to integrate this custom invertor cell into our design.

![inv-lef](/Images/inv-LEF-generation.PNG "inv-lef")

### Integrate custom lib cell in our design

Copy the LEF file for our custom cell and the lib files into our design src area:

<pre>cp sky130_inv.lef /home/sunny/Desktop/OS-RTL2GDS/vsdflow/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/</pre>

<pre>cp sky130_fd_sc_hd__* /home/sunny/Desktop/OS-RTL2GDS/vsdflow/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
</pre>

Now we should have the following files in your design/picorv32a/src area
<pre><font color="#4E9A06"><b>sunny@sunny-VirtualBox</b></font>:<font color="#3465A4"><b>~/Desktop/OS-RTL2GDS/vsdflow/work/tools/openlane_working_dir/openlane/designs/picorv32a/src</b></font>$ 
picorv32a.sdc
picorv32a.v
sky130_fd_sc_hd__fast.lib
sky130_fd_sc_hd__slow.lib
sky130_fd_sc_hd__typical.lib
sky130_inv.lef
</pre>

Update the design/picorv32a/config.tcl file to include the LIBS and LEFs
<pre>#Sunny Added the following to include the custom designed invertor cell
set ::env(LIB_SYNTH) &quot;$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib&quot;
set ::env(LIB_SYNTH) &quot;$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib&quot;
set ::env(LIB_SYNTH) &quot;$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib&quot;
set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/picorv32a/src/*.lef]
###########

</pre>

**Include the lefs after the prep stage:**

<pre>bash-4.1$ ./flow.tcl -interactive
<font color="#06989A">[INFO]: </font>
<font color="#06989A">      ___   ____   ___  ____   _       ____  ____     ___</font>
<font color="#06989A">     /   \ |    \ /  _]|    \ | |     /    ||    \   /  _]</font>
<font color="#06989A">    |     ||  o  )  [_ |  _  || |    |  o  ||  _  | /  [_</font>
<font color="#06989A">    |  O  ||   _/    _]|  |  || |___ |     ||  |  ||    _]</font>
<font color="#06989A">    |     ||  | |   [_ |  |  ||     ||  _  ||  |  ||   [_</font>
<font color="#06989A">     \___/ |__| |_____||__|__||_____||__|__||__|__||_____|</font>


<font color="#06989A">[INFO]: Version: rc2</font>
<font color="#D3D7CF">% package require openlane 0.9</font>
<font color="#D3D7CF">0.9</font>
<font color="#D3D7CF">% prep -design picorv32a -tag custom_inv </font>
<font color="#06989A">[INFO]: Using design configuration at /openLANE_flow/designs/picorv32a/config.tcl</font>
<font color="#D3D7CF">mergeLef.py : Merging LEFs</font>
<font color="#D3D7CF">sky130_fd_sc_hd.lef: SITEs matched found: 0</font>
<font color="#D3D7CF">sky130_fd_sc_hd.lef: MACROs matched found: 437</font>
<font color="#D3D7CF">mergeLef.py : Merging LEFs complete</font>
<font color="#D3D7CF">padLefMacro.py : Padding technology lef file</font>
<font color="#D3D7CF">Derived SITE width (microns): 0.46</font>
<font color="#D3D7CF">Derived SITE height (microns): 5.44</font>
<font color="#D3D7CF">Right cell padding (microns): 3.68</font>
<font color="#D3D7CF">Left cell padding (microns): 0.0</font>
<font color="#D3D7CF">Top cell padding (microns): 0.0</font>
<font color="#D3D7CF">Bottom cell padding (microns): 0.0</font>
<font color="#D3D7CF">Skipping LEF padding for MACRO  sky130_fd_sc_hd__tapvgnd_1</font>
<font color="#D3D7CF">Skipping LEF padding for MACRO  sky130_fd_sc_hd__tap_1</font>
<font color="#D3D7CF">Skipping LEF padding for MACRO  sky130_fd_sc_hd__tapvgnd2_1</font>
<font color="#D3D7CF">Skipping LEF padding for MACRO  sky130_fd_sc_hd__decap_3</font>
<font color="#D3D7CF">Skipping LEF padding for MACRO  sky130_fd_sc_hd__decap_4</font>
<font color="#D3D7CF">Skipping LEF padding for MACRO  sky130_fd_sc_hd__tapvpwrvgnd_1</font>
<font color="#D3D7CF">Skipping LEF padding for MACRO  sky130_fd_sc_hd__decap_12</font>
<font color="#D3D7CF">Skipping LEF padding for MACRO  sky130_fd_sc_hd__fill_1</font>
<font color="#D3D7CF">Skipping LEF padding for MACRO  sky130_fd_sc_hd__fill_4</font>
<font color="#D3D7CF">Skipping LEF padding for MACRO  sky130_fd_sc_hd__decap_8</font>
<font color="#D3D7CF">Skipping LEF padding for MACRO  sky130_fd_sc_hd__fill_2</font>
<font color="#D3D7CF">Skipping LEF padding for MACRO  sky130_fd_sc_hd__decap_6</font>
<font color="#D3D7CF">Skipping LEF padding for MACRO  sky130_fd_sc_hd__fill_8</font>
<font color="#D3D7CF">Skipping LEF padding for MACRO  sky130_fd_sc_hd__tap_2</font>
<font color="#D3D7CF">padLefMacro.py : Finished</font>
<font color="#06989A">[INFO]: Preparation complete</font>
<font color="#D3D7CF">% set lefs [glob $::env(DESIGN_DIR)/src/*.lef]</font>
<font color="#D3D7CF">/openLANE_flow/designs/picorv32a/src/sky130_inv.lef</font>
<font color="#D3D7CF">% add_lefs -src $lefs</font>
<font color="#06989A">[INFO]: Merging /openLANE_flow/designs/picorv32a/src/sky130_inv.lef</font>
</pre>

Now, reruning the synthesis and placement, we should be able to see our custom designed invertor cell

![custom-inv-in-synth](/Images/synth-with-custom-inv.PNG "custom-inv-in-synth")

**Post-placement: Viewing in Magic layout**
<pre>magic -T ../../../pdks/sky130A/libs.tech/magic/sky130A.tech lef read runs/custom_inv/tmp/merged.lef def read runs/custom_inv/results/placement/picorv32a.placement.def &amp;</pre>

![custom-inv-in-magic](/Images/custom-inv-in-magic.PNG "custom-inv-in-magic")


# Clock tree synthesis CTS

So far we have been using ideal clocks in our design. After a legalized placed design, we can perform clock-tree-synthesis. Main function of CTS will be to minimize clock-skew and clock-propagation (latency) delays for the design. After CTS, new clock buffers will be inserted into the design and new def will be created.

To run CTS use: *run_cts* command 






























