# BazarLoader

BazarLoader is used as part of the initial access portion of multiple threat groups. The threat focuses on bypassing defenses and is commonly changing TTPs. In this threat, we cover a number of different examples observed in the wild.

## BazarLoader Excel to WMIC to PowerShell to rundll32.exe
This CTI comes from https://app.any.run/tasks/c5dc698b-8e86-4e50-8b32-d8f45f7538b3/

### BazarLoaderStage0
1. Download and import the threats in JSON format to your SCYTHE instance
2. Create a new campaign `BazarLoaderStage0` with HTTPS
3. Import from Existing Threat: BazarLoaderStage0
4. Modify the process hollowing command to match the location where Microsoft Excel is installed in the target system
5. Launch the Campaign
6. Start Stage 1 and 2 before execution

### BazarLoaderStage1
1. Create a new campaign `BazarLoaderStage1` with HTTPS
2. Import from Existing Threat: BazarLoaderStage1
3. Launch the Campaign
4. Save the 64-bit EXE as `BazarLoaderStage1.exe`
5. Upload to VFS:/shared/threats/BazarLoader/BazarLoaderStage1.exe

### BazarLoaderStage2
1. Create a new campaign `BazarLoaderStage2` with HTTPS
2. Add any automation you would like to execute
3. Launch the Campaign
4. Download the DLL with entry-point set to `setscreen`
5. Save the DLL as `87764675478.dll`
6. Upload the DLL to VFS:/shared/threats/BazarLoader/87764675478.dll

## BazarLoader LOG File in ISO
This CTI comes from https://twitter.com/th3_protoCOL/status/1488600980979552256?s=20&t=QKD0ws_NAZdzjYI8LXmIQA

### BazarLoaderStage1
1. Download and import the BazarLoaderStage1 SCYTHE Threat JSON
2. Create a new Campaign, Select DiavolStage1 in Existing User-Defined Threats, and click Add Steps
3. Start Campaign
4. Select Download, File type DLL, and change the Entry-point function name to spload
5. Download and change the file name to DumpStack.log 
6. Create a new folder and place DumpStack.log into it
7. In the new folder right click navigate to New and select Shortcut
8. In the Type the location of the item paste in “cmd.exe /c xcopy /y DumpStack.log c:\programdata\ && C:\Windows\System32\rundll32.exe C:\programdata\DumpStack.log,spload && exit” and select next
9. Name the shortcut Attachments
10. Once the shortcut is created, right click and select properties
11. In the properties window clear the Start in field so it’s blank and apply the changes
12. Launch the application Folder2Iso available at https://github.com/scythe-io/compound-actions/tree/main/T1553.005%20-%20Mark-of-the-Web%20Bypass/src
13. For the Select Folder field choose the new folder you placed the .log and .lnk files in
14. Click Select Output and enter a name for the Select Output field, we chose Report
15. Click Generate ISO
16. Place the ISO file on the target host system, click the ISO to mount it, and then click the shortcut to execute the campaign