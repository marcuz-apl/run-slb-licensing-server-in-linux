# SLB Licensing Server in Linux

by Marcus Zou | Initialized: 21 April 2025 | Updated: 19 December 2025

[TOC]

## Intro

This is an alternative approach to run SLB Licensing Tools inside a Linux box, saving some efforts of running up and maintaining the SLB Licensing Tools on a specific Windows box or Virtual machine.

**Pre-requisites**:

- A Linux box, either a physical box or virtual machine, a WSL in Windows is preferred;

  - Linux edition: RedHat/Rocky Linux is preferred, but Ubuntu/Debian Server also works very well;

- "**SLB_Licensing_2025.1_linux.tar.gz**" or likewise from Schlumberger website or a trustworthy source;

- A **lsb** package installed in Linux:
  ```shell
  # Debian/Ubuntu Linux
  sudo apt install lsb-base
  # RedHat/Rocky/Oracle Linux
  yum install lsb
  ## yum install redhat-lsb
  ```

- Some knowledge of Linux system.



## A) Install the SLB License Server

**1- Install CodeMeter Dongle Driver**

You may need to download appropriate CodeMeter Runtime rpm package (driver of CodeMeter Dongle) from [WIBU system website](https://www.wibu.com/support/user/user-software.html) and install it:

```shell
# Optionally install some packages
apt install net-tools iptables

## For Redhat/Oracle/Rocky Linux
rpm -ich CodeMeter-8.40.7154-505.x86_64
## For Debian/Ubuntu Linux
apt install codemeter_8.40.7154.505_amd64.deb
```

> [!IMPORTANT]
>
> Once installed, reboot the server to complete CodeMeter installation.

**2- Download the SLB Licensing package**

Go to https://www.sdc.software.slb.com/, click **SIS**, and log in. In the left pane, click **More Supported Products**, click **Utilities**, and then click **Flexlm**.

Then, extract the package using tar command in Linux

```shell
mkdir -p /opt/slb-licensing/
tar xzvf /mnt/e/folder/to/SLB_Licensing_2025.1_linux.tar.gz -C /opt/slb-licesning/
```

The "**-C**" argument specifies the folder holding the licensing app. You may need administrator privilege to create such folder.

**3- Edit the license file for Linux case**

If you have a license file bearing the **slbfd** or **lmgrd.slb** as "**VENDOR**", you must change the ***VENDOR
slbfd*** or ***VENDOR lmgrd.slb*** line to read ***VENDOR slbsls***. When altered, the file should resemble the following. The second line of the file has changed:

```text
SERVER this_host 0123456789
VENDOR slbsls
USE_SERVER
INCREMENT gasfield slbfd 2022.0 1-jun-2024 1 SUPERSEDE=gasfield \
NOTICE="0738198 A2GF-P1" START=12-jun-2024 AUTH={ slbfd=( \
SIGN="003E 9B74 A1DC 645B D177 B400 A079 E400 1D40 09BC 2C27
9800 B0AE DA4B FC48") lmgrd.slb=( LK=7689E5620621 SIGN="008D\
3799 4265 25A4 25C5 DA12 534D A800 4331 A349 9740 7B86 36BF \
A613 FBF6" SIGN2="00F9 B765 0F28 3BEE 6179 6718 22C7 0D00 B1E0 \
7030 1CEB B59E D3CB D5C8 E569") }
```

**4- Copy the altered license file to the specific location**

Simply run:

```shell
# copy the Petrel-2024-license.lic file over to the default license folder and rename it
cp /mnt/e/folder/to/Petrel-2024-license.lic /opt/slb-licensing/license.dat
```

**5- Spin up the licensing service: license daemon (lmgrd) + vendor server (slbsls)**

We are gonna use the license daemon`lmgrd` to launch the server instance `slnsls` with the server name being `slbsls`. Simply run the commands:

```shell
# Enter into the folder where the license daemon resides
cd /opt/slb-licensing
# Spin up the license service by
./lmgrd -c ./license.dat -2 -p -l +/var/log/flex/flex.log
```

If you check the log file by running `tail /var/log/flex/flex.log`, you should see the following messages:

```text
 9:15:39 (slbsls) (@slbsls-SLOG@) Running on Hypervisor: Unknown Hypervisor
 9:15:39 (slbsls) (@slbsls-SLOG@) ===============================================
 9:15:39 (slbsls) Ecomms: Encrypted Communication disabled
 9:15:40 (slbsls) TCP_NODELAY NOT enabled
 9:15:40 (slbsls) Listener Thread: running
 9:15:40 (slbsls) Starting diagnostics port listener thread (DPLT)
 9:15:40 (slbsls) Starting diagnostics output thread (DRQT)
 9:15:40 (slbsls) DPLT: running
 9:15:40 (slbsls) DRQT: running
 9:15:40 (slbsls) DPLT: waiting for logger to connect
```

The last line means the licensing server is up running and ready for calls from clients: Petrel, Techlog, etc.

**Note: Key usage of FlexNet (FLEXlm) Network Licensing: lmgrd**

`lmgrd` is the main daemon for FlexNet (FLEXlm) network licensing, responsible for managing software license checkout and starting vendor daemons. It is invoked via command line to read a license file (`-c`) and generate a debug log (`-l`). The standard command is `lmgrd -c <path_to_license> -l <path_to_log>`.

Key Usage and Commands

- **Start Server:** `lmgrd -c license.lic -l debug.log`.
- **Alternative Port:** By default, `lmgrd` runs on port 27000.
- **Troubleshooting:** Use `lmstat` to check server status and `lmdown` to shut down the server gracefully.
- **Best Practices:** Always use the latest version of `lmgrd` available from [Revenera/Macrovision](https://docs.revenera.com/fnp/2022r3/LicAdmin_Guide/Content/helplibrary/lmgrd_Command_Line_Syntax.htm).
- **Log Location:** Avoid saving the `lmgrd.log` to a network path to prevent startup failures.

Common Command-Line Options:

| Option   | Description                                                  |
| :------- | :----------------------------------------------------------- |
| `-c`     | Specifies the license file or directory path.                |
| `-l`     | Specifies the debug log file path.                           |
| `-local` | Runs license management on the local machine only.           |
| `-z`     | Runs in foreground. The default behavior is to run in the background. If `-l debug_log_path` is present, then no windows are used, but if no `-l` argument specified, separate windows are used for `lmgrd` and each vendor daemon. |
| `-v`     | Displays version information.                                |
| -2 -p    | All platforms: -2 -p is effective, and supported, only when used together with ‑local. <br />On UNIX systems, -2 -p restricts usage of lmdown, lmreread, and lmremove—as well as lmswitch, lmswitchr, and lmnewlog—to a license administrator who is by default root. If there is a UNIX group called lmadmin, then use is restricted to only members of that group. If root is not a member of this group, then root does not have permission to use any of the above utilities<br />On Windows systems, if lmgrd is started with -2 -p -local, lmgrd and the vendor daemon can only interact with the command-line utilities (lmreread, lmnewlog, lmdown, lmremove, and lmswitch) if these are located on the same machine and if they are run with LOCALSYSTEM privileges. |

**Windows Service Management**

On Windows, `lmgrd` is generally installed and managed as a service using `lmtools.exe`. You can configure it to start automatically at boot.

**6- Launch your Petrel 2024 to connect**

The license server shall be in the format of `<server-port>@<LicComputerIPAddress>;`, such as:

```text
27000@172.23.96.76;
```

I have put the license server into a WSL Ubuntu 24.04, then the IP address is the one from WSL Ubuntu Environment, knowing such info by running command: `ip a`.

**7- Conduct port-forwarding as needed** 

You can forward that port from WSL Ubuntu/Rocky to Windows Host by running PowerShell commands.

Open **PowerShell as Administrator** and run the following command, replacing `<Win_Port>`, `<WSL_Port>`, and `<WSL_IP>`:

```shell
netsh interface portproxy add v4tov4 listenport=<Win_Port> listenaddress=0.0.0.0 connectport=<WSL_Port> connectaddress=<WSL_IP>
```

*Example for forwarding WSL port 27000 to Windows port 27000:*

```shell
netsh interface portproxy add v4tov4 listenport=27000 listenaddress=0.0.0.0 connectport=27000 connectaddress=172.23.96.76

## Show the result
netsh interface portproxy show v4tov4
```

**8- Allow Traffic Through the Windows Firewall**

If you want this port to be accessible from other devices on your LAN (Local Area Network), add a rule to the Windows Firewall in **PowerShell (Run as Administrator)**:

```PowerShell
New-NetFirewallRule -DisplayName "WSL Port Forward" -Direction Inbound -Protocol TCP -LocalPort <Win_Port> -Action Allow
```



## B) Create a New Startup Script: the Modern way

> [!NOTE]
>
> The following example applies to `systemd`-based Linux distributions (Ubuntu and Rocky Linux).

**1- Create a new group and user: `flexlm`.**

```shell
groupadd flexlm
useradd flexlm -c "flexlm user" -g flexlm -s /sbin/nologin
```

**2- Create a directory `/var/log/flex`, and assign ownership to the new user `flexlm` by:**

```shell
mkdir -p /var/log/flex
chown flexlm: /var/log/flex
```

**3- Give ownership of installed files `lmgrd` and `slbsls` to the `flexlm` user.**

```shell
cd /opt/slb-licensing
chown flexlm:flexlm lmgrd slbsls
```

**4- Give give the owner read and execute permissions.**

```shell
chmod 544 lmgrd slbsls
```

**5- Create a new file called slbsls.service in /etc/systemd/system**

```shell
nano /etc/systemd/system/slbsls.service
```

**6- Add the following lines to the file:**

```text
[Unit]
Description=Licence manger for SLBSLS
After=network.target
[Service]
Type=simple
User=flexlm
WorkingDirectory=<inst_dir>
ExecStart=<inst_dir>lmgrd -z -c <your_license_file> -2 -p -l +/var/log/flex/flex.log
SuccessExitStatus=15
Restart=always
RestartSec=30
[Install]
WantedBy=multi-user.target
```

<inst_dir> is the installation path and <your_license_file> is the path to the license file.

**7- Handle the log file**

```shell
rm -rf /var/log/flex/flex.log
touch /var/log/flex/flex.log
choown flexlm:flexlm /var/log/flex/flex.log
```

**8- Configure the service to run automatically by**

```shell
systemctl enable --now slbsls
systemctl start slbsls
```

> [!NOTE]
>
> The `-2 -p` options on the command line for starting the license server prevent other users from initiating a shut down or reread.

> [!IMPORTANT]
>
> Reboot the Linux box to have a test out. 



### B2) Create a New Startup Script: The Legacy way



**1- Create a new file called `slbsls-flexstart` in the folder of `/etc/init.d/` **

```shell
nano /etc/init.d/slbsls-flexstart
```

**2- Add the following lines to the file**

```text
#!/bin/sh
/opt/slb-licensing/lmgrd -c /opt/slb-licensing/license.dat -2 -p -l +/var/log/flex/flex.log
```

Replace the path: `/opt/slb-licensing/` with yours.

**3- Make the script executable using the following command:**

```shell
chmod 755 /etc/init.d/slbsls-flexstart
```

**4- Create links in both the `/etc/rc3.d` and `/etc/rc5.d` folders so the script is run at system startup.
Change to the appropriate directory (for example /etc/rc3.d) and create a link.**

```shell
cd /etc/rc3.d
ln -s /etc/init.d/slbsls-flexstart S99slbsls-flexstart

cd /etc/rc5.d
ln -s /etc/init.d/slbsls-flexstart S99slbsls-flexstart
```

> [!IMPORTANT]
>
> Reboot the Linux box to have a test out.



## C) Stop an Existing License Server

> [!NOTE]
>
> If you start from scratch or you have never install SLB Licensing server on the subject Linux box, you can ignore this step.

**1- Run the command:**

```shell
ps -ef | grep lmgrd
```

The command shows if any **lmgrd** processes are running. If you have a running license server, the
output will be something similar to:

```text
root        1726     458  0 Apr29 pts/0    00:00:03 ./lmgrd -C ./license.dat -2 -p -l +/tmp/flex.log
root        1728    1726  1 Apr29 ?        00:15:11 slbsls -T EXP-1678 11.19 9 -c :./license.dat: -p -lmgrd_port 6978 -srv 5k2OWA2a4U6kiS6MI08FFExA3nbOlDoaiyEJkRV721Gqr6V72BhS5uCOHVbvtT6 --lmgrd_start 69f1c59f -vdrestart 0 -l /tmp/flex.log
root        4263     459  0 08:18 pts/0    00:00:00 grep --color=auto lmgrd
```

The server name may be one of **lmgrd.slb**, **slbfd**, or **slbsls**.

**2- Kill the service:**

If there is only one **lmgrd**, stop it using the following command:

```shell
pkill lmgrd
## or kill the process-ID, "1722"
kill -9 1726
```

**3- Check that the lmgrd process is missing from the output using the command:**

```shell
ps aux | grep init | grep -v grep lmgrd
```

**4- Check if the license is automatically started when the machine boots:**

There are several ways this can be configured.

* The systemd service tables in **/etc/systemd/system** file may contain an entry to start the license
  server. Search **/etc/systemd/system** for **lmgrd** or **slbsls**, remove the file (or copy to /tmp) and run:

  ```shell
  systemctl daemon-reload
  systemctl reset-failed
  ```

* The **/etc/rc.local** file may contain an entry to start the license server. Search the file for **lmgrd**
  and comment out the command.

* On older systems, there could be a startup script in either **/etc/rc3.d** or **/etc/rc5.d**. There is no
  fixed name for the startup script, although it usually begins with a capital S. If you are having
  trouble finding it, try running the command:

  ```shell
  cd /etc
  grep -r lmgrd *
  ```

  If this command finds any scripts that contain the string lmgrd, the name is shown on the screen.
  When you find the script, move it to /tmp using the following command:

  ```shell
  mv <filename> /tmp
  ```



## D) Maximum Client Connections to License Daemon

SLB license server has a maximum connection limit of 30,000 jobs, but best practice is to limit connections to 10,000 jobs for optimal performance. However, this limit can be exceeded when using more powerful hardware.

On Linux systems each connection requires a file descriptor, if the ulimit for file descriptors is exceeded,
clients may encounter the error LM_CANTCONNECT (-15)and the server debug log may show the entry:
“***This license server system can handle no more concurrent clients since it is out of file descriptors***”..

To prevent this, `ulimit` for file descriptors should be adjusted up to at least 10,000 or as large as the maximum number of connections expected.

### How to adjust it using `systemd`

**1- Create or Edit the override configuration file for `slbsls` service**

```shell
sudo systemctl edit slbsls.service
```

Add:

```text
[Service]
LimitNOFILE=10000
```

**2- Reload `systemd` and restart the service**

```shell
systemctl daemon-reload
systemctl restart slbsls.service
```

### Test the installed server

After you create the scripts, you can test if the server works correctly using the following command:

```shell
systemctl start slbsls
```


If the server has started successfully, a log file called `/var/log/flex/flex.log`. If any problems are encountered during operation or start-up, details are also output to the log file.



## E) Linux Firewall Settings

To set up a firewall in Linux, the simplest solution is to lock the license server to fixed ports, then open those ports in the firewall. To lock the server to fixed ports, you must edit the license file.

**First**, add a port number for `lmgrd` at the end of the line starting **SERVER** and add a different port to the
end of the **VENDOR** line for the vendor daemon. You must select unused ports. 

The example license file below sets the ports to `7321` and `7322` for `lmgrd` and `slbsl`, respectively.

```text
# Dongle Tracking Number: 1-1047472
SERVER this_host SLBID=7C3D50----00000 7321
VENDOR slbsls port=7322
USE_SERVER
FEATURE Petrel_0_MAAAAAGBHsTUA slbsls 2018.07 09-aug-2018 1 \
VENDOR_STRING=nynyyyyyyyyyynnnynynnnnnnynn \
SUPERSEDE=Petrel_0_MAAAAAGBHsTUA ISSUED=10-May-2007 \
SN=1-BSC7D5-0 TS_OK AUTH={ lmgrd.slb=( LK=15EFFBAF2EA4 \
SIGN="0053 DA87 00B1 12CE BEA6 32DE 6A17 8A00 0569 E602 6264 \
8094 B139 D7E0 B43E" SIGN2="0084 2DFF B6EE 3CD5 CE26 8196 DE97 \
CF00 6616 BC9E B580 1F62 3875 52A5 9928") \
slbfd=( SIGN="0089 8171 DE2E 43D0 32E2 ABCC 4575 F800 6F2A \
6B94 2C27 71EC 6C00 0D5A 19D8") slbsls=( SIGN="0089 8171 DE2E 43D0 \
32E2 ABCC 4575 F800 6F2A 6B94 2C27 71EC 6C00 0D5A 19D8") \
```

**Next**, open the firewall ports with the following commands. A root login will likely be required to run the
following commands.

* On systems that use iptables:

  ```shell
  iptables -A INPUT -p tcp --dport 7321 -j ACCEPT
  iptables -A INPUT -p tcp --dport 7322 -j ACCEPT
  iptables save
  ```

* On systems that use the firewall-cmd command:

  ```shell
  firewall-cmd --add-port=7321/tcp --permanent
  firewall-cmd --add-port=7322/tcp --permanent
  ```

When setting up a firewall, it is recommended to set the port information in the license variables on the
clients, even if you have set the default of 27000–27009. For example, use the setting `27000@licenseserver` rather than the default `@license-server`. This speeds up communication between the client and server.



## Outro

Credits to:

	- Schlumberger SIS 
	- Marcus Zou