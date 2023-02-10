# Setup GPU Passthrough
This guide will cover the process of passing through a GPU to a virtual machine, allowing the VM to use the GPU as if it were a physical device. 
This can be a useful technique for users who want to use specialized software or applications that require a dedicated graphics processing unit (GPU). 
With GPU passthrough, users can achieve near-native performance in their VMs, making it a powerful tool for a wide range of use cases. 
In this documentation, I will walk through the necessary steps to set up GPU passthrough on a Linux system and provide tips and troubleshooting advice along the way.



## Requirements
Before following the steps in this guide, you should have a working QEMU virtual machine (VM) environment. 
This includes having a basic VM set up and running, as well as having QEMU and any necessary dependencies installed.
<br> If you need help with that, I wrote a guide that you can follow [here](../virtualization/basic-vm.md).

You will also need to have a dedicated GPU that you want to assign to the VM. Make sure that the GPU is properly installed and recognized by the host system.

Some knowledge of the Linux command line and system administration may also be helpful.


## Setting up IOMMU
To allow our VM to use the dedicated GPU we need to enable IOMMU.
IOMMU can be used to improve the performance and security of a system by allowing devices to directly access memory, rather than having to go through the CPU.

In short: IOMMU allows the VM to use the GPU as if it were accessing a physical device.

### Enable IOMMU in grub config:
To enable IOMMU on your system, follow these steps:

1. Open the `grub` configuration file:
    ```bash
    sudo nano /etc/default/grub
    ```

2. Depending on your system, add either the `intel_iommu=on` or `amd_iommu=on` flag to the `GRUB_CMDLINE_LINUX_DEFAULT` line:
    * For Intel systems:
        ```bash
        GRUB_CMDLINE_LINUX_DEFAULT="... intel_iommu=on ..."
        ```
    * For AMD systems:
        ```bash
        GRUB_CMDLINE_LINUX_DEFAULT="... amd_iommu=on ..."
        ```

3. Rebuild the `grub` configuration:
    ```bash
    sudo grub-mkconfig -o /boot/grub/grub.cfg
    ```

4. Reboot your system for the changes to take effect.

And that's it! IOMMU should now be enabled on your system, allowing your VM to use a dedicated GPU.



## Verifying IOMMU Groups
In order to use a dedicated GPU with a virtual machine (VM), it is necessary to ensure that the GPU is properly isolated in an IOMMU group. This section provides steps for verifying the IOMMU group assignments on your system.

### List the IOMMU Groups
To view the IOMMU groups and their associated devices on your system, run the following script:
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

The output will show the devices assigned to each IOMMU group, along with their PCI IDs. 
For example:
```
IOMMU Group 17:
        01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GA104 [GeForce RTX 3070 Ti] [10de:13c2] (rev a1)
        01:00.1 Audio device [0403]: NVIDIA Corporation GA104 High Definition Audio Controller [10de:0fbb] (rev a1)
```

Make note of the PCI IDs for the GPU and any associated devices, as these will be needed in the next step.
In this case the PCI IDs are `10de:13c2` and `10de:0fbb`.

### Isolate GPU in IOMMU Group
To ensure that the GPU is not used by the host, we need to load the `vfio-pci` driver early in the boot process. 
Follow these steps:

1. Edit the `mkinitcpio.conf` file:
    ```bash
    sudo nano /etc/mkinitcpio.conf
    ```

2. Add the `vfio_pci`, `vfio`, `vfio_iommu_type1`, and `vfio_virqfd` modules to the `MODULES` section:
    ```
    MODULES=(... vfio_pci vfio vfio_iommu_type1 vfio_virqfd ...)
    ```

3. Make sure the `modconf` hook is present in the `HOOKS` section:
    ```
    HOOKS=(... modconf ...) # may already be set
    ```

4. Rebuild the initramfs:
    ```bash
    sudo mkinitcpio -P 
    ```

5. Edit the grub configuration file:
    ```bash
    sudo nano /etc/default/grub
    ```

6. Add the PCI IDs of the GPU and associated devices to the `GRUB_CMDLINE_LINUX_DEFAULT` line:
    ```
    GRUB_CMDLINE_LINUX_DEFAULT="... vfio-pci.ids=10de:13c2,10de:0fbb ..."
    ```

7. Rebuild the `grub` configuration:
    ```bash
    sudo grub-mkconfig -o /boot/grub/grub.cfg  
    ```

8. Reboot your system.

After these steps, the GPU should be isolated in its own IOMMU group and no longer be used by the host.

### Make sure everything worked
To confirm that your GPU has been properly isolated in an IOMMU group, you can use the `lspci` command to view the kernel driver in use for the device.
```bash
sudo lspci -nnk -d 10de:13c2
```

The output should show that the kernel driver in use is `vfio-pci`:
```
Kernel driver in use: vfio-pci
```

If the `vfio-pci` driver is not in use, it may be necessary to troubleshoot the IOMMU setup process. Make sure that the steps to isolate the GPU and load the `vfio-pci` driver have been properly followed.



## Final Notes
Here are a few tips to keep in mind when setting up IOMMU and isolating a GPU for use with a virtual machine:

* Make sure that IOMMU is enabled in the BIOS/UEFI settings of your system. 
    This may not be required for the steps in this guide to work properly but I heard of cases where this was needed.

* Remember to reboot your system after making changes to the grub configuration. 
    This is necessary for the changes to take effect.