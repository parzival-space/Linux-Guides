# Mesa (Nonfree)
Manjaro removed the hardware accelerated H264/H265 encoding from the Mesa drivers due to a patent issue. 
This can be a problem for users with AMD GPUs who need these codecs for tasks like streaming with OBS. 
Fortunately, it's possible to "re-enable" the codecs on Manjaro using the (Nonfree.eu repository)[https://nonfree.eu/].

## Background
Other distros have also removed these codecs from the Mesa driver. 
The reason for this is that H264/H265 encoding is patented, which means that it can't be distributed freely in open-source software. 
As a result, some distros have chosen to remove these codecs from the Mesa driver to avoid potential legal issues.

## Install Mesa-Nonfree
To install the nonfree codecs on Manjaro, you'll need to add the Mesa-Nonfree repository and then update or reinstall the Mesa package.

### Adding the Repository
Adding the repository is simple.
1. First, you'll need to trust the CI key for the Nonfree.eu repository:
   ```bash
   sudo pacman-key --recv-keys B728DB23B92CB01B
   sudo pacman-key --lsign-key B728DB23B92CB01B
   ```
2. Next, add the repository configuration:
   ```bash
   sudo sh -c "curl -s https://nonfree.eu/$(pacman-mirrors -G)/ > /etc/pacman.d/mesa-nonfree.pre.repo.conf"
   ```
3. Include the repository configuration in your pacman.conf file:
   ```bash
   sudo sed -i '/^\[core\]/i \Include = /etc/pacman.d/mesa-nonfree.pre.repo.conf\n' /etc/pacman.conf
   ```
4. Finally, update your repository list:
   ```bash
   sudo pacman -Syy
   ```

### Update or reinstall mesa
Now that the repository is set up, you can update or reinstall the Mesa package to get the nonfree codecs:
```bash
sudo pacman -S mesa
```

## Final notes
Now that you have enabled the proprietary codecs in the Mesa driver, you can enjoy hardware acceleration for H264/H265 encoding and decoding on Manjaro. 
Keep in mind that using proprietary codecs can have legal implications in some regions, so make sure to consult the laws in your area.