# Windows
Installation and configuration instructions for Windows 10 (Version 1903).

<!--
Fix drive previously used for unix partitions with `diskpart`.

```cmd
DISKPART> list disk
DISKPART> select disk 1
DISKPART> clean
DISKPART> create partition primary
DISKPART> active
DISKPART> format fs=Fat32 quick
```
-->

## Preparations
Download the latest [Windows 10](https://www.microsoft.com/en-us/software-download/windows10) image
and create the installation media using [Rufus](https://rufus.ie/).

Create the file `\sources\ei.cfg` on the installation media.

```ini
[EditionID]
Professional
[Channel]
Retail
[VL]
0
```

Create the file `\sources\pid.txt` on the installation media.

```ini
[PID]
Value={windows key}
```

Copy this repo and the latest graphics drivers installer to the installation media.

Set the BIOS date and time to the current local time.

**Keep the system disconnected from the network.**

## Installation
Boot the installation media.

```
Language to install: English (United States)
Time and currency format: {Current Time Zone Country}
Keyboard or input method: {Current Hardware Keyboard}
```

Choose a single word username starting with a capital letter to keep the `%UserProfile%` path
consistent and free from spaces.

## Setup
Modify the [setup/system.ps1](setup/system.ps1) script and execute it using the "Run with PowerShell" context menu.

## Universal Apps
List remaining apps.

```ps
Get-AppxPackage | Select Name,PackageFullName | Sort Name
```

Uninstall unwanted apps.

```ps
Get-AppxPackage "Microsoft.3DBuilder" | Remove-AppxPackage
```

Uninstall unwanted optional features.

```
Settings > Apps > Manage optional features
- Microsoft Quick Assist
- Windows Hello Face
```

Clean up the Start Menu.

## Group Policies
Configure group policies.

```
gpedit.msc > Computer Configuration > Administrative Templates > Windows Components

Speech
+ Allow Automatic Update of Speech Data: Disabled

Windows Defender Antivirus
+ Turn off Windows Defender Antivirus: Enabled

Windows Defender Antivirus > MAPS
+ Join Microsoft MAPS: Disabled

Windows Defender Antivirus > Network Inspection System
+ Turn on definition retirement: Disabled
+ Turn on protocol recognition: Disabled

Windows Defender Antivirus > Real-time Protection
+ Turn off real-time protection: Enabled
```

## Drivers & Updates
Disable automatic driver application installation.

```
Start > "Change device installation settings"
◉ No (your device might not work as expected)
```

1. Reboot the system.
2. Connect to the Internet.
3. Install Windows updates.

## Settings
Settings that have yet to be incorporated into the [setup/system.ps1](setup/system.ps1) script.

### System
```
Notifications & actions
+ Edit your quick actions
  ☑ All settings
  ☑ Network
  ☑ Night light
  ☑ Screen snip
+ Get notifications from these senders
  ☐ Focus assist (via Search)
```

### Devices
```
Typing
+ Spelling
  ☐ Autocorrect misspelled words
+ Typing
  ☐ Add a period after I double-tap the Spacebar

Pen & Windows Ink
+ Windows Ink Workspace
  ☐ Show recommended app suggestions

AutoPlay
+ Choose AutoPlay defaults
  Removable drive: Open folder to view files (File Explorer)
  Memory card: Open folder to view files (File Explorer)
```

### Network & Internet
```
Proxy
+ Automatic proxy setup
  ☐ Automatically detect settings
```

### Personalization
```
Colors
+ Choose your color
  Windows colors: Navy blue
Start
+ Start
  ☐ Show app list in Start menu
```

### Time & Language
```
Region > Additional date, time, & regional settings > Change date, time, or number formats
+ Formats > Additional settings...
  + Numbers
    Decimal symbol: .
    Digit grouping symbol: ␣
    Measurement system: Metric
  + Currency
    Decimal symbol: .
    Digit grouping symbol: ␣
  + Time
    Short time: HH:mm
    Long time: HH:mm:ss
  + Date
    Short date: yyyy-MM-dd
    Long date: ddd, d MMMM yyyy
    First day of week: Monday
+ Administrative > Copy settings...
  ☑ Welcome screen and system accounts
  ☑ New user accounts
```

### Ease of Access
```
Narrator
+ Use Narrator
  ☐ Turn on narrator
  ☐ Allow the shortcut key to start Narrator
Keyboard
+ Use your device without a physical keyboard
  ☐ Use the On-Screen Keyboard
+ Use Sticky Keys
  ☐ Allow the shortcut key to start Sticky Keys
+ Use Toggle Keys
  ☐ Allow the shortcut key to start Toggle Keys
+ Use Filter Keys
  ☐ Allow the shortcut key to start Filter Keys
```

### Privacy
```
General
+ Change privacy options
  ☐ Let apps use advertising ID to make ads more interesting …
  ☑ Let websites provide locally relevant content by accessing my language list
  ☐ Let Windows track app launches to improve Start and search results
  ☐ Show me suggested content in the Settings app
```

## Windows Libraries
Move unwanted Windows libraries.

1. Right click on `%UserProfile%\Pictures\Camera Roll` and select `Properties`.<br/>
   Select the `Location` tab and change the path to `%AppData%\Pictures\Camera Roll`.
2. Right click on `%UserProfile%\Pictures\Saved Pictures` and select `Properties`.<br/>
   Select the `Location` tab and change the path to `%AppData%\Pictures\Saved Pictures`.

## Indexing Options
Configure Indexing Options to only track the "Start Menu" and rebuild the index.

<!--
## Windows Mobile Device Center (WMDC)
Use the following script to fix Windows CE device connections.

```cmd
taskkill /im wmdc.exe /f
taskkill /im wmdcBase.exe /f
@timeout 1
sc stop WcesComm
sc stop RapiMgr
@timeout 1
reg add "HKLM\SOFTWARE\Microsoft\Windows CE Services" /v "GuestOnly" /t REG_DWORD /d 1 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Services\RapiMgr" /v "SvcHostSplitDisable" /t REG_DWORD /d 1 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Services\WcesComm" /v "SvcHostSplitDisable" /t REG_DWORD /d 1 /f
sc config WcesComm obj= "LocalSystem" password= ""
sc config RapiMgr obj= "LocalSystem" password= ""
@timeout 1
sc start RapiMgr
@timeout 1
sc start WcesComm
@timeout 1
@rem start "WMDC" "%windir%\WindowsMobile\wmdc.exe"
start "WMDC" "%windir%\WindowsMobile\wmdcBase.exe"
```

If the system does not need that script, switch to headless device center.

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows CE Services" /v "GuestOnly" /t REG_DWORD /d 1 /f
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v "Windows Mobile Device Center" /t REG_EXPAND_SZ /d "%windir%\WindowsMobile\wmdcBase.exe" /f
```
-->

## Firewall
Disable all rules in Windows Firewall except the following entries.

```
wf.msc
+ Inbound Rules
  Connect
  Core Networking - …
  Delivery Optimization (…)
  Hyper-V …
  Network Discovery (…)
+ Outbound Rules
  Connect
  Core Networking - …
  Hyper-V …
  Network Discovery (…)
```

Enable inbound rules for `File and Printer Sharing (Echo Request …)`. Modify `Private,Public`
rules for inbound and outbound IPv4 and IPv6 Echo Requests and select "Any IP address" under
`Remote IP address` in the `Scope` tab.

## Keymap
1. Install [res/keymap.zip](res/keymap.zip) to input German characters on a U.S. keyboard.
2. Configure Windows language preferences.
3. Reboot the system.

<!--
Use the [Microsoft Keyboard Layout Creator](https://www.microsoft.com/en-us/download/details.aspx?id=22339) to
create new keyboard layouts.
-->

## Microsoft Software
Configure [Microsoft Edge](https://en.wikipedia.org/wiki/Microsoft_Edge).

```
Settings
+ General
  Open Microsoft Edge with: A specific page or pages
    about:blank
  Open new tabs with: A blank page
  Show the favorites bar: Off
  Show the home button: Off
  Show sites I frequently visit in "Top sites": Off
  Show definitions inline for: Off
  Ask me what to do with each download: Off
+ Privacy & security
  Show search and site suggestions as I type: Off
  Show search history: Off
  Use page prediction: Off
+ Passwords & autofill
  Save form data: Off
```

Configure [Internet Explorer](https://en.wikipedia.org/wiki/Internet_Explorer).

```
Internet options
+ General
  Home page: about:blank
  Startup: Start with tabs from the last session
+ General > Tabs
  When a new tab is opened, open: A blank page
```

<!--
Configure [Outlook 2016](https://products.office.com/en/outlook).

```cmd
reg add "HKCU\SOFTWARE\Microsoft\Office\16.0\Outlook\Setup" /v "DisableOffice365SimplifiedAccountCreation" /t REG_DWORD /d 1 /f
```
-->

## Fonts
Install useful fonts.

* [DejaVu](https://sourceforge.net/projects/dejavu/files/dejavu)
* [Iconsolata](http://www.levien.com/type/myfonts/inconsolata.html)
* [IPA](http://ipafont.ipa.go.jp)

## Applications
Install third party software.

* [7-Zip](http://www.7-zip.org)
* [Chrome](https://www.google.com/chrome/)
* [Affinity Photo](https://affinity.serif.com/photo)
* [Affinity Designer](https://affinity.serif.com/designer)
* [Sketchbook Pro](http://www.autodesk.com/products/sketchbook-pro/overview)
* [FontForge](https://fontforge.github.io/en-US/downloads/windows-dl/)
* [Blender](https://www.blender.org/)
* [Gimp](https://www.gimp.org/)
* [MPV](https://mpv.srsfckn.biz/)
* [HxD](https://mh-nexus.de/en/downloads.php?product=HxD20)

<!--
* [Chrome Enterprise(https://cloud.google.com/chrome-enterprise/browser/download/#download)
* [Tor Browser](https://www.torproject.org/download/)

Install [qBittorrent](https://www.qbittorrent.org/) and configure the proxy.

```
Tor Browser > Menu > Options > General > Network Proxy > Settings…
```
-->

Install [Git](https://git-scm.com/downloads) with specific settings.

```
Select Destination Location
  C:\Program Files\Git

Select Components
  ☐ Windows Explorer integration
  ☑ Git LFS (Large File Support)
  ☐ Associate .git* configuration files with the default text editor
  ☐ Associate .sh files to be run with Bash

Select Start Menu Folder
  ☑ Don't create a Start Menu folder

Choosing the default editor used by Git
  [Select other editor as Git's default editor]
  Location of editor: C:\Program Files (x86)\Vim\vim81\gvim.exe
  [Test Custom Editor]

Adjusting your PATH environment
  ◉ Use Git from Git Bash only

Choosing the SSH executable
  ◉ Use OpenSSH

Choosing HTTPS transport backend
  ◉ Use the OpenSSL library

Configuring the line ending conversions
  ◉ Checkout as-is, commit as-is

Configuring the terminal emulator to use with Git Bash
  ◉ Use Windows' default console window

Configuring file system caching
  ☑ Enable file system caching
  ☑ Enable Git Credential Manager
  ☑ Enable symbolic links
```

Add `%ProgramFiles%\Git\cmd` to `Path`.

Install [gVim](http://www.vim.org) and create a config directory.

```cmd
git clone https://github.com/qis/vim %UserProfile%\vimfiles
```

<!--
## Server
Install software for Windows Server administration.

* [SQL Server Management Studio](https://msdn.microsoft.com/en-us/library/mt238290.aspx)
* [Remote Server Administration Tools for Windows 10](https://www.microsoft.com/en-us/download/details.aspx?id=45520)

Configure the WinRM client.

```cmd
Get-NetConnectionProfile
Set-NetConnectionProfile -InterfaceIndex {InterfaceIndex} -NetworkCategory Private
Enable-PSRemoting -SkipNetworkProfileCheck -Force
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "*" -Force
```

Configure the WinRM server.

```ps
Enable-PSRemoting -SkipNetworkProfileCheck -Force
Set-NetFirewallRule -Name "WINRM-HTTP-In-TCP-PUBLIC" -RemoteAddress Any
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "*" -Force
```

Configure the WinRM server to accept HTTPS connections.

```ps
winrm enumerate winrm/config/listener
New-SelfSignedCertificate -DnsName "{DomainName}" -CertStoreLocation Cert:\LocalMachine\My
cmd /C 'winrm create winrm/config/Listener?Address=*+Transport=HTTPS @{Hostname="{DomainName}"; CertificateThumbprint="{Thumbprint}"}'
netsh advfirewall firewall add rule name="Windows Remote Management (HTTPS-In)" dir=in action=allow protocol=TCP localport=5986
```

Connect over HTTP.

```ps
Enter-PSSession -ComputerName host.domain -Port 5985 -Credential administrator@domain
```

Connect over HTTPS.

```ps
$soptions = New-PSSessionOption -SkipCACheck
Enter-PSSession -ComputerName host.domain -Port 5986 -Credential administrator@domain -SessionOption $soptions -UseSSL
```
-->

<!--
## Control Panel
Add Control Panel shortcuts to the Windows start menu (use icons from `C:\Windows\System32\shell32.dll`).

[Control Panel Command Line Commands](https://www.lifewire.com/command-line-commands-for-control-panel-applets-2626060)
-->

<!--
## Anti-Virus
Suggested third party anti-virus exclusion lists.

```
Excluded Processes

%ProgramFiles(x86)%\Microsoft Visual Studio\2017\Professional\Common7\IDE\devenv.exe
%ProgramFiles(x86)%\Microsoft Visual Studio\2017\Professional\Common7\IDE\PerfWatson2.exe
%ProgramFiles(x86)%\Microsoft Visual Studio\2017\Professional\Common7\IDE\VcxprojReader.exe
%ProgramFiles(x86)%\Microsoft Visual Studio\2017\Professional\VC\Tools\MSVC\14.12.25827\bin\HostX86\x64\CL.exe
%ProgramFiles(x86)%\Microsoft Visual Studio\2017\Professional\VC\Tools\MSVC\14.12.25827\bin\HostX86\x64\link.exe

Excluded Directories

%ProgramFiles(x86)%\Microsoft Visual Studio\
%ProgramFiles(x86)%\Windows Kits\
%UserProfile%\AppData\Local\lxss\
C:\Workspace\
```
-->

## Windows Subsystem for Linux
Install a WSL distro from <https://aka.ms/wslstore>, launch it and download config files.

```sh
curl -L https://raw.githubusercontent.com/qis/windows/master/wsl/.bashrc -o .bashrc
curl -L https://raw.githubusercontent.com/qis/windows/master/wsl/.profile -o .profile
curl -L https://raw.githubusercontent.com/qis/windows/master/wsl/.tmux.conf -o .tmux.conf
rm -f .bash_history .bash_logout
touch .viminfo
```

Configure [sudo(8)](http://manpages.ubuntu.com/manpages/xenial/man8/sudo.8.html) with `sudo EDITOR=vim visudo`.

```sh
# Locale settings.
Defaults env_keep += "LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET"

# Profile settings.
Defaults env_keep += "MM_CHARSET EDITOR PAGER CLICOLOR LSCOLORS TMUX SESSION"

# User privilege specification.
root  ALL=(ALL) ALL
%sudo ALL=(ALL) NOPASSWD: ALL

# See sudoers(5) for more information on "#include" directives:
#includedir /etc/sudoers.d
```

Create `/etc/wsl.conf`.

```sh
[automount]
enabled=true
options=case=off,metadata,uid=1000,gid=1000,umask=022
```

Add the following line to `/etc/mdadm/mdadm.conf` (fixes some `apt` warnings).

```sh
# definitions of existing MD arrays
ARRAY <ignore> devices=/dev/sda
```

Modify the following lines in `/etc/pam.d/login` and `/etc/pam.d/sshd` (disables message of the day).

```sh
#session    optional    pam_motd.so motd=/run/motd.dynamic
#session    optional    pam_motd.so noupdate
```

Execute `chmod -x /etc/update-motd.d/*{-help-text,-motd-news}` to reduce spam.<br/>

**IMPORTANT**: Completely restart `bash.exe` to apply `/etc/wsl.conf` settings.

Create Windows symlinks.

```sh
mkdir -p ~/.config
ln -s /mnt/c/Users/Qis/.gitconfig ~/.gitconfig
ln -s /mnt/c/Users/Qis/vimfiles ~/.config/nvim
ln -s /mnt/c/Users/Qis/vimfiles ~/.vim
ln -s /mnt/c/Users/Qis/Documents ~/documents
ln -s /mnt/c/Users/Qis/Downloads ~/downloads
ln -s /mnt/c/Workspace ~/workspace
mkdir -p ~/.ssh; chmod 0700 ~/.ssh
for i in config id_rsa id_rsa.pub known_hosts; do
  ln -s /mnt/c/Users/Qis/.ssh/$i ~/.ssh/$i
done
chmod 0600 /mnt/c/Users/Qis/.ssh/* ~/.ssh/*
sudo mkdir -p /root/.config
sudo ln -s /mnt/c/Users/Qis/vimfiles /root/.config/nvim
sudo ln -s /mnt/c/Users/Qis/vimfiles /root/.vim
```

Install packages.

```sh
sudo apt update
sudo apt upgrade
sudo apt dist-upgrade
sudo apt autoremove
sudo apt install p7zip p7zip-rar zip unzip tree pwgen python-minimal pv
sudo apt install imagemagick pngcrush webp
sudo apt install apache2-utils
```

Install youtube-dl.

```sh
sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
sudo chmod 0755 /usr/local/bin/youtube-dl
```

Install [WSLtty](https://github.com/mintty/wsltty) for better terminal support.

<!--
Install [VcXsrv](https://github.com/ArcticaProject/vcxsrv/releases) for Xorg application support.
-->

Install an international locale.

```sh
sudo curl -L https://raw.githubusercontent.com/qis/windows/master/wsl/en_XX -o /usr/share/i18n/locales/en_XX
sudo tee -a /etc/locale.gen <<EOF
en_XX.UTF-8 UTF-8
EOF
sudo locale-gen
```

## Development
Follow the [development](development.md) guide to set up a developer workstation.

## Start Menu
![Start Menu](res/start.png)
