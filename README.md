# Custom Windows ISO

In this tutorial, we'll guide you through the process of creating a custom Windows Lite image specifically optimized for best overall performance and battery health. By creating a custom image, you can remove any unnecessary bloatware and services, reduce boot times, and improve overall system performance.

Before we begin, it's important to note that this process can be time-consuming and requires some technical knowledge. It's recommended that you have experience with using Windows PowerShell and a basic understanding of Windows system components. Additionally, creating a custom image can potentially void your warranty, so proceed with caution. Also, you may want to read this article on Obsidian ('cause this is Obsidian Markdown).

With that said, let's get started!

## Prerequisites

- First things first, get the [Official Windows 11 ISO](https://www.microsoft.com/software-download/windows11) (remember to check the file checksum),
- [7-Zip](https://7-zip.org/),
- [Windows 11 ADK](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install) (just the Deployment tools),
- and finally, fairly basic PowerShell knowledge.

## Warming Up

We are using a specific folder structure. You need to create a root folder where all the necessary components will be saved:

- **M**: For wim **M**ount folder,
- **I**: For **I**SO contents,
- **D**: For **D**rivers,
- **U**: for Windows **U**pdate packages.

To create this folder structure, you can use the PowerShell script provided. 
```powershell
$WLite_ROOT = "C:\W" # This is the root folder. Change it to wherever you want.
New-Item -ItemType Directory -Path $WLite_ROOT | Out-Null
Set-Location $WLite_ROOT
New-Item -ItemType Directory -Path ".\M" | Out-Null
New-Item -ItemType Directory -Path ".\I" | Out-Null
New-Item -ItemType Directory -Path ".\D" | Out-Null
New-Item -ItemType Directory -Path ".\U" | Out-Null
```


> [!tip]- WLite_ROOT
> Be sure to replace `C:\W` with the actual path to the location where you want the `$WLite_ROOT` folder to be located. Also, keep in mind that Windows can have issues with long paths, so choose a short path if possible.

### Extracting ISO contents

To begin modifying a Windows image, you must first extract the ISO file to the designated **I** folder. The file that we will primarily modify is located at `I\sources\install.wim`.

Below is the PowerShell script that extracts the Windows 11 ISO file to the designated **I** folder:

```powershell
$ISO_location = "$env:USERPROFILE\Downloads\Win11_22H2_English_x64v1.iso"
Set-Location "$env:PROGRAMFILES\7-Zip"
.\7z.exe x -o"$WLite_ROOT\I" $ISO_location
```

> [!tip]- Adjust Paths
> Before using the PowerShell script to extract the Windows 11 ISO file, make sure to replace the `$ISO_location` variable with the exact location of your Windows 11 ISO file. Also, ensure that you specify the location of the `7z.exe` file by replacing the path in the script. This will ensure that the ISO file is correctly extracted to the designated **I** folder.

### Downloading Updates

To apply updates to your custom Windows 11 image, go to the [Microsoft Update Catalog](https://www.catalog.update.microsoft.com/Home.aspx) and download the updates you want to apply. Once downloaded, move the update packages to the designated **U** folder. This will ensure that the updates are correctly applied during the image creation process.

### Exporting Drivers

If you're feeling a bit lazy, you can use this PowerShell script to export the drivers from your current Windows 11 installation to the designated **D** folder. This will ensure that the necessary drivers are included in the custom Windows 11 image:

```powershell
Export-WindowsDriver -Online -Destination "$WLite_ROOT\D"
```

### Mounting Windows 11 Edition

To begin customizing your Windows 11 image, you'll need to mount the desired Windows edition from the `I\sources\install.wim` file. In this guide, we'll be using Windows 11 Pro.

First, you can check what editions you currently have in the `install.wim` file by using this PowerShell command:

```powershell
$wim_location = "$WLite_ROOT\I\sources\install.wim"
Get-WindowsImage -ImagePath $wim_location
```

Next, to mount the Windows 11 Pro edition to the designated **M** folder, use the following PowerShell command:

```powershell
$edition = 6 # Windows 11 Pro
Mount-WindowsImage -Index $edition -ImagePath $wim_location -Path "$WLite_ROOT\M" -Optimize -CheckIntegrity
```

This will allow you to make any necessary modifications to the mounted Windows 11 image in the **M** folder.

### Installing Updates

This PowerShell script will install Windows updates located in the **U** folder:

```powershell
Set-Location $WLite_ROOT
Get-ChildItem -Path .\U -Filter *.msu | foreach {  Add-WindowsPackage -PackagePath $_.FullName -Path "$WLite_ROOT\M" -NoRestart }
```

## Removing Bloatware

Let's jump right into removing the bloatwares.

### Unnecessary Packages

Starting with the unnecessary packages. These packages are not commonly used or needed and may take up unnecessary space on your system. It's recommended to remove them to optimize your system's performance and free up space.

#### Removing Wi-Fi, Ethernet Packages

Instead of using Windows pre-installed Wi-Fi/Ethernet packages, install your drivers.

```powershell
$packages = Get-WindowsPackage -Path .\M | Where-Object { $_.PackageName -clike "Microsoft-Windows-Ethernet-Client*" -or $_.PackageName -clike "Microsoft-Windows-Wifi-Client*" }
foreach ($p in $packages) {
	Remove-WindowsPackage -Path .\M -PackageName $p.PackageName -NoRestart
}
```

#### Other Packages

Consider removing Notepad, WordPad, Internet Explorer, and Media Player from your system. There are better alternatives available, so it's worth giving them a try instead.

```powershell
# For Mail, Calend...
Get-WindowsPackage -Path .\M | Where-Object { $_.PackageName -clike "Microsoft-OneCore-ApplicationModel-Sync-Desktop-FOD-Package*" } | Remove-WindowsPackage -Path .\M
# Windows Hello Face (just face, not fingerprint or any other)
Get-WindowsPackage -Path .\M | Where-Object { $_.PackageName -clike "Microsoft-Windows-Hello-Face-Package*" } | foreach { Remove-WindowsPackage -Path .\M -PackageName $_.PackageName }
# IE
Get-WindowsPackage -Path .\M | Where-Object { $_.PackageName -clike "Microsoft-Windows-InternetExplorer*" } | foreach { Remove-WindowsPackage -Path .\M -PackageName $_.PackageName }
# Media Player
Get-WindowsPackage -Path .\M | Where-Object { $_.PackageName -clike "Microsoft-Windows-MediaPlayer*" } | foreach { Remove-WindowsPackage -Path .\M -PackageName $_.PackageName }
# Notepad
Get-WindowsPackage -Path .\M | Where-Object { $_.PackageName -clike "Microsoft-Windows-Notepad*" } | foreach { Remove-WindowsPackage -Path .\M -PackageName $_.PackageName }
# PowerShell Script Editor, absolutely unnecessary
Get-WindowsPackage -Path .\M | Where-Object { $_.PackageName -clike "Microsoft-Windows-PowerShell-ISE*" } | foreach { Remove-WindowsPackage -Path .\M -PackageName $_.PackageName }
# Printing, you may not want to remove this one
Get-WindowsPackage -Path .\M | Where-Object { $_.PackageName -clike "Microsoft-Windows-Printing*" } | foreach { Remove-WindowsPackage -Path .\M -PackageName $_.PackageName }
# Stepsrecorder
Get-WindowsPackage -Path .\M | Where-Object { $_.PackageName -clike "Microsoft-Windows-StepsRecorder*" } | foreach { Remove-WindowsPackage -Path .\M -PackageName $_.PackageName }
# TabletPCMath
Get-WindowsPackage -Path .\M | Where-Object { $_.PackageName -clike "Microsoft-Windows-TabletPCMath*" } | foreach { Remove-WindowsPackage -Path .\M -PackageName $_.PackageName }
# More Wallpapers
Get-WindowsPackage -Path .\M | Where-Object { $_.PackageName -clike "Microsoft-Windows-Wallpaper-Content-Extended-FoD-Package*" } | foreach { Remove-WindowsPackage -Path .\M -PackageName $_.PackageName }
# WordPad
Get-WindowsPackage -Path .\M | Where-Object { $_.PackageName -clike "Microsoft-Windows-WordPad*" } | foreach { Remove-WindowsPackage -Path .\M -PackageName $_.PackageName }
# SSH
Get-WindowsPackage -Path .\M | Where-Object { $_.PackageName -clike "OpenSSH-Client*" } | foreach { Remove-WindowsPackage -Path .\M -PackageName $_.PackageName }
```

### Remove Capabilities

If you are using Text to speech, OCR, or Speech features, you may want to skip this part.

```powershell
$allcaps = Get-WindowsCapability -Path .\M | Where-Object { $_.State -ne "NotPresent" }
$rem_caps = @(
# Language features...
"Language.Handwriting*", 
"Language.OCR*", 
"Language.Speech*", 
"Language.TextToSpeech*",
# StepsRecorder
"App.StepsRecorder*", 
# IE
"Browser.InternetExplorer*",
# Windows Hello (just the Face part)
"Hello.Face*",
# MathRecognizer
"MathRecognizer*",
# Wallpapers
"Microsoft.Wallpapers.Extended*",
# Notepad
"Microsoft.Windows.Notepad*",
# for Powershell IDE
"Microsoft.Windows.PowerShell.ISE*",
# WordPad
"Microsoft.Windows.WordPad*",
# for syncing with Mail, Calendar, People, Contacts
"OneCoreUAP.OneSync*",
# for SSH agent
"OpenSSH.Client*")
foreach ($r in $rem_caps) {
	foreach ($c in $allcaps) {
		if ($c.Name -like $r) {
			echo "Removing $($c.Name)..."
			Remove-WindowsCapability -Name $c.Name -Path .\M
		}
	}
}
```


### Disable Windows Features

To keep Printer features or any other you may want, just take them out from the list before you run the script below.

```powershell
$features = @(
# Printing
"Printing-PrintToPDFServices-Features",
"Printing-Foundation-Features",
"Printing-Foundation-InternetPrinting-Client",
# Remote Desktop
"MSRDC-Infrastructure",
# Powershell
"MicrosoftWindowsPowerShellV2Root",
"MicrosoftWindowsPowerShellV2",
# .NET 3.5, 4, and etc.
"NetFx4-AdvSrvs",
"WCF-Services45",
"WCF-TCP-PortSharing45",
# Windows Media Player, etc.
"MediaPlayback",
"SearchEngine-Client-Package",
# File sharing, searching, syncing etc.
"WorkFolders-Client",
"SmbDirect")

foreach ($f in $features) {
	Disable-WindowsOptionalFeature -Path .\M -FeatureName $f
}
```

### Uninstall Provisioned Apps

If you wish to keep an app from the list below, simply delete it from the list before running the script.

```powershell
$apps = @("Clipchamp.Clipchamp*",
"Microsoft.549981C3F5F10*", # Cortana
"Microsoft.Bing*",
"Microsoft.GamingApp*",
"Microsoft.GetHelp*",
"Microsoft.Getstarted*",
"Microsoft.MicrosoftOfficeHub*",
"Microsoft.MicrosoftSolitaireCollection*",
"Microsoft.MicrosoftStickyNotes*",
"Microsoft.Paint*",
"Microsoft.People*",
"MicrosoftCorporationII.MicrosoftFamily*"
"Microsoft.PowerAutomateDesktop*",
"Microsoft.ScreenSketch*",
"Microsoft.Todos*",
#"Microsoft.StorePurchaseApp*", # Store Purchase App
"Microsoft.Windows.Photos*",
"Microsoft.WindowsAlarms*",
#"Microsoft.WindowsCalculator*",
"Microsoft.WindowsCamera*",
# Outlook, Calendar, etc.
"microsoft.windowscommunicationsapps*",
"Microsoft.WindowsFeedbackHub*",
"Microsoft.WindowsMaps*",
"Microsoft.WindowsNotepad*",
"Microsoft.WindowsSoundRecorder*",
#"Microsoft.WindowsStore*",
"Microsoft.WindowsTerminal*",
"Microsoft.Xbox*",
"Microsoft.YourPhone*",
# Music and Video Player
"Microsoft.Zune*",
"MicrosoftCorporationII.QuickAssist*",
# Widgets
"MicrosoftWindows.Client.WebExperience*")

foreach ($a in $apps) {
	Get-AppxProvisionedPackage -Path .\M | Where-Object { $_.PackageName -like $a } | foreach { Remove-AppxProvisionedPackage -Path .\M -PackageName $_.PackageName }
}
```

### Removing Microsoft Edge

```powershell
rd "C:\scratchdir\Program Files (x86)\Microsoft"
```

### Removing OneDrive

```powershell
takeown /f C:\W\M\Windows\System32\OneDriveSetup.exe
icacls C:\W\M\Windows\System32\OneDriveSetup.exe /grant Administrators:F /T /C
cmd /c del /f /q /s "C:\W\M\Windows\System32\OneDriveSetup.exe"
takeown /f C:\W\M\Windows\System32\OneDrive.ico
icacls C:\W\M\Windows\System32\OneDrive.ico /grant Administrators:F /T /C
cmd /c del /f /q /s "C:\W\M\Windows\System32\OneDrive.ico"
Get-ChildItem -Filter *onedrive-setup* -Path ".\M\Windows\WinSxs\" | foreach { takeown /f $_.FullName; icacls $_.FullName /grant Administrators:F /T /C; del -Force -Recurse -Path $_.FullName }
```

## Registry Tweaks

### Load hives

```powershell
reg load HKLM\zCOMPONENTS ".\M\Windows\System32\config\COMPONENTS"
reg load HKLM\zDEFAULT ".\M\Windows\System32\config\default"
reg load HKLM\zNTUSER ".\M\Users\Default\ntuser.dat"
reg load HKLM\zSOFTWARE ".\M\Windows\System32\config\SOFTWARE"
reg load HKLM\zSYSTEM ".\M\Windows\System32\config\SYSTEM"
```

### Disable Teams from Auto Installing

```powershell
reg add "HKLM\zSOFTWARE\Microsoft\Windows\CurrentVersion\Communications" /v "ConfigureChatAutoInstall" /t REG_DWORD /d "0" /f
```

### Disabling Sponsored Apps

```powershell
reg add "HKLM\zNTUSER\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager" /v "PreInstalledAppsEnabled" /t REG_DWORD /d "0" /f
reg add "HKLM\zNTUSER\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager" /v "SilentInstalledAppsEnabled" /t REG_DWORD /d "0" /f
reg add "HKLM\zNTUSER\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager" /v "DisableWindowsConsumerFeature" /t REG_DWORD /d "1" /f
reg add "HKLM\zSOFTWARE\Microsoft\PolicyManager\current\device\Start" /v "ConfigureStartPins" /t REG_SZ /d '{"pinnedList": [{}]}' /f
```

### Enable local accounts on OOBE

```powershell
reg add "HKLM\zSOFTWARE\Microsoft\Windows\CurrentVersion\OOBE" /v "BypassNRO" /t REG_DWORD /d "1" /f
```

### Disable Chat icon

```powershell
reg add "HKLM\zSOFTWARE\Policies\Microsoft\Windows\Windows Chat" /v "ChatIcon" /t REG_DWORD /d "3" /f
reg add "HKLM\zNTUSER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced" /v "TaskbarMn" /t REG_DWORD /d "0" /f
```

### Disable Unnecessary Services

```powershell
# ActiveX Installer (AxInstSV)
reg add "HKLM\zSYSTEM\ControlSet001\Services\AxInstSV" /v "Start" /t REG_DWORD /d "4" /f
# dmwappushsvc
reg add "HKLM\zSYSTEM\ControlSet001\Services\dmwappushservice" /v "Start" /t REG_DWORD /d "4" /f
# Downloaded Maps Manager
reg add "HKLM\zSYSTEM\ControlSet001\Services\MapsBroker" /v "Start" /t REG_DWORD /d "4" /f
# Geolocation Service
reg add "HKLM\zSYSTEM\ControlSet001\Services\lfsvc" /v "Start" /t REG_DWORD /d "4" /f
# Internet Connection Sharing (ICS)
reg add "HKLM\zSYSTEM\ControlSet001\Services\SharedAccess" /v "Start" /t REG_DWORD /d "4" /f
# Link-Layer Topology Discovery Mapper
reg add "HKLM\zSYSTEM\ControlSet001\Services\lltdsvc" /v "Start" /t REG_DWORD /d "4" /f
# Phone
reg add "HKLM\zSYSTEM\ControlSet001\Services\PhoneSvc" /v "Start" /t REG_DWORD /d "4" /f
# Network Connection Broker, disabling prevents uwp apps to access network
#reg add "HKLM\zSYSTEM\ControlSet001\Services\NcbService" /v "Start" /t REG_DWORD /d "4" /f
# Microsoft Passport
#reg add "HKLM\zSYSTEM\ControlSet001\Services\NgcSvc" /v "Start" /t REG_DWORD /d "4" /f
# Microsoft Passport Container
#reg add "HKLM\zSYSTEM\ControlSet001\Services\wlidsvc" /v "Start" /t REG_DWORD /d "4" /f
# Print Spooler
reg add "HKLM\zSYSTEM\ControlSet001\Services\Spooler" /v "Start" /t REG_DWORD /d "4" /f
# Printer Extensions and Notifications
reg add "HKLM\zSYSTEM\ControlSet001\Services\PrintNotify" /v "Start" /t REG_DWORD /d "4" /f
# Program Compatibility Assistant Service
reg add "HKLM\zSYSTEM\ControlSet001\Services\PcaSvc" /v "Start" /t REG_DWORD /d "4" /f
# Radio Management Service
reg add "HKLM\zSYSTEM\ControlSet001\Services\RmSvc" /v "Start" /t REG_DWORD /d "4" /f
# Sensor Data Service
reg add "HKLM\zSYSTEM\ControlSet001\Services\SensorDataService" /v "Start" /t REG_DWORD /d "4" /f
# Sensor Monitoring Service
reg add "HKLM\zSYSTEM\ControlSet001\Services\SensrSvc" /v "Start" /t REG_DWORD /d "4" /f
# Shell Hardware Detection, disabling disables autoplay, which is fine for me.
reg add "HKLM\zSYSTEM\ControlSet001\Services\ShellHWDetection" /v "Start" /t REG_DWORD /d "4" /f
# Smart Card Device Enumeration Service
reg add "HKLM\zSYSTEM\ControlSet001\Services\ScDeviceEnum" /v "Start" /t REG_DWORD /d "4" /f
# SSDP Discovery
reg add "HKLM\zSYSTEM\ControlSet001\Services\SSDPSRV" /v "Start" /t REG_DWORD /d "4" /f
# Still Image Acquisition Events
reg add "HKLM\zSYSTEM\ControlSet001\Services\WiaRpc" /v "Start" /t REG_DWORD /d "4" /f
# Touch Keyboard and Handwriting Panel Service
reg add "HKLM\zSYSTEM\ControlSet001\Services\TabletInputService" /v "Start" /t REG_DWORD /d "4" /f
# UPnP Device Host
reg add "HKLM\zSYSTEM\ControlSet001\Services\upnphost" /v "Start" /t REG_DWORD /d "4" /f
# WalletService
reg add "HKLM\zSYSTEM\ControlSet001\Services\WalletService" /v "Start" /t REG_DWORD /d "4" /f
# Windows Camera Frame Server
reg add "HKLM\zSYSTEM\ControlSet001\Services\FrameServer" /v "Start" /t REG_DWORD /d "4" /f
reg add "HKLM\zSYSTEM\ControlSet001\Services\FrameServerMonitor" /v "Start" /t REG_DWORD /d "4" /f
# Touch Keyboard and Handwriting Panel Service
reg add "HKLM\zSYSTEM\ControlSet001\Services\TabletInputService" /v "Start" /t REG_DWORD /d "4" /f
# Windows Image Acquisition (WIA)
reg add "HKLM\zSYSTEM\ControlSet001\Services\stisvc" /v "Start" /t REG_DWORD /d "4" /f
# Windows Insider Service
reg add "HKLM\zSYSTEM\ControlSet001\Services\wisvc" /v "Start" /t REG_DWORD /d "4" /f
# Windows Push Notifications System Service
reg add "HKLM\zSYSTEM\ControlSet001\Services\WpnService" /v "Start" /t REG_DWORD /d "4" /f
# Xbox
reg add "HKLM\zSYSTEM\ControlSet001\Services\XblAuthManager" /v "Start" /t REG_DWORD /d "4" /f
reg add "HKLM\zSYSTEM\ControlSet001\Services\XblGameSave" /v "Start" /t REG_DWORD /d "4" /f
reg add "HKLM\zSYSTEM\ControlSet001\Services\XboxNetApiSvc" /v "Start" /t REG_DWORD /d "4" /f
# Windows Update
reg add "HKLM\zSYSTEM\ControlSet001\Services\wuauserv" /v "Start" /t REG_DWORD /d "4" /f
# Windows Search
reg add "HKLM\zSYSTEM\ControlSet001\Services\WSearch" /v "Start" /t REG_DWORD /d "4" /f
# Windows Security
reg add "HKLM\zSYSTEM\ControlSet001\Services\SecurityHealthService" /v "Start" /t REG_DWORD /d "4" /f
reg add "HKLM\zSYSTEM\ControlSet001\Services\wscsvc" /v "Start" /t REG_DWORD /d "4" /f
reg add "HKLM\zSYSTEM\ControlSet001\Services\webthreatdefsvc" /v "Start" /t REG_DWORD /d "4" /f
# Delivery Optimization
reg add "HKLM\zSYSTEM\ControlSet001\Services\DoSvc" /v "Start" /t REG_DWORD /d "4" /f
# Background Intelligent Transfer Service, if u want to use store, just comment this line too..
reg add "HKLM\zSYSTEM\ControlSet001\Services\BITS" /v "Start" /t REG_DWORD /d "4" /f
# AllJoyn
reg add "HKLM\zSYSTEM\ControlSet001\Services\AJRouter" /v "Start" /t REG_DWORD /d "4" /f
# Distributed Link Tracking Client
reg add "HKLM\zSYSTEM\ControlSet001\Services\TrkWks" /v "Start" /t REG_DWORD /d "4" /f
# Bitlocker
reg add "HKLM\zSYSTEM\ControlSet001\Services\BDESVC" /v "Start" /t REG_DWORD /d "4" /f
# Dialog Blocking Service
reg add "HKLM\zSYSTEM\ControlSet001\Services\DialogBlockingService" /v "Start" /t REG_DWORD /d "4" /f
# MS AppV Client
reg add "HKLM\zSYSTEM\ControlSet001\Services\AppVClient" /v "Start" /t REG_DWORD /d "4" /f
# MS Keyboard Filter
reg add "HKLM\zSYSTEM\ControlSet001\Services\MsKeyboardFilter" /v "Start" /t REG_DWORD /d "4" /f
# Offline Files
reg add "HKLM\zSYSTEM\ControlSet001\Services\CscService" /v "Start" /t REG_DWORD /d "4" /f
# User Experience Virtualization Service
reg add "HKLM\zSYSTEM\ControlSet001\Services\UevAgentService" /v "Start" /t REG_DWORD /d "4" /f
# Windows Connect Now
reg add "HKLM\zSYSTEM\ControlSet001\Services\wcncsvc" /v "Start" /t REG_DWORD /d "4" /f
# SysMain, frequently used apps, optimizing etc. just pure bullshit.
reg add "HKLM\zSYSTEM\ControlSet001\Services\SysMain" /v "Start" /t REG_DWORD /d "4" /f
# Mixed Reality, if you don't have a headset disable it.
reg add "HKLM\zSYSTEM\ControlSet001\Services\MixedRealityOpenXRSvc" /v "Start" /t REG_DWORD /d "4" /f
# Related with Store.
reg add "HKLM\zSYSTEM\ControlSet001\Services\PushToInstall" /v "Start" /t REG_DWORD /d "4" /f
# Remote Registry and Access
reg add "HKLM\zSYSTEM\ControlSet001\Services\RemoteRegistry" /v "Start" /t REG_DWORD /d "4" /f
reg add "HKLM\zSYSTEM\ControlSet001\Services\RemoteAccess" /v "Start" /t REG_DWORD /d "4" /f
```

### Save registry tweaks

```powershell
reg unload HKLM\zCOMPONENTS
reg unload HKLM\zDRIVERS
reg unload HKLM\zDEFAULT
reg unload HKLM\zNTUSER
reg unload HKLM\zSCHEMA
reg unload HKLM\zSOFTWARE
reg unload HKLM\zSYSTEM
```

## Pre-Install Drivers

To pre-install drivers using Windows PowerShell, you can follow these steps:

### Step 1: Export Host Drivers

The first step is to export all drivers currently installed on your host computer. To do this, enter the following command in Windows PowerShell:

```powershell
Export-WindowsDriver -Online -Destination .\D
```

This command exports all currently active drivers on your computer and saves them to the **D** folder.

### Step 2: Installing Drivers

Now that you have exported all active drivers on your computer and copied the drivers you want to pre-install to a driver package folder, you can install them using the following command:

```powershell
Add-WindowsDriver -Recurse -Path .\M -Driver .\D
```

## Commit or Save

To save the changes we made, run the following command:

```powershell
Dismount-WindowsImage -Save -Path .\M
```

> [!bug]+ Applying changes via Commit way
> When making changes to a Windows image using DISM, there are two ways to apply those changes: committing or saving. While committing the changes may seem like a quick and easy way to apply your modifications, it is not as stable as saving. Committing the changes can sometimes result in unexpected errors or issues that may not be immediately apparent, whereas saving the changes ensures that the image is in a stable and reliable state. Therefore, it is recommended to always save your changes when modifying a Windows image to ensure the stability and reliability of the resulting image.


## Compress the Image

```powershell
Export-WindowsImage -SourceImagePath .\I\sources\install.wim -DestinationImagePath .\I\sources\install.max.wim -CompressionType max -SourceIndex $edition
del .\I\sources\install.wim
mv .\I\sources\install.max.wim .\I\sources\install.wim
```

## Create the iso

Press Ctrl+Esc keys at the same time or just press the Meta/Windows key on your keyboard to access Windows Start Menu, then start typing **Deployment and Imaging Tools Environment**, and launch it. Type the following to create a bootable media

```cmd
oscdimg -m -o -u2 -udfver102 -lWin11_Lite_v0.1 -bootdata:2#p0,e,bC:\W\I\boot\etfsboot.com#pEF,e,bC:\W\I\efi\microsoft\boot\efisys.bin C:\W\I C:\W\Windows11Lite.iso
```

## Disclaimer

This is just pure educational purposes only. I do not (re)distribute anything Windows related.
