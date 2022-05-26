This threat is explained further in SCYTHE's Threat Thursday blog: https://www.scythe.io/library/threatthursday-evil-corp

## Automated Emulation with SCYTHE

1. Download and import the threat in JSON format to your SCYTHE instance
2. Go to the Threat Catalog and select "WastedLocker"
3. Click "Create Campaign from Threat"
4. Name the Campaign
5. Ensure the HTTPS paramaters are correct for your SCYTHE instance
6. Launch the Campaign

## Manual Emulation with SCYTHE
Start a campaign and execute the following steps in a shell:
```
Start (with https, loader, and controller)
loader --load run
loader --load file
run cmd /c mkdir "%USERPROFILE%\Desktop\EvilCorp"
file --create --path "%USERPROFILE%\Desktop\EvilCorp\important_files.docx" --size 5MB --count 50
STEP = IMPACT
loader --load crypt
crypt --target "%USERPROFILE%\Desktop\EvilCorp\" --encrypt --password "3v1lC0rP" --erase --recurse --extension wasted
loader --load downloader
downloader --src "https://pastebin.com/raw/ZyVJEYB4" --dest "%USERPROFILE%\Desktop\wasted_info.txt"
run powershell notepad "%USERPROFILE%\Desktop\wasted_info.txt"
STEP = CLEAN UP
run powershell Remove-Item -Recurse -Force "%USERPROFILE%\Desktop\EvilCorp"
run powershell del "%USERPROFILE%\Desktop\wasted_info.txt"
controller --shutdown
```