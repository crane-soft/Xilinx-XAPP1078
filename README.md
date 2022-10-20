## XAPP1078 2018.2 for Xilinx Board ZC702

**This project and the following informations are based on '[XAPP1078 Latest Information][1]', The original files can be found under [Updated Design Files for Vivado 2014.4 and Petalinux 2014.4 tools][2] the original Application note [here][3]**

Thanks a lot to [Gustavo][4] which helps me to build the embedded Linux kernel.

### Creating the hardware design

The instructions has been setup such that all implementation work will be done in a directory called "xapp1078_2018.2/"
* Extract xapp1078_2018.2.zip into a folder named 'xapp1078_2018.2' (or clone Git repository and chage the name to 'xapp1078_2018.2'
* Create a new work directory  xapp1078_2018.2/work
* Open Vivado 2018.2 and in the Tcl command window change to the work directory:<br>
 `cd xapp1078_2018.2/work`
* Create  the hdwr design by running the included script using the tcl command:<br>
 `source ../src/scripts/create_proj_702.tcl`

This script does the following:
* Create a new project
* Set the properties of the project such as part and board used.
* Set Vivado to use the included I.P. repository in<br>
 `xapp1078_2018.2/src/my_ip.`<br>
 This repository includes the custom IP that can create interrupts towards the PS using either a register or chipscope VIO.
* Create the IPI design using the tcl script xapp1078_2018.2/src/scripts/create_bd_702.tcl. This script was originally created after creating the IPI design manually, and then issuing the command:  
 `write_bd_tcl ../src/scripts/create_bd_702.tcl`.
* Validate and save the IPI design.
* Create the top level HDL wrapper and add it to the project. (Same as navigating to 'Project Manager', right clicking on design_1.bd, and selecting 'Create HDL Wrapper')

When the script has finished, build the design by clicking 'Generate Bitstream' in the lower left conrner.  
Then go to the filemenu and export the hardware:  
 `File/Export/Export Hardware`  
Finally lauch the SDK:  
 `File/Lauch SDK`  

### Create the Application that will run on CPU1
* Select File->New->Application_Project<br>
  Enter the project name `app_cpu1`<br>
  Change Processor to ps7_cortexa9_1<br>
  Leave 'Board Support Package' to 'Create New' 'app_cpu1_bsp' and select Next
* Select the template 'Empty Application' then select Finish.
* The 'Board Support Package Settings' file system.mss will be opened.<br>
  Select Overview->standalone and change stdin and stdout to None<br>
  Select Overview->drivers->ps7_cortexa9_1 and change the extra_compiler_flags value to contain<br>
  '-DUSE_AMP=1 -DSTDOUT_BASEADDRESS=1'<br>
  Select OK<br>
* Import the C and linkerscript files for app_cpu1 by navigating to SDK Project Explorer and right clicking on app_cpu1/src and select 'Import'<br>
  Select General->File_System then select Next<br>
  In the 'From directory', browse to and select xapp1078_2018.2/design/src/apps/app_cpu1<br>
  In the right window, select all .c, .h and lscript.ld then select Finish. Answer Yes to overwrite lscript.ld<br>

### Create, configure & build Petalinux Project

On a linux host machine that includes the Petalinux 2018.2 build environment, create and move to the directory where you want to create the petalinux project<br>
`	mkdir xapp1078_2018.2`<br>
`	cd xapp1078_2018.2`<br>
`	source /petalinux-install-dir>/settings.sh`<br>
This command will setup the Petalinux environment. 

  `petalinux-create -t project -n plnx-project --template zynq`<br>
This command creates a new petalinux project named 'plnx-project' in the current directory<br>
  `cd plnx-project`<br>
  `mkdir ../hwdef`<br>
Creates a directory that will contain the Vivado exported hdf file<br>
  `cp xapp1078_2018.2/work/project_1/project_1.sdk/design_1_wrapper.hdf ../hwdef`<br>
  This command copies the Vivado exported hdf file<br>
  `petalinux-config --get-hw-description=../hwdef`<br>
This command configures petalinux to point to the directory that contains the hdf file.<br>

When the Linux System Configuration window appears:<br>
```
  Select DTG settings --->
    Select Kernel Bootargs 
     Deselect 'generate boot args automatically' by pressing 'n'
     Select 'user set kernel bootargs' 
       Enter the following:
        console=ttyPS0,115200 earlyprintk maxcpus=1 mem=768M
		Enter OK
	  Select Exit
    Select Exit
  Select Exit	  
  Select Yes
  Saves the new configuration
```

`petalinux-create -t apps --template c --name softuart`<br>
Creates a new application template for the linux softuart function
`cp xapp1078_2018.2/src/apps/softUart/softuart.c  [dir]/plnx-project/project-spec/meta-user/recipes-apps/softuart/files/softuart.c`<br>
Replace the petalinux project's templated softuart.c with the src that was included in the xapp
`petalinux-config -c rootfs`<br>
Configure the petalinux project to include softuart on the rootfs.
Once the Linux/rootfs configuration window appears:<br>
```
  //Add softuart apps to filesystem
  Select apps
    Select softuart and press 'y'
    Select peekpoke and press 'y'
  Select Exit
  //*OPTIONAL* Enable Dropbear for ssh
    Select Filesystem Packages
      Select console/network
        Select dropbear
          Select dropbear and press 'y'
          Select Exit
        Select Exit
      Select Exit
    Select Exit
   Select Yes
   Saves the new configuration
```

`petalinux-build`<br>
Builds the petalinux project. Log is at build/build.log Generated output files are in images/linux Compiled softuart app is found in build/linux/rootfs/apps/softuart The built image.ub is a FIT format and contains: linux kernel, ramdisk, and device tree

Create the bootable file for the SD card.<br>
`cp images/linux/zynq_fsbl.elf <xapp1078_2018.2>/work/bootgen`<br>
`cp images/linux/u-boot.elf <xapp1078_2018.2>/work/bootgen`<br>
In Windows: Within SDK, select Xilinx_tools->launch_shell A new command shell is started with an environment pointing to the SDK and Vivado tools<br>
`cd ..\bootgen createBoot.bat`<br>
This batch file runs bootgen and uses bootimage.bif for input. The bif is used to package the fsbl, u-boot, cpu1 app, and fpga bit file into boot.bin

In Linux cd <xapp1078_2018.2>/work/bootgen bootgen -image bootimage.bif -o i BOOT.BIN -w on This command (bootgen) and uses bootimage.bif for input.The bif is used to package the fsbl, u-boot, cpu1 app, and fpga bit file into boot.bin

Copy the boot and embedded Linux files to the SD card.<br>
`cp <xapp1078_2018.2>/work/bootgen/BOOT.BIN  -> SD-card`
`cp <xapp1078_2018.2>/plnx-project/images/linux/image.ub -> SD-card`

### Running the Design

Boot the board from the SD card and open a terminal emulator configured for 115200 baud. U-boot will automatically run sdboot and start the kernel. If old uboot environment settings have been stored, it may be necessary to enter uboot and run the following command which will set the environment to the defaults that were defined during the petalinux build:<br>
`env default -a saveenv`

The kernel will boot. Login as user 'root' and passwd 'root'

Enter the linux commands:<br>
`peek 0xffff8000`<br>
Checks the heartbeat of CP1. It should be 0 because cpu1 hasn't started.<br>
`poke 0xfffffff0 0x30000000`<br>
Start cpu1 by writing a start address to the wfe loop.<br>
`peek 0xffff8000`<br>
Checks the heartbeat. cpu1 is now started so the read value should increment every second.<br>
`softuart&`<br>
Start the softuart application and run it in the background.<br>
`poke 0x78600000 0x00000001`<br>
Using Linux on cpu0, create an interrupt towards cpu1 by writing to the control register within the irq_gen_0 core. Cpu1 will received an interrupt then service it. Upon servicing the irq, cpu1 uses OCM to communicate to the Linux softuart app. Cpu1 will cause the app to display:<br>
`CPU1: IRQ clr `<br>

  [1]: https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841653/XAPP1078+Latest+Information
  [2]: https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841653/XAPP1078+Latest+Information#XAPP1078LatestInformation-UpdatedDesignFilesforVivado2014.4andPetalinux2014.4tools
  [3]: https://github.com/crane-soft/Xilinx-XAPP1078/tree/main/docs/xapp1078-amp-linux-bare-metal.pdf
  [4]: https://github.com/gustandil/xapp1078_2018.3_zybo_zed
  