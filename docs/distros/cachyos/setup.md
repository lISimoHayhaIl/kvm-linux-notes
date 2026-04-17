1. Install vfio packages and reboot
    sudo pacman -S qemu-full virt-manager swtpm
    == echo 'firewall_backend = "iptables"' | sudo tee -a /etc/libvirt/network.conf
    systemctl enable --now libvirtd.service
    systemctl enable --now libvirtd.socket
    == sudo virsh net-autostart default
    == sudo ufw route allow from 192.168.122.0/24
    * works without == lines, but I am using LAN device with pass-through

2. Change boot kernel parameters. Sometimes limine.conf might be in another folder, search for it
    sudo nano /etc/default/limine
KERNEL_CMDLINE[default]+="quiet nowatchdog splash rw rootflags=subvol=/@ root=UUID=b33b1f0a-d49f-4eef-87f8-25f774aa65c5 nvidia_drm.modeset=1 rd.driver.blacklist=nouveau modprobe.blacklist=nouveau video=efifb:off rd.driver.pre=vfio-pci kvm.ignore_msrs=1 vfio-pci.ids=10de:2704,10de:22bb,14e4:1677 amd_iommu=force_enable iommu=pt hugepages=16384"

3. Reboot
    CachyOS with Limine, is not using GRUB, and reads the config on the fly, so no need to rebuild before reboot

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