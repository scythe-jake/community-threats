# APT41 - Adversary Emulation Plan

This threat is explained further in SCYTHE's Threat Thursday blog: https://www.scythe.io/library/threatthursday-apt41

## Automated Emulation with SCYTHE
1. Download and import the threat in JSON format to your SCYTHE instance.
2. Download the Virtual File System (VFS) files from the VFS folder.
3. Upload the VFS files to your SCYTHE VFS in the following location: VFS:/shared/threats/APT41
4. Click "Create Campaign from Threat"
5. Name the Campaign
6. Restrict campaign to the device or domain where it will execute. This is to emulate APT41 use of Execution Gaurdrails: Environmental Keying (T1480.001)
7. Select the Communication module. Use DNS, HTTP, or HTTPS over default TCP port 53, 80 or 443 respectively - HTTP and HTTPS will be much quicker. However, APT41 uses DNS as well. We recommend using DNS as long-haul C2 only.
8. Start Campaign
