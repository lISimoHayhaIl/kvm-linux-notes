1. Install vfio packages and reboot
    sudo pamac install qemu libvirt ovmf virt-manager ebtables dnsmasq bridge-utils linux-headers

2. Change boot kernel parameters
    sudo nano /etc/default/grub

GRUB_CMDLINE_LINUX_DEFAULT='udev.log_priority=3 pcie_acs_override=downstream,multifunction initcall_blacklist=simpledrm_platform_driver_init rd.driver.blacklist=nouveau modprobe.blacklist=nouveau video=efifb:off amd_iommu=on iommu=pt rd.driver.pre=vfio-pci kvm.ignore_msrs=1 vfio-pci.ids=10de:2704,10de:22bb,10ec:8126 hugepages=16384 nohz_full=0-7,16-23 rcu_nocbs=0-7,16-23 rcu_nocb_poll'
GRUB_DISABLE_OS_PROBER=true

3. Update Grub and reboot
    sudo update-grub

4. Check if the VGA is bound to vfio-pci instead of its driver
    lspci -nnk | grep -A 3 -E "VGA|3D"

5. Make your user as the user that runs the libvirt service. If not, it will need root password everytime you start a VM
    sudo nano /etc/libvirt/qemu.conf
    user = "yourusername"
    group = "kvm"
then add your user to that group
    sudo usermod -aG libvirt,kvm $USER

6. Add your PCI IDs that you want the system to ignore during boot in modprobe configuration. You can add any device here, not only your VGA
    sudo nano /etc/modprobe.d/vfio.conf
	options vfio-pci ids=10de:2704,10de:22bb,14e4:1677

7. Also, make sure the correct modules and hooks are loaded during boot. Might be already there, just verify existence and order
    sudo nano /etc/mkinitcpio.conf
    MODULES="vfio vfio_iommu_type1 vfio_pci irqbypass kvm kvm_amd xhci_hcd"
    HOOKS="base udev autodetect modconf block keyboard keymap filesystems"

8. Rebuild initramfs before rebooting
    sudo mkinitcpio -P