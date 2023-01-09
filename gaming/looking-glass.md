# Requirements

- A working gpu passthrough VM is required.
- Updated graphics drivers.
- Pulseaudio


# Installation

## Add Shared-Memory device

```xml
<shmem name='looking-glass'>
  <model type='ivshmem-plain'/>
  <size unit='M'>32</size>
</shmem>
```

<details>
    <summary>Determining memory size</summary>

    width x height x pixel size x 2 = frame bytes

    frame bytes / 1024 / 1024 = frame megabytes

    frame megabytes + 10 MiB = total megabytes
</details>

## Disable Mem-Balloon

```xml
<memballoon model="none"/>
```

## Configure Shared-Memory permissions

Replace ``{{user}}`` with your ``$UID``

```bash
sudo nano /etc/tmpfiles.d/10-looking-glass.conf
```
```
# Type Path               Mode UID  GID Age Argument

f /dev/shm/looking-glass 0660 {{user}} kvm -
```


## Configure HID

- Find your ``<video>`` device, and set ``<model type='vga'/>``
  - If you can’t find it, make sure you have a <graphics> device, save and edit again.
- Remove the ``<input type='tablet'/>`` device, if you have one.
- Create an ``<input type='mouse' bus='virtio'/>`` device, if you don’t already have one.
- Create an ``<input type='keyboard' bus='virtio'/>`` device to improve keyboard usage.


## Downgrade edk2-ovmf

Install downgrade
```
sudo pamac install downgrade
```

Downgrade edk2-ovmf to ``2022-08 3``
```bash
sudo DOWNGRADE_FROM_ALA=1 downgrade edk2-ovmf
```

Finally change you loader using XML to 
```xml
<os>
    ...
    <loader readonly="yes" type="pflash">/usr/share/edk2-ovmf/x64/OVMF_CODE.fd</loader>
    ...
</os>
```

Your VM should now be able to start.

## Install Looking-Glass in Guest

Download and install the latest Windows Guest build.
> https://looking-glass.io/downloads

## Install IddSampleDriver
The IddSampleDrivers prevents your GPU from shutdown so you dont need a dummy plug.

```powershell
# download and install scoop
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex

# install required dependencies
scoop install git sudo

# add repositories
scoop bucket add extras
scoop bucket add nonportable

# install iddsampledriver
sudo scoop install iddsampledriver-ge9-np -g
```

You can now shutdown your guest and disable the Video Display by setting your ``<video>`` type to ``None``.

### Optional: Costomize IddSampleDriver settings

You can customize the driver settings in C:\IddSampleDriver\option.txt


## Install Looking-Glass in Host OS

Download the latest source for linux.
> https://looking-glass.io/downloads

Clone the following repository and copy the extracted source into the cloned repository:
> https://github.com/pavolelsig/L-G_helper

Now just execute the script.
```bash
sudo chmode +x ./lg.sh
sudo ./lg.sh
```

# Run Looking-Glass

Make sure your VM is running.

Switch to the directory containing the build binaries:
```bash
cd ./looking-glass-B6-rc1/client/build/
```

Run the Looking-Glass client.
```bash
./looking-glass
```

## First run only

Install the SPICE guest tools
> https://www.spice-space.org/download.html#windows-binaries


Install VIRTIO drivers
> https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/
