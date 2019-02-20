# Microsoft Dynamics NAV Javascript Control Add-In Template
Control Add-In Template for Visual Studio.

## Description

<p align="center">
    <img src="https://github.com/setrange/NAVJSControlAddIn/blob/master/Microsoft%20Dynamics%20NAV%20Objects/NAVView.png">
</p>

## Installation
1. Install objects from "Microsoft Dynamics NAV Objects" folder in Dynamics NAV. Codeunit "Control Add-In Management" in ControlAddInManagement.fob contain RegisterJavaScriptAddInFromBase64 function that automatical deploy Control Add-In from Build process of the Visual Studio.
2. Edit "Post-build event command line" in Project properties. This script deploy controladdin.dll, restart Dynamics NAV server and load javascript.zip to control add-in record of the NAV.
```ruby
call "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\Tools\VsDevCmd.bat" > NUL

echo Build NAVControlAddIn.zip

set res=$(ProjectDir)Resource\JavascriptControlAddIn.zip

echo %res%

if exist "%res%" del "%res%"

set zip1=$(ProjectDir)Build Tools\7z.exe
set zip2=$(ProjectDir)Resource\*.*

echo %zip1%
echo %zip2%

"$(ProjectDir)Build Tools\7z.exe" a -tzip -r "%res%" "$(ProjectDir)Resource\*.*" > NUL

echo Register Add-In in Microsoft Dynamics NAV 
powershell -ExecutionPolicy unrestricted -command "&'$(ProjectDir)Build Tools\ImportResource.ps1' -Folder '$(ProjectDir)'"

if $(ConfigurationName) == Resource goto Resource

sn -T "$(TargetPath)"

echo copy files to localpath
copy "$(TargetPath)" "C:\Program Files (x86)\Microsoft Dynamics NAV\90\RoleTailored Client\Add-ins" 

echo copy files to server
copy "$(TargetPath)" "C:\Program Files\Microsoft Dynamics NAV\90\Service\Add-ins" 

echo restart Dynamics NAV 2016
echo stop
net stop MicrosoftDynamicsNAVServer$DynamicsNAV90

echo 
echo start
net start MicrosoftDynamicsNAVServer$DynamicsNAV90
```
3. Edit InportResource.ps1. This powershell script import zip file to control add-in record in the Dynamics NAV. Change sn = dcce7894fd66d083 to sn key from output window.
```Ruby
Param(
	[string]$Folder
)

Import-Module 'C:\Program Files\Microsoft Dynamics NAV\90\Service\Microsoft.Dynamics.Nav.Management.dll'

Function RegisterClientAddIn
{
    Param(
		[String]$AddIn,
		[String]$Source
    )

	$arg = "$AddIn"
	if ($Source -ne "")
	{
		$arg = "$arg;$([System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes($Source)))"
	}
	Invoke-NAVCodeunit -ServerInstance DynamicsNAV90 -CodeunitId 99999 -MethodName RegisterJavaScriptAddInFromBase64 -Argument "$arg"
}

RegisterClientAddIn -AddIn "JavascriptControlAddIn;dcce7894fd66d083;1.0.0.0;NAV Control Add-In Template" -Source "$($Folder)Resource\JavascriptControlAddIn.zip"

```
## How it works
Tipical arhitecture of the Javascript Control Add-In.
1. Mainfest.xml - description of control add-in.
2. controladdin.dll - interface description.
<p align="center">
    <img src="https://github.com/setrange/NAVJSControlAddIn/blob/master/Microsoft%20Dynamics%20NAV%20Objects/SchemeJSAddin.png">
</p>

## Links
Autodeploy idea - http://vjeko.com/deploy-your-resource-automatically-from-visual-studio/
