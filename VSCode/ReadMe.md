# Configuring PSAppDeployToolkit for Intellisense in Visual Studio Code

1. Install the PowerShell Extension for VS Code
	1. https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell
1. Set the default language to PowerShell (optional)
	1. Under Settings (Ctrl+,) find Files: Default Language
	1. Set this to powershell
	1. Close Settings
1. Download PSAppDeployToolkit
	1. https://github.com/PSAppDeployToolkit/PSAppDeployToolkit/releases
	1. Extract the contents to a folder
 	1. Copy the .\Toolkit\AppDeployToolkit folder to a path of your choosing (such as these options)
    1. $Home\Documents\WindowsPowerShell\Modules
		1. $Env:ProgramFiles\WindowsPowerShell\Modules
	1. Mine is copied to: **C:\Users\user\Documents\WindowsPowerShell\Modules\AppDeployToolkit**
1. Convert the script to a module
	1. In ISE: Open **\Documents\WindowsPowerShell\Modules\AppDeployToolkit\AppDeployToolkitMain.ps1**
	1. File > Save As
		1. Change type to **PowerShell Script Module**
		1. Change name to: **AppDeployToolkit.psm1**
		1. Don't forget to remove  ...Main... from the filename.
	1. Delete AppDeployToolkitMain.ps1, it is no longer needed
1. Update the PSADT XML
	1. Edit **AppDeployToolkitConfig.xml**
	1. Edit Toolkit_RequireAdmin (line 17) from True to **False**
1. Create a VS Code profile
	1. Launch VS Code and select the **PowerShell Integrated Console** terminal then type in **notepad $profile**
		1. If no profile exists, Notepad will ask to create the file, select Yes
	1. Add a line to import the module:
		1. Import-Module $Home\Documents\WindowsPowerShell\Modules\AppDeployToolkit
	1. I also add a line to clear the host, as I don't want to see the import each time.
		1. Clear-Host
	1. Save **Microsoft.VSCode_profile.ps1** and close Notepad
1. Relaunch VS Code
	1. It will switch to the PowerShell extension, perform the import from the profile, then clear the host.
	1. You can confirm by running: ``` get-help copy-file ```

Credit to Christian Nyhuus for documenting the steps for ISE
> https://www.nyhu.us/powershell/psadt/intellisense-psadt-powershell-ise/
> https://www.reddit.com/r/MDT/comments/5e3n5e/autocomplete_for_psadt_in_powershell_ise/
