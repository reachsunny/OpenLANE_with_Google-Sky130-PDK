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


**Extraction with Magic**

Before extracting and dumping the spice files, ensure there are no DRC errors in the layout.

![Invertor-Parasitic-Extraction](/Images/extraction_from_magic_layout1.PNG "inv-pext from magic")


Once the spice file is dumped from the magic layout. We can now edit the spice file to incorporate spice commands for transient analysis. See the highlighted sections.

![Spice-deck-updates](/Images/inv-ngspice-read_spice-deck.PNG "Spice-deck-updates")

Check the Voltages and capacitance values, once satisfactory 

we can now plot the **waveform {output(y) vs time with sweeping input(a)}** as shown below. With the plots, we can now identify library cell output characteristics. Some of the important stdcell parameters are:

1. Transition time (rise and fall) usually measured 20% - 80%
2. Propagation delay (rise and fall) usually measured at 50% ouput to input

![spice-plot-sweep](/Images/inv-spice-plot.PNG  "Spice-plot-sweep")

Here are example of calculating the rise and fall propagation delay for our invertor.

**Rise prop dly = @50% (out rise time -input fall time)**

**Fall prop dly = @50% (out fall time - in rise time)**

![inv-rise-dly](/Images/inv-rise-prop-dly.PNG "inv-rise-dly")
![inv-fall-dly](/Images/inv_fall_prop-dly.PNG "inv-fall-dly")
























