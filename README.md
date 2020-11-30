# Table of Contents

1. [OpenLANE tools and Flows](#openlane_with_google-sky130-pdk)
2. [Useful GIT_REPOS](#useful-git-repos)
3. [Setting Up Virtual Machine](#setting-up-virtual-machine-and-openlane-flows)
4. [Design Prep](#design-prep)
5. [Synthesis](#Synthesis)

# OpenLANE_with_Google-Sky130-PDK

**Experimentation and Learning on fully open-sourced RTL2GDS OpenLANE flows**


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
<pre>export PDK_ROOT=&apos;<absolute-path-to-your-pdks>&apos;
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

![Synthesis](/Images/synth-stats.png)

You can do some simple analysis on the design:

Flop Ratio   = Number of Flops / Total number of cells

Buffer Ratio = Total number of buffers / Total number of cells

Chip Design Area , Timing reports etc.







