# Setting up WSL 2 with VS Code In Windows



## Pre-requisites for Running WSL 2

- For x64 systems: Windows 10 Version 1903 or higher, with build 18362 or higher
- For ARM64 systems: Windows 10 Version 2004 or higher, with build 19041 or higher



## Step 1 - Ensure a WSL in Place

if not, launch the PowerShell as Administrator and run -

```shell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```



## Step 2 - Enable Virtual Machine Platform

Run PowerShell as Admin:

```shell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```



## Step 3 - Restart the machine

```shell
shutdown -r -f
```



## Step 4 - Download the Linux kernel update package

Download the Linux kernel update package for x64 machines:

:--: [WSL2 Linux kernel update package for x64 machines](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

Install it by double-clicking the installer.

then update the `WSL`:

```shell
wsl.exe --update
```

Then Set the default version to 2 in PowerShell window:

```shell
wsl --set-default-version 2
```



## Step 5 - Install the WSL Distribute you like

Firstly take a look at what Microsoft can offer:

```shell
wsl -l -o
```

Then install a favorable Linux distro, say "Ubuntu":

```shell
wsl --install Ubuntu
```



## Step 6 - Install "Remote WSL" extension on the Host

Easy - Launch VS Code in Windows, and Install "Remote WSL" extension made by Microsoft.



## Step 7 - Prepare and Launch VS Code from WSL Linux

Installing `buildtools` in WSL Linux is very simple:

```shell
sudo apt-get update
sudo apt-get install build-essential gdb

whereis g++
whereis gcc
whereis gdb
```

then link Windows projects folder to WSL Linux:

```shell
ln -s /mnt/e/projects/ projects
```

Now that, you can launch the VS Code from WSL Ubuntu, it will install "VS Code Server for Linux" the first time (once-a-time thing):

```shell
## launch the VS Code from Ubuntu
code .
## The outout belike:
## ------------------
## Installing VS Code Server for Linux x64 (848b80aeb52026648a8ff9f7c45a9b0a80641e2e)
## Downloading: 100%
## Unpacking: 100%
## Unpacked 2042 files and folders to /home/zenusr/.vscode-server/bin/848b80aeb52026648a8ff9f7c45a9b0a80641e2e.
## Looking for compatibility check script at /home/zenusr/.vscode-server/bin/848b80aeb52026648a8ff9f7c45a9b0a80641e2e/bin/helpers/check-requirements.sh
## Running compatibility check script
## Compatibility check successful (0)
```



## Step 8 - Access Files between Linux and Windows

Ensure the WSL Linux Distro is running, then launch "File Explorer" from Windows and type at the address bar to dig into the Linux folders and files:

```shell
\\wsl$
```

From WSL Ubuntu, it's easy to explore since all of the files of Windows are automatically mounted at:

```shell
//mnt/
```



## The End