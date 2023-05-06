# Custom Windows ISO

In this tutorial, we'll guide you through the process of creating a custom Windows Lite image specifically optimized for best overall performance and battery health. By creating a custom image, you can remove any unnecessary bloatware and services, reduce boot times, and improve overall system performance.

Before we begin, it's important to note that this process can be time-consuming and requires some technical knowledge. It's recommended that you have experience with using Windows PowerShell and a basic understanding of Windows system components. Additionally, creating a custom image can potentially void your warranty, so proceed with caution.

With that said, let's get started!

# Prerequisites

- First things first, get the [Official Windows 11 ISO]([Download Windows 11 (microsoft.com)](https://www.microsoft.com/software-download/windows11)) (remember to check the file checksum),
- [7-Zip]([7-Zip](https://7-zip.org/)),
- [Windows 11 ADK]([Download and install the Windows ADK | Microsoft Learn](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install)) (just the Deployment tools),
- and finally, fairly basic PowerShell knowledge.

# Warming Up

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

## Extracting ISO contents

To begin modifying a Windows image, you must first extract the ISO file to the designated **I** folder. The file that we will primarily modify is located at `I\sources\install.wim`.

Below is the PowerShell script that extracts the Windows 11 ISO file to the designated **I** folder:

```powershell
$ISO_location = "$env:USERPROFILE\Downloads\Win11_22H2_English_x64v1.iso"
Set-Location "$env:PROGRAMFILES\7-Zip"
.\7z.exe x -o"$WLite_ROOT\I" $ISO_location
```

> [!tip]- Adjust Paths
> Before using the PowerShell script to extract the Windows 11 ISO file, make sure to replace the `$ISO_location` variable with the exact location of your Windows 11 ISO file. Also, ensure that you specify the location of the `7z.exe` file by replacing the path in the script. This will ensure that the ISO file is correctly extracted to the designated **I** folder.

## Downloading Updates

To apply updates to your custom Windows 11 image, go to the [Microsoft Update Catalog](https://www.catalog.update.microsoft.com/Home.aspx) and download the updates you want to apply. Once downloaded, move the update packages to the designated **U** folder. This will ensure that the updates are correctly applied during the image creation process.

## Exporting Drivers

If you're feeling a bit lazy, you can use this PowerShell script to export the drivers from your current Windows 11 installation to the designated **D** folder. This will ensure that the necessary drivers are included in the custom Windows 11 image:

```powershell
Export-WindowsDriver -Online -Destination "$WLite_ROOT\D"
```

## Mounting Windows 11 Edition

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

## Installing Updates

This PowerShell script will install Windows updates located in the **U** folder:

```powershell
Set-Location $WLite_ROOT
Get-ChildItem -Path .\U -Filter *.msu | foreach {  Add-WindowsPackage -PackagePath $_.FullName -Path "$WLite_ROOT\M" -NoRestart }
```

# Removing Bloatware

## Removing Wi-Fi, Ethernet Packages

Instead of using Windows pre-installed Wi-Fi/Ethernet packages, install your drivers.

```powershell
$packages = Get-WindowsPackage -Path .\M | Where-Object { $_.PackageName -clike "Microsoft-Windows-Ethernet-Client*" -or $_.PackageName -clike "Microsoft-Windows-Wifi-Client*" }
foreach ($p in $packages) {
	Remove-WindowsPackage -Path .\M -PackageName $p.PackageName -NoRestart
}
```

## Other Packages

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

## Remove Capabilities

```powershell
Get-WindowsCapability -Path .\M | Where-Object { $_.State -ne "NotPresent" -and $_.Name -clike "Language.Handwriting*" } | foreach { Remove-WindowsCapability -Path .\M -Name $_.Name }
Get-WindowsCapability -Path .\M | Where-Object { $_.State -ne "NotPresent" -and $_.Name -clike "Language.OCR*" } | foreach { Remove-WindowsCapability -Path .\M -Name $_.Name }
Get-WindowsCapability -Path .\M | Where-Object { $_.State -ne "NotPresent" -and $_.Name -clike "Language.Speech*" } | foreach { Remove-WindowsCapability -Path .\M -Name $_.Name }
Get-WindowsCapability -Path .\M | Where-Object { $_.State -ne "NotPresent" -and $_.Name -clike "Language.TextToSpeech*" } | foreach { Remove-WindowsCapability -Path .\M -Name $_.Name }
```


## Disable Windows Features

```powershell
$features = "Printing-PrintToPDFServices-Features",
"MicrosoftWindowsPowerShellV2Root",
"MicrosoftWindowsPowerShellV2",
"NetFx4-AdvSrvs",
"WCF-Services45",
"WCF-TCP-PortSharing45",
"MediaPlayback",
"SearchEngine-Client-Package",
"WorkFolders-Client",
"Printing-Foundation-Features",
"Printing-Foundation-InternetPrinting-Client",
"SmbDirect"

foreach ($f in $features) {
	Disable-WindowsOptionalFeature -Path .\M -FeatureName $f
}
```


## Uninstall Provisioned Apps

```powershell
$apps = "Clipchamp.Clipchamp*",
"Microsoft.549981C3F5F10*",
"Microsoft.Bing*",
"Microsoft.GamingApp*",
"Microsoft.GetHelp*",
"Microsoft.Getstarted*",
"Microsoft.MicrosoftOfficeHub*",
"Microsoft.MicrosoftSolitaireCollection*",
"Microsoft.MicrosoftStickyNotes*",
"Microsoft.Paint*",
"Microsoft.People*",
"Microsoft.PowerAutomateDesktop*",
"Microsoft.ScreenSketch*",
"Microsoft.Todos*",
"Microsoft.StorePurchaseApp*", # Store Purchase App
"Microsoft.Windows.Photos*",
"Microsoft.WindowsAlarms*",
"Microsoft.WindowsCalculator*",
"Microsoft.WindowsCamera*",
"microsoft.windowscommunicationsapps*",
"Microsoft.WindowsFeedbackHub*",
"Microsoft.WindowsMaps*",
"Microsoft.WindowsNotepad*",
"Microsoft.WindowsSoundRecorder*",
"Microsoft.WindowsStore*",
"Microsoft.WindowsTerminal*",
"Microsoft.Xbox*",
"Microsoft.YourPhone*",
"Microsoft.Zune*",
"MicrosoftCorporationII.QuickAssist*",
"MicrosoftWindows.Client.WebExperience*"

foreach ($a in $apps) {
	Get-AppxProvisionedPackage -Path .\M | Where-Object { $_.PackageName -clike $a } | foreach { Remove-AppxProvisionedPackage -Path .\M -PackageName $_.PackageName }
}
```