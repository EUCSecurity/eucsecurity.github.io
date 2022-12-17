---
layout: post
title: Disable Legacy PowerShell
hide_description: true
sitemap: false
categories: [PowerShell]
---
This script will disable PowerShell v2 (legacy) on your system

```powershell
# Determine where to do the logging
$logPS = "C:\Windows\Temp\disable_posh_legacy.log"
 
# Set the start Date and Time
Write-Verbose "Setting Script Parameters" -Verbose
$StartDTM = (Get-Date)

# Start the transcript for the install
Start-Transcript $LogPS

# Disable Legacy Powershell Versions
try 
{
    Write-Verbose "Disabling Powershell v2" -Verbose
    Disable-WindowsOptionalFeature -Online -FeatureName microsoftwindowspowershellv2
    Write-Verbose "Powershell v2 Disabled" -Verbose
}
catch 
{
    Write-Verbose "There was a problem disabling Powershell v2 - Please check the logs for more information" -Verbose
}

# Stop Logging
Write-Verbose "Stop logging" -Verbose
$EndDTM = (Get-Date)
Write-Verbose "Elapsed Time: $(($EndDTM-$StartDTM).TotalSeconds) Seconds" -Verbose
Write-Verbose "Elapsed Time: $(($EndDTM-$StartDTM).TotalMinutes) Minutes" -Verbose
Stop-Transcript
```