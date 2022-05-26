# Industroyer2 Adversary Emulation Plan

This threat is based on the UA-CERT reporting for Industroyer2 - https://cert.gov.ua/article/39518 

## Summary from Cyber Threat Intelligence
Details of the malware targeting industrial control systems are not fully known. The threat actor used malware based on a debugging stub distributed by Hex-Rays (the company behind IDA Pro). The threat actor used scheduled tasks to trigger actions and those scheduled tasks are emulated in this plan.

The threat actor appears to be interested in either autologin accounts or cached domain credentials given targeting of the SECURITY hive. It's common to see threat actors go after SYSTEM and SAM hives, but targeting the SECURITY hive is a bit less common. 

Note: These actions were taken by the threat actor after they had elevated privileges on target systems. This campaign is intended to be run as admin.


## Emulate with SCYTHE
**Note this threat, if executed with administrative privileges, will disable services, delete Volume Shadow Copies, delete Windows Event Logs, and modify bootup. For testing purposes, you may want to test in non-production, make an offline backup, and/or or take a snapshot.

1. Download and import the threat in JSON format to your SCYTHE instance 
2. Create a new campaign, selecting HTTPS, and configure your HTTPS communication options.
3. Import from Existing Threat: Industroyer2
4. Launch Campaign
5. Download the EXE and rename it to Industroyer2_64_SCY.exe

## Emulate Manually
Because this campaign uses multiple scheduled tasks and threat actor-provided binaries, it does not lend itself to full manual emulation. However the following commands can be run for emulation purposes.

Open an elevated command line interface and run:
- cmd /c reg save HKLM\SYSTEM C:\Users\Public\sys.reg /y
- cmd /c reg save HKLM\SECURITY C:\Users\Public\sec.reg /y
- cmd /c reg save HKLM\SAM C:\Users\Public\sam.reg /y
- rundll32.exe C:\windows\system32\comsvcs.dll,Minidump <lsass pid> C:\users\public\mem.dmp full

## Cleaning up
The emulation plan creates numerous files and directories that may be cleaned up. The plan intentionally does not delete these files so teams can hunt for artifacts post-execution. A separate json file has been provided for cleanup. This can be combined with the main emulation plan to integrate all steps into a single campaign (optionally providing a delay between the main emulation and cleanup).

#### 1. Remove the following files created as a result of our actions:
* C:\users\public\mem.dmp
* C:\users\public\sam.reg
* C:\users\public\sec.reg
* C:\users\public\sys.reg
* c:\windows\system32\tasks\40115
* c:\windows\system32\tasks\Nezla
* c:\windows\system32\tasks\vatt

#### 2. Remove files we downloaded and do not intend to persist:
* C:\tmp\cdel.exe
* C:\Dell\108_100.exe
* C:\perflogs\40 115.exe
* C:\dell\watt.exe

#### 3. Files we created with the file module:
* C:\users\pa1.pay
* c:\perflogs\pa.pay
* c:\perflogs\40 115.log

#### 4. Nonstandard directories we (may have) created
* C:\dell
* C:\tmp
