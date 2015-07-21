# Chrome Vs Group Policy

For various reasons many companies have their Group Policy configured to makes sure that Windows machines use Internet Explorer as their default browser. Most developers have to use other browsers for development and testing so this default can cause problems. Even if you are able to change the default to Chrome or Firefox it will get changed back to IE periodically.

Obviously I don't endorse disregarding your corporate security policies. Some employers may view this as a violation of corporate policy, etc. Be smart; use this workaround at your own discretion.

Requirements: 
* Windows 8/8.1
* User must have local admin rights
 
The work around is to create a scheduled task that will update your default browser. Specifically you can set the task to be triggered on Group Policy Success events (8000-8007). If you open task scheduler in Windows and create a new task you can add these event triggers. It should be under **Microsoft-Windows-GroupPolicy/Operational**. You'll want to create a trigger for each event Id 8000 through 8007.

<img src="https://github.com/shiitake/Chrome-Vs-Group-Policy/blob/master/Images/event-triggers.png" />

The task itself will run an VB script which will in turn run a PowerShell script. **Two scripts? That seems dumb!**  It is dumb but it was the only way to avoid any popup windows when the powershell script runs. Even using the *nologo* and *command* options it would still show a brief popup. If you are able to figure out a way around this feel free to update the article.  Create the task action to start a program with the "Program/script" set to **wscript** and put your script location in the "Add arguments" field. 

<img src="https://github.com/shiitake/Chrome-Vs-Group-Policy/blob/master/Images/task-actions.png" />

On the General tab:
* Specify your user account
* Run only when user is logged on
* Run with highest privileges

On the Settings tab:
* Make sure if the task is already running it is set to "Do not start a new instance"
 
Here's what the VB script looks like: 

```vb.net
Dim objShell,objFSO,objFile
Set objShell=CreateObject("WScript.Shell")
Set objFSO=CreateObject("Scripting.FileSystemObject")
 
'enter the path for your PowerShell Script
strPath="C:\Users\shiitake\Documents\MakeChromeDefault\MakeChromeDefault.ps1"
 
If objFSO.FileExists(strPath) Then
'return short path name
set objFile=objFSO.GetFile(strPath)
strCMD="powershell -nologo -command" & Chr(34) & "&{" &_
objFile.ShortPath & "}" & Chr(34)
 
'Uncomment next line for debugging
'WScript.Echo strCMD
 
'use 0 to hide window
objShell.Run strCMD,0
Else
 
'Display error message
WScript.Echo "Failed to find " & strPath
WScript.Quit
End If
```

Here is the PowerShell script:
```PowerShell
Set-ItemProperty -Path HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\http\UserChoice -Name Hash -Value "JlBpKY40AG4="
Set-ItemProperty -Path HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\http\UserChoice -Name ProgId -Value "ChromeHTML"
Set-ItemProperty -Path HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\https\UserChoice -Name Hash -Value "0VTP7o++1Ys="
Set-ItemProperty -Path HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\https\UserChoice -Name ProgId -Value "ChromeHTML"
```

This script updates the registry entries to set Chrome as my default browser and it updates the DNS suffix list to include clickmotive domains. These registry values are specific to Windows 8; if you are using Windows 7 you will want to verify that these are correct. 
