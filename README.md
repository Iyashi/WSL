# WSL
This is a step-by-step guide explaining how to setup your WSL workspace.

## Requirements
You need to enable virtualization in your BIOS as well as in Windows.

You also need 5-10GB available on `C:` drive.  
Most of it can be recovered later by moving the WSL distro to another drive, but the initial setup will be done on `C:`.

### Enable Virtualization in BIOS
Boot into your BIOS and check that your proprietary virtualization technologies are enabled. (e.g. VT-d, VT-x)

### Enable Windows Features
Enable the *Hyper-V* and *WSL* Windows features.  
**(Reboot required!)**

- Windows-Subsystem for Linux
- Hyper-V
- Windows PowerShell 2.0 (not required, but highly recommended)

Although not required, I highly recommend to enable *PowerShell* aswell so we can work more efficiently on Windows side.  
PowerShell comes with a powerful terminal which is much more modern and easier to handle compared to the old `cmd`.

### Setup WSL engine
I call it "WSL engine" for a lack of a better term.

Basically, we have one **WSL engine** which runs various **WSL distro** (like our workspace).  
On the surface, it is similar to docker. We have one **docker engine** which runs various **docker containers**.  
Although it probably works completely different under the hood, but this analogy may help with understanding the concept.

First we need to setup the **WSL engine**, then we can create our workspace **WSL distro**.

### Test WSL engine
We can test if the WSL engine is working correctly by opening up the terminal and running:
```ps1
wsl --status
```
It should print the WSL default distro (`Ubuntu`) and default version (`2`).

### Update WSL engine
Ensure your WSL engine is on the latest version:
```ps1
wsl --update
```
It is recommended to update the WSL engine regulary. It provides bug fixes and compatibility and performance improvements.

### Update WSL default version
Your WSL default version should be `2` and if it's not, set it like so:
```ps1
wsl --set-default-version 2
```

### Setup directories
I recommend to create a `wsl/` directory directly on an SSD. (e.g. `W:\wsl\`)

The reason is that by default WSL installs everything somewhere in your `%AppData%` which bloats your `C:` drive.

Everything regarding your WSL distro/images/backups should be placed there:
```ps1
W:
└── wsl
    ├── disks
    │   ├── DockerLib.vhdx  # /var/lib/docker (docker cache)
    │   └── Workspace.vhdx  # ~/workspace (your codes/projects)
    ├── distros
    │   ├── Workspace       # / (rootfs for Workspace)
    │   │   └── ext4.vhdx
    │   └── OtherWorkspace  # / (rootfs for OtherWorkspace)
    │       └── ext4.vhdx
    ├── images
    │   └── Workspace.tar.gz
    ├── workspace.ps1       # starter script for distro
    └── other_workspace.ps1 # starter script for distro
```

Lets set it up in PowerShell (or create it manually in explorer):
```ps1
W: # choose your drive
mkdir wsl/disks
mkdir wsl/distros
mkdir wsl/images
```

## Create a base distro
With all requirements fulfilled, WSL should now be ready and we can install our first WSL distribution.
```ps1
wsl --install Ubuntu-24.04
```
For more options, refer to [Cheatsheet: Install a WSL distro](./README.md#install-a-wsl-distro)

This will 
1. download `Ubuntu-24.04.tar` from Microsoft
1. installs it as `Ubuntu-24.04` WSL distro
1. runs the `Ubuntu-24.04` WSL distro which
1. asks you to initially setup your Ubuntu user

If everything worked correctly, you should see a bash prompt inside your WSL distro and are ready to go.

However, by default WSL downloads and installs everything somewhere in your `%AppData%` which will bloat your `C:` drive.

## Create a Workspace distro from base distro
Lets create our `Workspace` from `Ubuntu-24.04` base and store it in the previously created directory during [Requirements: Setup directories](./README.md#setup-directories).  

```ps1
wsl --export Ubuntu-24.04 W:\wsl\images\Ubuntu-24.04.tar.gz
wsl --unregister Ubuntu-24.04
wsl --import Workspace W:\wsl\distros\Workspace W:\wsl\images\Ubuntu-24.04.tar.gz
```

Next, lets tell WSL to use `Workspace` by default if not otherwise specified with `-d OtherDistro`:
```ps1
wsl --set-default Workspace
```

If everything worked correctly, you're ready to go for real.

## Setup Workspace distro
Now we can finally setup our `Workspace` WSL distro to our liking.

First of all, of course, we need to log into the distro:
```ps1
wsl -d Workspace
```

### Create a user
Upon the first launch of a freshly installed distro, you'll be asked to create the initial sudo user.

I will use `dev` (short for developer)

### Update system
Once inside our distro, first of all, we update the system and packages to the latest version:
```bash
sudo apt-get update
sudo apt-get upgrade
```

### Install basic utilities
Next we're going to install some basic and commonly used tools:
```bash
sudo apt-get install git nano vim curl wget rsync tree zip
```

### Change WSL config
Next we're going to configure how WSL engine interacts with the WSL distro. ([`wsl.conf` documentation](https://learn.microsoft.com/en-us/windows/wsl/wsl-config#wslconf))
```bash
sudo nano /etc/wsl.conf
```

#### Customize distro hostname in `/etc/wsl.conf`
By default, the distro will use the Windows hostname, which will be something like `DESKTOP-Q2K437Q`.

Because linux shells usually have a prompt like `user@hostname:/path/to/cwd$`
```bash
dev@DESKTOP-Q2K437Q:/path/to/cwd$
dev@DESKTOP-Q2K437Q:~$ # dev users home dir
```
Which not only looks ugly, but also makes it hard to differentiate between multiple WSL distros.

So, we're going to use a meaningful name like `workspace`.
```ini
# /etc/wsl.conf
[network]
hostname=workspace
```
Now `exit` back to PowerShell and restart the distro using `wsl -t Workspace` followed by `wsl -d Workspace` and you should be greeted with a nice looking
```bash
dev@workspace:/path/to/cwd$
dev@workspace:~$ # dev users home dir
```

#### Customize default login user in `/etc/wsl.conf`
By default, WSL engine will log into the distro using `root` user.

You can also pass `-u myuser` to `wsl` command, but we don't want to do that everytime.

So, we're going to change the WSL distro default login user to `dev`. (or whatever you used)
```ini
# /etc/wsl.conf
[user]
default=dev
```
Now `exit` back to PowerShell and restart the distro using `wsl -t Workspace` followed by `wsl -d Workspace` and you should be logged in as `dev`.

### Install ZSH
Instead of using bash, I recommend using zsh with [oh-my-zsh](https://ohmyz.sh).

Install zsh and oh-my-zsh:
```bash
sudo apt-get install zsh fzf
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Change oh-my-zsh theme to `agnoster` (or choose your own theme):
```bash
sed -i 's/ZSH_THEME=".*"/ZSH_THEME="agnoster"/' ~/.zshrc
```

Install oh-my-zsh plugins:
```bash
ZSH_PLUGINS=(wd cp fzf) # activate default plugins shipped with oh-my-zsh
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
ZSH_PLUGINS=($ZSH_PLUGINS zsh-autosuggestions)
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
ZSH_PLUGINS=($ZSH_PLUGINS zsh-syntax-highlighting)
sed -i "s/plugins=(\(.*\))/plugins=(\1 $ZSH_PLUGINS)/" ~/.zshrc
source ~/.zshrc # reload shell
```

### Install Golang
If you're developing with Golang, you should install it.

Developing 

Install Golang:
```bash
GO_VERSION=1.23.2
wget https://go.dev/dl/go${GO_VERSION}.linux-$(dpkg --print-architecture).tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go${GO_VERSION}.linux-$(dpkg --print-architecture).tar.gz
cat >> ~/.zshrc << 'EOF'

# Golang
if [ -d /usr/local/go ] ; then
    export GOROOT=/usr/local/go
    export GOPATH=$HOME/go
    export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
fi
EOF
source ~/.zshrc # reload shell
```
Test Golang installation:
```bash
go version
```

### Install Nodejs / NVM (Node Version Manager)
If you're developing with Nodejs, you should install it.

I'm going to use NVM for that, so we can easily switch between Nodejs versions.

Install NVM:
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
cat >> ~/.zshrc << 'EOF'

# NVM (Node Version Manager)
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
EOF
source ~/.zshrc # reload shell
```
Test NVM installation:
```bash
nvm version # should tell you that you dont have any node versions installed yet
```
Install node versions through NVM:
```bash
nvm ls-remote # list node versions available for install
nvm install 20 # first one will be used as default
nvm install 18
nvm install 16
nvm install 22
nvm ls # list installed node versions to verify
```

### Install Docker
If you're working with docker, you should install it.

Add docker apt repository:
```bash
sudo apt-get install ca-certificates
sudo install -m 0755 -d /etc/apt/keyrings
. /etc/os-release # get ubuntu $ID and $VERSION_CODENAME
sudo curl -fsSL https://download.docker.com/linux/$ID/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/$ID \
  $(echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
Install docker from apt:
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
In order to not use `sudo` infront of every docker command, your ubuntu user needs to be in the `docker` group:
```bash
sudo usermod -aG docker $USER
# restart the shell (exit & reopen, source ~/.zshrc is not enough) in order to
```
Test docker installation:
```bash
docker run hello-world
```

### Install NVIDIA Container Toolkit (for Docker GPU integration)
If you need GPU acceleration for docker containers, you need the NVIDIA Container Toolkit. (`libnvidia-container`)

Add NVIDIA Container Toolkit apt repository:
```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \\
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \\
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \\
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
```
Install NVIDIA Container Toolkit from apt:
```bash
sudo apt-get install -y nvidia-container-toolkit
```
**Reboot required!** `wsl --terminate Workspace`

Test NVIDIA Container Toolkit:
```bash
docker run --rm -it --gpus=all nvcr.io/nvidia/k8s/cuda-sample:nbody nbody -gpu -benchmark
```
> _**Note**: I only have an NVIDIA GPU. For AMD, Intel and others, look for a solution yourself._

## Create an image of the freshly setup `Workspace`
Now that we have all the basics installed and configured, we should back it up so we can restore it when needed.

```ps1
wsl --export Workspace W:\wsl\images\Workspace.tar.gz
```

From now on you can always reset to this exact point by executing:
```ps1
wsl --unregister Workspace # delete current workspace (DISK DELETED!!!)
wsl --import Workspace W:\wsl\distros\Workspace W:\wsl\images\Workspace.tar.gz
```

## Further tips and tricks

### Use specific virtual disks for "BigData"
I recommend to create a `.vhdx` for all kinds of "bigdata".

For example, `/var/lib/docker` can grow to several dozens if not hundreds of Gigabytes.  
The images you pull and containers you build, the bigger it will grow and by default it is stored on the root disk.

Another example would be AI/LLM models for ollama and/or comfyui and stable diffusion.

If the root disk increases, so do its backups/images.

Lets create a `DockerLib.vhdx` and mount it:
```ps1
# create virtual disk
New-VHD -Path W:\wsl\disks\DockerLib.vhdx -Size 100GB -Dynamic
# mount virtual disk (bare, because it's still unformatted and without a filesystem)
wsl --mount --bare --vhd W:\wsl\disks\DockerLib.vhdx
```

Now log into the workspace to format the virtual disk using `ext4`:
```bash
lsblk # find device (like /dev/sdX)
sudo mkfs.ext4 /dev/sdX # format to ext4
```

Now that it's a properly formatted disk, we can remount it with a name for automount.
```ps1
wsl --unmount W:\wsl\disks\DockerLib.vhdx # unmount the previous --bare mount
wsl --mount --vhd W:\wsl\disks\DockerLib.vhdx --name dockerlib # mount with name
# will automount at /mnt/wsl/dockerlib
```

Now that it's formatted and mounted at `/mnt/wsl/dockerlib`, we can replace `/var/lib/docker`:
```bash
systemctl stop docker # stop docker
mv /var/lib/docker/* /mnt/wsl/dockerlib/ # move docker lib
rm -rf /var/lib/docker # remove docker lib dir
ln -s /mnt/wsl/dockerlib /var/lib/docker # replace it with symlink to mountpoint
systemctl start docker # start docker again
```

## Cheatsheet
Here is a list of usefull stuff regarding WSL which you'll probably need over time.

### Install a WSL distro
You can get a list of distros available for download:
```ps1
wsl --list --online
# NAME                            FRIENDLY NAME
# Ubuntu                          Ubuntu
# Debian                          Debian GNU/Linux
# Ubuntu-18.04                    Ubuntu 18.04 LTS
# Ubuntu-20.04                    Ubuntu 20.04 LTS
# Ubuntu-22.04                    Ubuntu 22.04 LTS
# Ubuntu-24.04                    Ubuntu 24.04 LTS
# [...]
```

Choose a distro, then install it using:
```ps1
wsl --install <NAME> --no-launch
```
This is already a fully working WSL distro which you can log into:
```ps1
wsl -d <NAME>
```

> _**Note**: The default distribution is `Ubuntu`. If you leave out `<NAME>`, `Ubuntu` is implied._
> ```ps1
> wsl --install # defaults to Ubuntu
> ```

> _**Note**: There are a lot more distributions available by the community in the Windows store. (usually free of charge)_

Without `--no-launch`, the distro will be launched after install and you'll be asked to create the initial user.

### Create an image of a WSL distro
You can create an image of a WSL distro.

This is very usefull.

You can use that image as a backup and restore it when something breaks.

You can also use that image as "installer" for new WSL distros. It comes will all installed software and configured, so after importing, you can just continue working.

Create image in PowerShell:
```ps1
wsl --export Workspace W:\wsl\images\Workspace.tar.gz
```

Create new distro from image in PowerShell:
```ps1
wsl --import OtherWorkspace W:\wsl\distros\Workspace W:\wsl\images\Workspace.tar.gz
wsl -d OtherWorkspace # log into new distro
```

### Move a WSL distro to another location (e.g. other drive)
You can move WSL distro to another location.

It works by 
1. create an image
1. deleting the current distro
1. create new distro from image at another location
1. done

Create image in PowerShell:
```ps1
wsl --export Workspace W:\wsl\images\Workspace.tar.gz
wsl --unregister Workspace
wsl --import Workspace X:\other\location\Workspace W:\wsl\images\Workspace.tar.gz
wsl -d Workspace # log into moved distro
```

### Create another virtual disk
You can create new virtual disk files (`.vhdx`).

Dynamic size (starts at ~1GB, grows over time until given size is reached)
```ps1
New-VHD -Path W:\wsl\disks\Extra.vhdx -Size 100GB -Dynamic
```

Fixed size (exactly 100GB in size)
```ps1
New-VHD -Path W:\wsl\disks\Extra.vhdx -Size 100GB -Fixed
```

### Optimize / Shrink a virtual disk (vhdx)
Over time, a dynamic size virtual disk file (`.vhdx`) can allocate quite a lot of space without freeing it again.  

This happens, for example, when you download a big file (100gb) and then delete it. The space will be freed, but it is still allocated by the virtual disk.

However, you can optimize/shrink a virtual disk to the size it actually needs. (It can grow again after that.)

Ensure disk is not in use. Either
- `wsl --unmount W:\wsl\disks\Extra.vhdx` directly
- `wsl --shutdown` all running wsl distros

In the PowerShell:
```ps1
Optimize-VHD -Path W:\wsl\disks\Extra.vhdx -Mode Full
Optimize-VHD -Path W:\wsl\distros\Workspace\ext4.vhdx -Mode Full
```

### Mount another disk
You can mount disk files to your _WSL engine_, which makes them available for _all WSL distros_.

The base command is `wsl --mount` and you can view its docs with `wsl --help` to get all available options.

Once it's available in the engine, you can see it in the distro using `lsblk`.  
`lsblk` will also indicate wether it is already mounted or not.

The default automount location is in `/mnt/wsl/<name>`. The `<name>` is supplied via `wsl --mount <..> --name <name>`.

You can mount a disk:
```ps1
wsl --mount <PathToDisk> --name extra # automounts /mnt/wsl/extra
```

You can disable the automount by passing `--bare` to it:
```ps1
wsl --mount <PathToDisk> --name extra --bare # no automount
# manually mount in wsl distro:
lsblk # get device path /dev/sdX
sudo mkdir /mnt/extra
sudo mount -t ext4 /dev/sdX /mnt/extra
```

#### Mount a virtual disk (`*.vhdx`)
It is just another disk, but you need to pass `--vhd` and the path to the `.vhdx`.

If the vhdx is partitioned, you also need to specify the partition index using `--partition <idx>`.

Here are some examples:
```ps1
wsl --mount --vhd W:\wsl\disks\extra.vhdx --name extra                # automounts /mnt/wsl/extra
wsl --mount --vhd W:\wsl\disks\parted.vhdx --name part1 --partition 1 # automounts /mnt/wsl/part1
wsl --mount --vhd W:\wsl\disks\parted.vhdx --name part2 --partition 2 # automounts /mnt/wsl/part2
```

#### Mount a physical drive
It is just another disk, but you need to pass the **DeviceID**.

If the drive is partitioned, you also need to specify the partition index using `--partition <idx>`.

Here are some examples:
```ps1
wsl --mount \\.\PHYSICALDRIVE0 --name extra               # automounts /mnt/wsl/extra
wsl --mount \\.\PHYSICALDRIVE2 --name part2 --partition 2 # automounts /mnt/wsl/part2
wsl --mount \\.\PHYSICALDRIVE2 --name part3 --partition 3 # automounts /mnt/wsl/part3
```

You can find out the **DeviceID** by running the following in PowerShell:
```ps1
GET-CimInstance -query "SELECT * from Win32_DiskDrive"
# DeviceID           Caption                   Partitions Size          Model
# --------           -------                   ---------- ----          -----
# \\.\PHYSICALDRIVE2 Samsung SSD 840 EVO 120GB 4          120031511040  Samsung SSD 840 EVO 120GB
# \\.\PHYSICALDRIVE3 Samsung SSD 850 EVO 120GB 1          120031511040  Samsung SSD 850 EVO 120GB
# \\.\PHYSICALDRIVE0 ST2000DM001-1CH164        1          2000396321280 ST2000DM001-1CH164
# \\.\PHYSICALDRIVE1 Samsung SSD 840 EVO 120GB 1          120031511040  Samsung SSD 840 EVO 120GB
```
