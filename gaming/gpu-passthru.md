# Setting up IOMMU

## Enable IOMMU in grub config:

```bash
sudo nano /etc/default/grub
```
```
GRUB_CMDLINE_LINUX_DEFAULT="... intel_iommu=on ..."
```

## Rebuilt Grub Config:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg  
```
Reboot.


# Ensuring that the IOMMU groups are valid

## Get IOMMU Groups

Run this script:
```bash
#!/bin/bash
shopt -s nullglob
for g in $(find /sys/kernel/iommu_groups/* -maxdepth 0 -type d | sort -V); do
    echo "IOMMU Group ${g##*/}:"
    for d in $g/devices/*; do
        echo -e "\t$(lspci -nns ${d##*/})"
    done;
done;
```

backup iommu result of gpu


## Loading vfio-pci early

```bash
sudo nano /etc/mkinitcpio.conf
```
```
MODULES=(... vfio_pci vfio vfio_iommu_type1 vfio_virqfd ...)
HOOKS=(... modconf ...) # may already be set
```

## Rebuild initframes
```bash
sudo mkinitcpio -P 
```


## Isolate GPU

```bash
sudo nano /etc/default/grub
```
```
GRUB_CMDLINE_LINUX_DEFAULT="... vfio-pci.ids=10de:13c2,10de:0fbb ..."
```

Rebuild Grup again.
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg  
```
Reboot.

## Make sure everything worked
```bash
sudo lspci -nnk -d {{pci_id}}
```
```
Kernel driver in use: vfio-pci
```

# Setup Virt-Manager & QEMU

```bash
sudo pamac install qemu libvirt edk2-ovmf virt-manager

# automatically start libvirtd
sudo systemctl enable libvirtd 
sudo systemctl start libvirtd  

sudo usermod -aG libvirt ${whoami}
```

enable network
```bash
sudo virsh net-autostart default
```
