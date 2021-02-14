Warning: This is completely unsupported. Use at your own risk! If you don't understand and accept the risks of test mode, running modified drivers, or trusting hacky patches from people on the internet, then this probably isn't the repo for you!

# nvidia-kvm-patcher-updated

Generic fix to NVIDIA Code 43 on Virtual Machines. This fork just merges a few fixes by the community for convenience.

### Quick Instructions

    1. Start NVIDIA Driver Setup, Exit Before Installing (Unpacks to C:/NVIDIA, you might have to dupe the folder before exit on recent drivers as they seem to insta-nuke it)
    2. Install the appropriate WDK/DDK, See OS Support
    3. If on Windows 7, See Windows 7 Workaround
    4. Enable Test Mode (bcdedit /set testsigning on) and Reboot
    5. Open powershell and run patcher.ps1 C:/NVIDIA/DisplayDriver/Version/Win10_64/International/Display.Driver
    6. Install Driver Through Extracted Installer (In C:/NVIDIA/DisplayDriver/Version)

### Details

So, the story so far:
You have a VM that uses a passed-through NVIDIA graphics card

However, the driver errored out with code 43, or outright blue-screened your VM

This is because NVIDIA "Introduced a Bug" making their driver "Fail" on "Unsupported configurations", such as having a geforce, by "accidentally" detecting the prescence of a hypervisor

### Preferred Alternative for recent libvirt + qemu
```
<domain>
    ...
        <features>
            ...
            <kvm>
                <hidden state='on'/>
            </kvm>
            ...
            <hyperv>
                ...
                <vendor_id state='on' value='whatever'/>
            </hyperv>
            ...
        </features>
    ...
</domain>
```

### Troubleshooting

1. If testsigning fails, make sure you are running Windows 10 x64, and have the WDK Installed
2. Windows will attempt to overwrite the driver using versions from windows update, you will want to blacklists these updates using Microsoft's tool: https://support.microsoft.com/en-us/kb/3073930
3. Having another problem? File an Issue. I would love some feedback on my crappy script.
4. Still getting error 43? Ensure the graphics card is using Message Signaled Interrupts before rebooting: http://forums.guru3d.com/showthread.php?t=378044
5. Still getting error 43 even with MSIs? Some system configurations (hypervisor/hardware) may not be capable of supporting a passed through nvidia card. I have listed some systems that I have personally tested below.

### OS Support

* Windows 7 x64 is supported using the [Windows 7 DDK] (https://www.microsoft.com/en-us/download/details.aspx?id=11800)
* Windows 10 x64 is supported using the [Windows 10 WDK] (https://developer.microsoft.com/en-us/windows/hardware/windows-driver-kit)

### Windows 7 Workaround

For some reason, at least on the test system, signtool in the Windows 7 WDK Post-Dates The Timestamp (possible reverse timezone compensation???). To get around this, remove all instances of the SKSoftware Certificate using mmc (if you have ran the script before), pre-date your clock by 2 days, and execute gencert.ps1 using powershell.

### Tested Working Host Platforms
Tested with a Asus Z170-WS, i7-6700k, and kernel 4.7
* libvirtd 2.3.0 running qemu 2.6.50 using OVMF UEFI, with PCIe ACS Override patch
* xen 4.7 using bios
* retested system 03/29/2019 [Kernel 4.20, libvirtd 5.1.0, virt-manager 2.1.0, qemu 3.0.50, driver 419.67, Windows 10 1804]

Also tested with:
* Hardware: MSI Z370 Gaming Pro Carbon, i7-8700k, GTX 1080 Ti (Windows driver 391.35), Host system: Ubuntu 18.04, Kernel 4.15, libvirtd 4.0.0, qemu 2.11.1 (OVMF UEFI)
* ASRock Z170 Extreme7+, i7-6700k, GTX 980 Ti, and Windows Server 2016 Standard (v1607 build 14393.0), Stock Hyper-V role (Host) running Windows Server 2016 Standard (Guest, same version), Gen2 VM config v8, using Discrete Device Assignment

### Tested Non-Working Host Platforms
* libvirtd 2.3.0 running qemu 2.6.50 using bios

