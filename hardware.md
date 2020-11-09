# Hardware
## VR
### Headset keeps dropping in and out
- Plug headset into USB 3.0
- Bases up high, facing each other, diagonal

## Storage
### Diagnosing a HDD failure as having bad sectors
#### Symptoms
- Programs using that HDD will fuck up out of the blue while you're using them
- Might log you out
- System may fail to start a few times
- Eventually it will let you start it, but it might take a while and the HDD might make scratching noises.
#### Steps
- Do a full backup of the drive using CloneZilla
    - Make sure you pick the "Expert" option and specify `--rescue` in the options. (This will ignore bad blocks)
- I'm pretty sure you shouldn't do the following steps if you're rescuing an SSD.
- On a Linux live CD or another Linux machine, run 
- **Windows/NTFS partitions:** `chkdsk`
- **Linux/ext partitions:**
    - `sudo badblocks -v /dev/sdXY > badsectors.txt` to get a list of bad sectors
    - `sudo e2fsck -l badsectors.txt /dev/sdXY` to fix/remap those sectors from that list generated in the previous step 
        - Replace `e2fsck` with `fsck` for non ext* formats 
    - Source: https://www.tecmint.com/check-linux-hard-disk-bad-sectors-bad-blocks/
        - This is kind of an old method and I think there's a better way with `smartmontools` (`smartctl`)
