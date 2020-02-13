The ESU Installation and Activation is broken down into three sections.

1) A Global Condition to use for your Application's Deployment Type's Requirements
2) The Deployment script that will install and activate the ESU
3) The validation script for the Deployment Type

Information gathered from these sources:
- https://techcommunity.microsoft.com/t5/windows-it-pro-blog/obtaining-extended-security-updates-for-eligible-windows-devices/ba-p/1167091
- https://support.microsoft.com/en-us/help/4522133/procedure-to-continue-receiving-security-updates
- https://support.microsoft.com/en-us/help/4538483/extended-security-updates-esu-licensing-preparation-package

Credit to Ronni Pedersen for posting his PowerShell script to his blog and a comment from EVANS
- https://www.ronnipedersen.com/2019/12/18/managing-extended-security-updates-for-windows-7-using-microsoft-endpoint-manager/


On to the fun:

1) Create a Global Condition using this script.
The output will be `$true` if the Extended Security Updates (ESU) Licensing Preparation Package has been installed.
It will be `$false` if not detected.

```powershell
#region KB per OS for (ESU) Licensing Preparation Package
$OSESULicensingPreparationPackage = @{
    "Windows 7 SP1" = "KB4538483"
    "Server 2008 R2 SP1" = "KB4538483"
    "Server 2008 SP2" = "KB4538484"
}
#endregion

#region Match KB to current OS. If none of the three, then fail the script.
$OSName = (Get-WmiObject -class Win32_OperatingSystem -Property Caption).Caption
if ($osName.Contains("2008") -and -not $osName.Contains("R2")) {
    $ESUPKGKB = $OSESULicensingPreparationPackage.'Server 2008 SP2'
} elseif ($osName.Contains("Windows 7")) {
    $ESUPKGKB = $OSESULicensingPreparationPackage.'Windows 7 SP1'
} elseif ($osName.Contains("2008 R2")) {
    $ESUPKGKB = $OSESULicensingPreparationPackage.'Server 2008 R2 SP1'
} else {
    Exit 1
}
#endregion

#region Check if the KB is installed and return $true, otherwise $false
$hotfixInstalled = Get-WmiObject -Namespace "root\cimv2" -Class win32_quickfixengineering -Property HotFixID | Where-Object { $_.HotFixID -match $ESUPKGKB }
if ($hotfixInstalled) { return $true } else { return $false }
#endregion
```

2) You'll need to figure out what your ESU MAK keys are. Edit the lines below with your personal key.

```powershell
#region Define the ESU MAK Keys
$MAKKeys = @{
    "Win7"           = "*****-*****-*****-*****-*****" # ESU MAK Key Windows 7
    "Server2008OrR2" = "*****-*****-*****-*****-*****" # ESU MAK Key Server 2008/2008R2
}
#endregion

#region Match MAK Key to current OS. If none of the three, then fail the script.
$OSName = (Get-WmiObject -class Win32_OperatingSystem -Property Caption).Caption
if ($osName.Contains("2008")) {
    $MAKKey = $MAKKeys.Server2008OrR2
} elseif ($osName.Contains("Windows 7")) {
    $MAKKey = $MAKKeys.Win7
} else {
    Exit 1
}
#endregion

#region Installation
$Path = Get-Command slmgr.vbs | Select-Object -ExpandProperty Source
$IPKStatus = cscript.exe //Nologo $Path /ipk $MAKKey # Install Product Key
$Data = cscript.exe //Nologo $Path /dlv # Display Detailed License Information
$ESUData = $Data.Where({ $_ -match "-ESU-" },"SkipUntil", 3) # Return first three lines of the ESU code
#$Data = $Data | Select-String -Pattern "Activation ID: "| Select-Object -expand line # Filter to line with Activation ID
$ESUActivationID = $ESUData[2] -split ": " | Select-Object -first 1 -skip 1 # Get Activation ID
$ATOStatus = cscript.exe //Nologo $Path /ato $ESUActivationID # Activate Windows using Activation ID

if ($ATOStatus -match "Product activated successfully.") {
    return "Product activated successfully."
} else {
    return "Not activated"
}
#endregion
```

3) Use this script as the Detection Method. It will find identify the section for the ESU and that it is licensed.
If either no ESU license section nor Licensed status, then return no information. This reports back "Not Installed" to ConfigMgr.

```powershell
#region Validation
$Path = Get-Command slmgr.vbs | Select-Object -ExpandProperty Source
$Validation = cscript.exe //Nologo $Path /dlv # Display Detailed License Information
$ESUData = $Validation.Where({ $_ -match "-ESU-" },"SkipUntil", 12) # Return 12 lines of the ESU code
$ESUValidation = $ESUData | Select-String -Pattern "License Status: " | Select-Object -expand line # Filter to line with License Status
$ESULicenseStatus = $ESUValidation -split ": " | Select-Object -first 1 -skip 1 # Get License Status
If ($ESULicenseStatus -eq "Licensed") {
    Write-Host "ESU License is installed and activated."
    Exit 0
} Else {
    # Empty result means Application detection state = "Not installed"
    #Write-Error "ESU license error"
}
#endregion
```
