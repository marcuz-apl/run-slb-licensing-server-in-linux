# Bible to make WSL2 alive 24/7



## Question

WSL2 will automatically shut down when the terminal is closed. I need it alive in the background.

The thing is that I've installed Apache2 in WSL2 Ubuntu (It's actually `Ubuntu-24.04`).

```
sudo apt-get install apache2
sudo systemctl enable --now apache2
```

I can see in `ss -tuln | grep :80` that it's listening on port 80.

(Optionally) replace the `index.html` file in `/var/www/html/` folder:

```shell
<!DOCTYPE html>
<html>
<title>Welcome to DarkSink!</title>
<body>

<h1>This website is <font color="green">ACTIVE</font>. Hola!</h1>
<p>All rights reserved by: <font color="red">AlfaZen.org</font>.</p>

</body>
</html>
```

Then I load http://localhost/ on Windows host side and the Apache2 Ubuntu Default Page opens.

The Default Page  stops responding on the Windows host the moment I exit the WSL2 Ubuntu.



## Solutions



#### Step 1- Configure .wslconfig in Windows

You can force WSL2 to keep running with:

```
[wsl2]
vmIdleTimeout=-1
```

in `%userprofile%/.wslconfig` file.



#### Step 2- use `wsl --exec` command

1- Ensure the `dbus` is installed and running on WSL2 Distro: `dbus` is built-in in Ubuntu, while not installed in Debian.

```shell
## For Debian only
sudo apt install dbus
sudo apt install dbus-x11    ## a must-have package to run dbus-daemon remotely
sudo systemctl enable dbus
sudo systemctl start dbus
## Verify if dbus service is running
sudo systemctl status dbus
## Or check out dbus-daemon process
ps aux | grep dbus-daemon
```

2A- We can further refine method above to make the process automatic by adding the following bash commands to your WSL2 distro's `.bashrc` file:

```
#-----------------------
# keep wsl distro alive
#-----------------------
if ! pgrep -u "$(whoami)" -x "dbus-daemon" > /dev/null; then
   dbus-launch true
fi
```

- The `pgrep` line is to ensure that we don't create multiple `dbus` processes for this purpose (1 process suffices).

Restart  the WSL distro and close the terminal, the WSL instance shall stay alive. 

2B- Alternatively, the very service can be started from WSL by running the command below in __PowerShell__, as such the WSL2 instance should start, then you could `exit`, and the WSL2 instance remains running:

```powershell
wsl -d ubuntu24 --exec dbus-launch true
```

Please replace `ubuntu24` with the name of your WSL instance.

#### More observation and Issue

The `Apache2` service stopped after rebooting the Windows.



#### Step 3- Start the `wsl` upon startup of Windows

Back to Windows, Create a VBS and let it run after Windows startup. Here is the file content of `wsl.vbs`:

```shell
Set ws = CreateObject("Wscript.Shell")
ws.run "wsl -d ubuntu24 --exec dbus-launch true", vbhide
```

Note: `ubuntu24` in this file shall be changed as per your exact WSL2 distro name.

Then put this `wsl.vbs` file into the folder:

`%APPDATA%\Roaming\Microsoft\Windows\Start Menu\Programs\Start-up` (preferred)

Or the system path: 

`C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Start-up`

That's it.



#### Step 4- Schedule the`wsl.vbs` as needed

Sometimes, the Step 3 is not enough due to Windows system unable to auto-launch the `wsl.vbs` in the `startup` folder, then Run "`Task Scheduler`" to ensure an auto-launching of such:

```shell
Action = "Start a program"
	Details = "C:\Users\myUserID\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\wsl.vbs"
Trigger = "At log on"
	Details = "At log on of Domain\MyDomainID"
	Status = "Enabled"
```

Restart your Windows, you should have everything in place. Hola.



## The End
