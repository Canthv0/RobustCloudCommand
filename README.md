# Start-RobustCloudCommand

## Recommended Install
Start-RobustCloudCommand is published to the PowershellGallery and should be installed from there.
https://www.powershellgallery.com/packages/RobustCloudCommand

Install-Module -Name RobustCloudCommand

## Manual Install
If the client doesn't have access to the internet directly then:

* Download the module from https://github.com/Canthv0/RobustCloudCommand/releases
  * Extract the Zip file to a known location
  * From powershell in the extracted path run `Import-Module robustcloudcommand.psd1`
  * Change to a different directory than the one with the psd1 file

## Synopsis
Generic wrapper script that tries to ensure that a script block successfully finishes execution in O365 against a large object count.

Works well with intense operations that may cause throttling

## Description
Wrapper script that tries to ensure that a script block successfully finishes execution in O365 against a large object count.

It accomplishes this by doing the following:
* Monitors the health of the Remote powershell session and restarts it as needed.
* Restarts the session every X number seconds to ensure a valid connection.
* Attempts to work past session related errors and will skip objects that it can't process.
* Attempts to calculate throttle exhaustion and sleep a sufficient time to allow throttle recovery

## Cmdlet Options

Switch | Description|Default
-------|-------|-------
AutomaticThrottle|Value used to calculate time needed for throttle recovery|0.25
IdentifyingProperty|Property on recipient objects to identity the object in the log|"DisplayName","Name","Identity",<br>"PrimarySMTPAddress","Alias","GUID"
LogFile|Location and name of logfile|NA
ManualThrottle|Set number of seconds to delay between loops|None
NonInteractive|Suppresses screen output|False
Recipients|Collection of Objects to operate on|NA
ResetSeconds|Number of seconds between session rebuild|870
ScriptBlock|Operation to run on provided objects|NA
UserPrincipalName|UPN of the user that will be connecting to Exchange online.|NA

## Outputs
Creates the log file specified in -logfile.
Contains a record of all actions taken by the script.

## Examples
invoke-command -scriptblock {Get-mailbox -resultsize unlimited | select-object -property Displayname,PrimarySMTPAddress,Identity} -session (get-pssession) | export-csv c:\temp\mbx.csv

$mbx = import-csv c:\temp\mbx.csv

$cred = get-Credential

.\Start-RobustCloudCommand.ps1 -Credential $cred -recipients $mbx -logfile C:\temp\out.log -ScriptBlock {Set-Clutter -identity $input.PrimarySMTPAddress.tostring() -enable:$false}

Gets all mailboxes from the service returning only Displayname,Identity, and PrimarySMTPAddress.  Exports the results to a CSV
Imports the CSV into a variable
Gets your O365 Credential
Executes the script setting clutter to off using Legacy Credentials

## EXAMPLE
invoke-command -scriptblock {Get-mailbox -resultsize unlimited | select-object -property Displayname,PrimarySMTPAddress,Identity} -session (get-pssession) | export-csv c:\temp\recipients.csv

$recipients = import-csv c:\temp\recipients.csv

Start-RobustCloudCommand -recipients $recipients -logfile C:\temp\out.log -ScriptBlock {Get-MobileDeviceStatistics -mailbox $input.PrimarySMTPAddress.tostring() | Select-Object -Property @{Name = "PrimarySMTPAddress";Expression={$input.PrimarySMTPAddress.tostring()}},DeviceType,LastSuccessSync,FirstSyncTime | Export-Csv c:\temp\stats.csv -Append }

Gets All Recipients and exports them to a CSV (for restart ability)
Imports the CSV into a variable
Executes the script to gather EAS Device statistics and output them to a csv file using ADAL with support for MFA