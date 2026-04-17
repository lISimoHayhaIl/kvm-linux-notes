1. Install vfio packages and reboot
    sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
    sudo dnf group install --with-optional virtualization

2. Change boot kernel parameters. 
    sudo nano /etc/default/grub
    GRUB_CMDLINE_LINUX="rhgb pcie_acs_override=downstream rd.driver.blacklist=nouveau modprobe.blacklist=nouveau video=efifb:off amd_iommu=on rd.driver.pre=vfio-pci kvm.ignore_msrs=1 vfio-pci.ids=10de:2704,10de:22bb,14e4:1677 hugepages=16384 nohz_full=0-7,16-23 rcu_nocbs=0-7,16-23 rcu_nocb_poll"
    GRUB_DISABLE_OS_PROBER=true

3. Rebuild GRUB and reboot
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg

4. Configure drakut to load the vfio drivers with prioroty
    sudo nano /etc/dracut.conf.d/local.conf
    add_drivers+=" vfio vfio_iommu_type1 vfio_pci "
and rebuild before reboot
    sudo dracut -f --kver $(uname -r)    

5. Check if the VGA is bound to vfio-pci instead of its driver
    lspci -nnk | grep -A 3 -E "VGA|3D"

6. Fedora uses Selinux for security and that will prevent to write in the log file. In order for qemu to be able to log:
    sudo nano /etc/selinux/config
    SELINUX=permissive
and reboot

7. Make your user as the user that runs the libvirt service. If not, it will need root password everytime you start a VM
    sudo nano /etc/libvirt/qemu.conf
    user = "yourusername"
    group = "kvm"
then add your user to that group
    sudo usermod -aG libvirt,kvm $USER

