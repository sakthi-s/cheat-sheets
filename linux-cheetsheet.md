# Linux Cheat Sheet

Issues:
## External disk Issues
### Formatting Issues
1. Issue: `This partition cannot be modified because it contains a partition table; please reinitialize layout of the whole device. (udisks-error-quark, 11)`
Solution : - 
```
sudo umount /dev/sdc1
sudo dd if=/dev/zero of=/dev/sdc1 bs=4MB
```
  - Now open `Disks` and format the drive. 

### Mounting Issues
1. Issue: Mount NTFS file system with read write access
```
"Error mounting /dev/sdb1 at /media/fuzzy27/My Book: Command-line `mount -t "ntfs" -o "uhelper=udisks2,nodev,nosuid,uid=1000,gid=1000,dmask=0077,fmask=0177" "/dev/sdb1" "/media/fuzzy27/My Book"' exited with non-zero exit status 13: $MFTMirr does not match $MFT (record 0).
Failed to mount '/dev/sdb1': Input/output error
NTFS is either inconsistent, or there is a hardware fault, or it's a
SoftRAID/FakeRAID hardware. In the first case run chkdsk /f on Windows
then reboot into Windows twice. The usage of the /f parameter is very
important! If the device is a SoftRAID/FakeRAID then first activate
it and mount a different device under the /dev/mapper/ directory, (e.g.
/dev/mapper/nvidia_eahaabcc1). Please see the 'dmraid' documentation
for more details.
```
Solution: 
Not a very efficient one: Open `Disks` and reformat as FAT32. 

