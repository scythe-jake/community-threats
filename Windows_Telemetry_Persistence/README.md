# Windows Telemetry Persistence - Threat Emulation Plan 

This threat is based off research completed by TrustedSec that highlights the use of windows telemetry for persistence that works all the way through Windows 11 21H2 : https://www.trustedsec.com/blog/abusing-windows-telemetry-for-persistence/

## Executive Summary
In 2020 a few researchers from TrustedSec outlined a unique method of persistence that leverages Windows Telemetry. Collection of telemetry, or diagnostic data, is built into the operating system and used by Microsoft for a myriad of purposes. For example, Microsoft uses this data to identify security issues, improve reliability, analyze/fix software problems, and improve quality or design decisions for future releases. 

To briefly summarize the persistence method discovered by TrustedSec, Windows comes with an executable C:\Windows\System32\CompatTelRunner.exe that is used to run a variety of telemetry related tasks. This binary was designed to be extensible and relies on the registry to provide instruction on what commands to run. All one needs to do to setup persistence is:

* Create a registry key of any name to: HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\TelemetryController
* Inside this key create a Reg_SZ value of “Command” and set it’s data value to the .exe file you would like to run
* Create DWORD keys for Maintenance, Nightly, and Oobe and set them each equal to 1 (the Nightly key alone is enough for the executable to run once every 24 hours)

After this, the specified .exe file will run periodically from a Windows scheduled task as SYSTEM.


### Campaign Profile
The campaign will download a benign executable (benign.exe), create the registry keys, and then manually trigger the scheduled task to run benign.exe. The benign.exe provided doesn’t take any observable action on the system, other than writing to the debug console. We use this at SCYTHE to provide opportunities for detection engineering based on successful process execution alone. This allows teams to focus on the parent/child relationship of processes for detection, rather than incorrectly anchoring on the actions performed by the child process (such as writing a registry value or a file).

Take note that the last command is simply just to manually trigger the scheduled task rather than wait for it to occur. 

### Emulate with SCYTHE
**Note this threat will need to be run with administrative privileges.

1. Download and import the threat in JSON format to your SCYTHE instance 
2. Create a new Windows campaign, selecting HTTPS, and configure your HTTPS communication options.
3. Import from Existing Threat: Windows Telemetry Persistence
4. Launch Campaign
5. Download the EXE and transfer to your test device/VM
6. Right click and "Run as administrator"

**Clean up steps are included after a 5 minute delay to remove the benign.exe file and the created registry keys.