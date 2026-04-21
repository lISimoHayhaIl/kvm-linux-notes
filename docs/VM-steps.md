> This is my personal guide which I follow to setup my Windows 10 Virtual Machine, after an update, re-install of OS, etc.

# STEPS
1. Apply BIOS settings from docs/BIOS-Settings.md file
2. Install VFIO for the correct distro. Follow steps from docs/distros/DISTRO/setup.md
3. Open KVM Manager, go to Edit > Preferences and enable "Enable XML editing"
5. Create a new virtual machine using KVM Manager, use "Manual Install" method
6. If you are going to passthrough a SSD, then de-select "Enable storage for this virtual device"
7. Add your VM name, and select "Customise configuration before install" in order to pass-through devices
8. Before starting the installation, open configs/win10-games.xml file and copy sections to your new XML configuration
9. DO NOT overwrite the GUID at the beginning of the new XML, and replace the one from win10-games.xml with this one. It is unique for each VM
10. Sections you need to copy are:
    <memory/>
    <currentMemory/>
    <memoryBacking/>
    <vcpu/>
    <iothreads/>
    <cputune/>
    <sysinfo/> (change accordingly and replace the GUID)
    <features/>
    <cpu/>
    <clock/>
11. Add your pass-through devices*. For VGA you need to add both VGA and sound device
12. Add your bootable DVD image and give it boot priority
13. Start your VM and begin the installation

*do not add a NIC yet, to avoid Windows creating an online account