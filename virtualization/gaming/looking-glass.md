# Install Looking Glass
This guide will walk you through the process of installing Looking Glass in a virtual machine. I will cover everything from configuring Looking Glass for your virtual machine to installing Looking Glass in the guest operating system. 
By the end of this guide, you will have a fully-functioning gaming VM that can take advantage of the high-performance features of Looking-Glass. 
Let's get started!



## Requirements

- A working gpu passthrough VM is required.
  - [GPU Passthrough](./gpu-passthrough.md)
  - [Setup a basic VM](../virtualization/basic-vm.md)
- Updated graphics drivers (guest).
- Pipewire



## Installation
It is time to install the latest Looking Glass Host on our guest os.


### Add Shared-Memory device
To add a shared-memory device, add the following code to your VM's XML configuration file under the `<devices>...</devices>` section:
```xml
<shmem name='looking-glass'>
  <model type='ivshmem-plain'/>
  <size unit='M'>32</size>
</shmem>
```

#### Determining memory size
To determine the appropriate size for the shared-memory device, use the following formula.

The pixel size is 4 if your are using an SDR display and 8 if you are using an HDR capable display.
```
width x height x pixel size x 2 = frame bytes
frame bytes / 1024 / 1024 = frame megabytes
frame megabytes + 10 MiB = total megabytes
```


### Disable Mem-Balloon
Disable the mem-balloon device to prevent the VM from claiming too much memory. Modify the `<memballoon>` tag in your xml to have a `model` of `none`.
```xml
<memballoon model="none"/>
```



### Configure Shared-Memory permissions
Next, configure the shared-memory permissions. Create a new file called `10-looking-glass.conf` in the `/etc/tmpfiles.d/` directory and add the following code, replacing `{{user}}` with your `$UID`:

```bash
sudo nano /etc/tmpfiles.d/10-looking-glass.conf
```
```bash
# Type Path               Mode UID  GID Age Argument
f /dev/shm/looking-glass 0660 {{user}} kvm -
```



### Configure other devices

- Find your `<video>` device, and set `<model type='vga'/>`
  - If you can’t find it, make sure you have a `<graphics>` device, save and edit again.
- Remove the `<input type='tablet'/>` device, if you have one.
- Create an `<input type='mouse' bus='virtio'/>` device, if you don’t already have one.
- Create an `<input type='keyboard' bus='virtio'/>` device to improve keyboard usage.



### Downgrade edk2-ovmf
It may happen that your VM is no longer starting after the last change.
This is caused by a problem with the latest version of `edk2-ovmf`.
You can fix this by simply downgrading `edk2-ovmf`.

To downgrade edk2-ovmf to version 2022-08 3, first install the `downgrade` package:
```bash
sudo pamac install downgrade
```

Then run the following command. You will get a list of different versions.
Select `2022-08 3` and if you get asked to ignore updates for this packages, accept.
```bash
sudo DOWNGRADE_FROM_ALA=1 downgrade edk2-ovmf
```

Finally, change the loader in your VM's XML configuration file to:
```xml
<os>
    ...
    <loader readonly="yes" type="pflash">/usr/share/edk2-ovmf/x64/OVMF_CODE.fd</loader>
    ...
</os>
```
Your VM should now be able to start.



### Install Looking-Glass in Guest

Download and install the latest Windows Guest build.
> https://looking-glass.io/artifact/stable/host

You can install it just like every other Windows application.



### Install IddSampleDriver
> **You can skip this step if you have a Monitor or a <a href="https://a.co/d/hR8z40t">Dummy Plug</a> connected to your GPU.**
 <br>
 Also it is highly recommended to use a Dummy Plug/Monitor instead of this.

To prevent your GPU from shutting down, install the IddSampleDriver.
You can simply paste the script below in a Powershell window and you should be good to go.
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

#### Optional: Customize IddSampleDriver settings
If you would like to have less or more resolution to select, you can customize the driver settings in C:\IddSampleDriver\option.txt.



### Install Looking Glass in Host OS
Now its time to install Looking Glass on your host.

1. Download the latest source for Linux from the <a href="https://looking-glass.io/downloads">Looking Glass website</a> or directly from <a href="https://looking-glass.io/artifact/stable/source">here</a>.

2. Clone the LG Helper repository: https://github.com/pavolelsig/L-G_helper

3. Extract the Looking Glass source you just downloaded and place the `looking-glass-*` directory in the just cloned repository.

4. Run LG Helper:
   ```bash
   sudo chmode +x ./lg.sh
   sudo ./lg.sh
   ```

## Running Looking Glass
Before you begin, make sure your VM is running.

To run the Looking-Glass client, navigate to the directory containing the build binaries:
```bash
cd ./looking-glass-B6/client/build/
```

And then start Looking Glass by running the following command:
```bash
./looking-glass
```

### First run only
Once these tools and drivers are installed, you should be able to run Looking-Glass without any issues.

#### Installing SPICE Guest Tools
On the first run, you'll need to install the SPICE guest tools. You can find them at the following link:
<a href="https://www.spice-space.org/download.html#windows-binaries">https://www.spice-space.org/download.html#windows-binaries</a>

#### Installing VIRTIO Drivers
In addition, you'll need to install the VIRTIO drivers. They can be found at the following link:
<a href="https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/">https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/</a>

### (Optional) Launcher Entry
Create a new file at `~/.local/share/applications/looking-glass-client.desktop` with the following content:

```
[Desktop Entry]
Categories=Game
Comment[en_US]=VGA PCI Pass-through without an attached physical monitor, keyboard or mouse.
Comment=VGA PCI Pass-through without an attached physical monitor, keyboard or mouse.
Exec={{your_lg_dir}}/client/build/looking-glass-client
GenericName[en_US]=VGA PCI Pass-through App.
GenericName=VGA PCI Pass-through App.
Icon={{your_lg_dir}}/resources/lg-logo.svg
Name[en_US]=Looking Glass
Name=Looking Glass
StartupNotify=true
StartupWMClass=looking-glass
Terminal=false
Type=Application
X-KDE-SubstituteUID=false
```

Replace `{{your_lg_dir}}` with the path to your Looking Glass installation.

**Make sure to use the path of the `looking-glass-*` directory and not the directory where the `lg.sh` script is located.**

For example, if you installed Looking Glass in `/opt/looking-glass/looking-glass-B6`, but your `lg.sh` script is located in `/opt/looking-glass`, then `{{your_lg_dir}}` should be set to `/opt/looking-glass/looking-glass-B6`.

After creating the file, you should be able to find the Looking Glass entry in your start menu.

Note: `~/.local/share/applications/` is the default folder for user-specific applications in most Linux distributions, it may be different in other OS.



## Final Notes
Congratulations, you have successfully installed Looking Glass on your GPU passthrough VM!

If you experience any issues, check the Looking-Glass documentation and Discord for troubleshooting tips.

- Discord: <a href="https://looking-glass.io/discord">https://looking-glass.io/discord</a>
- Documentation: <a href="https://looking-glass.io/docs/">https://looking-glass.io/docs/</a>

Additionally, here are some other topics that may be of interest:

- **Streaming with Looking Glass:** Looking Glass has a OBS plugin that allows you to directly stream your Gaming VM into OBS. You read more <a href="https://looking-glass.io/docs/B6/obs/">here</a>.
- **Hiding your Gaming VM:** Some games, such as Rainbow Six Siege, prevent the usage of VMs. I may write a guide on how to prevent these games from detecting that you are using a VM.