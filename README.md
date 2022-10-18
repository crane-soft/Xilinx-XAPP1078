## XAPP1078 2014.4
**This is the original version which can be found in '[XAPP1078 Latest Information][1]' under [Updated Design Files for Vivado 2014.4 and Petalinux 2014.4 tools][2]**

The design files have been updated for Vivado 2014.4 and the instructions use Petalinux to build the Linux kernel, ramdisk, etc.There are scripts and instructions to target the following boards: ZC702, ZC706, and the Avnet MicroZed board.

There are also scripts for the Avnet ZedBoard but due to the reduced amount of DDR (512MB), the BSP and cpu1 linkerscript would require modifications. And the Linux kernel bootargs may need the 'mem=' value reduced.

In addition to updating the design to 2014.4, the following additional changes have been made:

* Updated standalone BSP to v4.2 then merged in changes used for 2014.2
* Modified BSP to include capture code for cpu0 and cpu1. This code is not exercised by XAPP1078
* Modified BSP to disable access to DDR between 0x00000000-0x2FFFFFFF
* Modified BSP attributes of DDR address range 0x30000000-0x3FFFFFFF to non-shared and outer cache disabled
* Modified CPU1 app to use the SCU timer instead of global timer via a #define. As long as profiling is not enabled, this change will allow cpu1 to be started either by FSBL or U-Boot.


  [1]: https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841653/XAPP1078+Latest+Information
  [2]: https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841653/XAPP1078+Latest+Information#XAPP1078LatestInformation-UpdatedDesignFilesforVivado2014.4andPetalinux2014.4tools