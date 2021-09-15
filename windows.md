# Windows 10
## Python
### pip `[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1108)` error when installing
_[26/3/20]_

Make sure `requests` is installed first via `pip install requests`.

## Broken EFI Partition (On GPT UEFI)
I got into a situation where Windows' bootloader would enter a reboot-loop, and refuse to boot properly.
This will reset Windows' EFI partition that it uses to boot.

### Getting to a command prompt in repair mode

For these steps, you need a Windows 10 Install USB. This is best done on another working Windows 10 install using Microsoft's tool.
You user-agent needs to be Windows for the site to actually let you download the install tool.
For Australians, make sure you have the US version rather than the UK version. Using forward slashes instead of back slashes won't work.

- Create the install USB
- Boot into the USB on the broken machine
- Select `Install` > `Repair your computer` > `Advanced Options` > `Open Command Prompt`

### (Optional) Checking to see what else is on the partition
Modifying the EFI will probably mess up booting to your Linux installs if they are sharing the same EFI partition, so it's good to look.

The EFI partition can be viewed in Linux (installed, or a live USB) by mounting it as you would any other drive and looking inside.

In CMD, from Windows repair USB:
- Open disk part: `diskpart`
- Look at all of your disks: `list disk` 
- Select the disk containing the EFI partition (usualy the one with Windows): `sel disk <number>`
- Look at the partitions: `list part`
- Select the EFI partition: (Normally 100MB in size, sometimes marked with "EFI"): `sel part <number>`
- Mount it: `assign letter=<any unused drive letter>`
- Look inside:
```batch
<drive letter>:
cd <drive letter>:\EFI\
dir
```

### Repairing from CMD
I followed [these](https://www.dell.com/support/kbdoc/en-au/000124331/how-to-repair-the-efi-bootloader-on-a-gpt-hdd-for-windows-7-8-8-1-and-10-on-your-dell-pc#instructions) steps, but substituing step 8 for the command provided in the "Note section."


- Open disk part: `diskpart`
- View all disks: `list disk` 
- Select the disk containing the EFI partition (usualy the one with Windows): `sel disk <number>`
- View all partitions: `list part`
- Select the EFI partition: (Normally 100MB in size, sometimes marked with "EFI"): `sel part <number>`
- Mount it: `assign letter=R` (Can use any unused letter)
- *You can view what is mounted so far with:* `list vol`
- Reset the EFI: `bcdboot C:\Windows /s R: /f UEFI`

It should say if it happened successfully.
The last step will not work if forward slashes are used for paths.
