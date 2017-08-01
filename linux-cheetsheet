# Linux Cheat Sheet

Issues:
### Formatting external disks
1. This partition cannot be modified because it contains a partition table; please reinitialize layout of the whole device. (udisks-error-quark, 11)
```
sudo umount /dev/sdc1
sudo dd if=/dev/zero of=/dev/sdc1 bs=4MB
```
Now open `Disks` and format the drive. 

