# Switching to the Manjaro Unstable Branch
If you're an advanced user who needs to use packages from the Arch User Repository (AUR), you may need to switch to the Manjaro unstable branch to ensure compatibility with those packages. 
In this guide, I'll show you how to switch to the unstable branch and update your system to use the latest packages.

## Important
Switching to the unstable branch can be risky and may cause instability on your system. 
Although Manjaro states that "using the unstable branch is usually safe but - in rare cases - may cause issues with your system!" and 
"Those that use Unstable need to have the skills to get themselves out of trouble when they move their system to this branch." 
Therefore, it's important to keep this in mind and proceed with caution.

## Get your current running Branch
First, let's check which branch your system is currently running on using the following command:
```bash
pacman-mirrors -G
```
This command will display your current branch in the output.

## Switching to the Unstable Branch
To switch to the unstable branch, you need to change your pacman-mirrors configuration. 
Replace 'unstable' with one of the following branches to switch to them instead: stable, testing, or unstable.
```bash
sudo pacman-mirrors --api --set-branch unstable
```

### Rebuild Mirrorlist
After changing the branch, rebuild the mirrorlist and update your packages:
```bash
sudo pacman-mirrors --fasttrack 5
sudo pacman -Syyu
```

## Final notes
Thats it. Welcome to the unstable branch.
Be aware that packages from the AUR are not officially supported by Manjaro, so use them at your own risk and be prepared to troubleshoot any issues that may arise.