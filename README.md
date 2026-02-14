# edge2
Repo for info on khadas edge2 single-board-computer uses SoC: RK3588S (NOT the tiny upgrade RK3588S2 https://dl.khadas.com/products/edge2/specs/documentation-of-changes-from-edge2-v12-to-v13.pdf )

So I have khadas edge v12, because vendor id is:
```
# sudo lsusb -vvv
Bus 003 Device 016: ID 2207:350b Fuzhou Rockchip Electronics Company 
Negotiated speed: High Speed (480Mbps)
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0 [unknown]
  bDeviceSubClass         0 [unknown]
  bDeviceProtocol         0 
  bMaxPacketSize0        64
  idVendor           0x2207 Fuzhou Rockchip Electronics Company
  idProduct          0x350b 
```

And source code of rockchip-burn.sh tools from khadas says (where edge 2l is the newer version):
```
# edge2l
Bus 007 Device 033: ID 2207:350e Fuzhou Rockchip Electronics Company

# edge2
Bus 007 Device 035: ID 2207:350b Fuzhou Rockchip Electronics Company
```


Ubuntu has the best video acceleration (i.e. actually watching youtube videos normally), on BreadOS (arch based) playback stutters.


# Low-level

### Booting

Useful links:
- https://github.com/edk2-porting/edk2-rk3588?tab=readme-ov-file#supported-oses
- https://wiki.bredos.org/en/install/Installation-of-UEFI
- https://opensource.rock-chips.com/wiki_Boot_option
- https://forum.khadas.com/t/guide-arch-linux-for-single-board-computers/21369

Edge has two memory storages, one is SPI Flash for default oowow and one is eMMC 5.1 64GB for everything else.
see: https://dl.khadas.com/products/edge2/specs/edge2-specs.pdf

No UEFI on the board.

### Flashing

Flashing SPI/eMMC is a mystery now, but some commands I used:
- 52520  2025-11-27 23:10  docker pull temach/edge2-rockchip-burn:ubuntu-v22-rockchip4
- 52496  2025-11-27 00:50  less write_latest_oowow_into_spi_flash
- 52497  2025-11-27 00:50  curl dl.khadas.com/online/rockchip-burn > rockchip-burn.sh

```
# less write_latest_oowow_into_spi_flash
curl dl.khadas.com/online/rockchip-burn | sh -s - oowow --refresh --write --spi
```

My question about flashing oowow with instructions: https://forum.khadas.com/t/oowow-update-with-new-version/26167/2

The commands for eMMC and SPI with rockchip-burn-v0.144.sh:
```
# rockchip-burn oowow --write --no-reset https://dl.khadas.com/products/oowow/system/versions/edge2/edge2-oowow-250801.000-sd.img.gz
# rockchip-burn oowow --write --no-reset --spi https://dl.khadas.com/products/oowow/system/versions/edge2/edge2-oowow-250801.000-spi.img.gz
```

### Images

Ubuntu: 
- Images for USB Flash Tool and for oowow : https://dl.khadas.com/products/edge2/firmware/ubuntu/ 
- Images for oowow: https://dl.khadas.com/products/oowow/images/edge2/

The two directories above are more or less kept up to date with each other.

BreadOS: https://dl.khadas.com/products/oowow/images/bredos/edge2/


### Swapping rootfs

Undo previous:

```
sudo umount /mnt/breados/boot 
sudo umount /mnt/breados/
sudo losetup -d /dev/loop0
```

Redo:
```
~/archlinux/khadas-edge2 (main)$ sudo losetup -Pf --show edge2-bredos-arm-cinnamon-v20250813.img
/dev/loop0
losetup: edge2-bredos-arm-cinnamon-v20250813.img: Warning: file does not end on a 512-byte sector boundary; the remaining end of the file will be ignored.

~/archlinux/khadas-edge2 (main)$ lsblk -f 
NAME        FSTYPE FSVER LABEL      UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
loop0                                                                                   
├─loop0p1   vfat   FAT32 BOOT       771E-137D                                           
└─loop0p2   btrfs        ROOTFS     a7e36363-fa76-4aaa-9618-79e45e1a0dc3                
nvme0n1                                                                                 
├─nvme0n1p1 vfat   FAT32 SYSTEM_DRV 0E39-FAAA                             132.7M    48% /boot
├─nvme0n1p2                                                                             
├─nvme0n1p3 ntfs                    92BA84C2BA84A477                                    
├─nvme0n1p4 ext4   1.0              e62afd4b-807f-45de-a445-fb6575d69b78  181.6G    64% /
├─nvme0n1p5 ntfs         windows    B67E86DB7E8693B1                                    
├─nvme0n1p6 ntfs         WINRE_DRV  96743AD5743AB837                                    
└─nvme0n1p7 ext4   1.0              fb9211ee-dd1f-486d-86c0-25484ee5bf8b


~/archlinux/khadas-edge2 (main)$ sudo mkdir -p /mnt/breados

~/archlinux/khadas-edge2 (main)$ sudo mount -t btrfs /dev/loop0p2 /mnt/breados

~/archlinux/khadas-edge2 (main)$ lsblk -f
NAME        FSTYPE FSVER LABEL      UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
loop0                                                                                   
├─loop0p1   vfat   FAT32 BOOT       771E-137D                                           
└─loop0p2   btrfs        ROOTFS     a7e36363-fa76-4aaa-9618-79e45e1a0dc3    1.6G    58% /mnt/breados
nvme0n1                                                                                 
├─nvme0n1p1 vfat   FAT32 SYSTEM_DRV 0E39-FAAA                             132.7M    48% /boot
├─nvme0n1p2                                                                             
├─nvme0n1p3 ntfs                    92BA84C2BA84A477                                    
├─nvme0n1p4 ext4   1.0              e62afd4b-807f-45de-a445-fb6575d69b78  181.6G    64% /
├─nvme0n1p5 ntfs         windows    B67E86DB7E8693B1                                    
├─nvme0n1p6 ntfs         WINRE_DRV  96743AD5743AB837                                    
└─nvme0n1p7 ext4   1.0              fb9211ee-dd1f-486d-86c0-25484ee5bf8b                

~/archlinux/khadas-edge2 (main)$ sudo btrfs subvolume list /mnt/breados   
ID 256 gen 29 top level 5 path @
ID 257 gen 10 top level 5 path @home
ID 258 gen 9 top level 5 path @log
ID 259 gen 9 top level 5 path @pkg
ID 260 gen 9 top level 5 path @.snapshots

~/archlinux/khadas-edge2 (main)$ sudo umount /mnt/breados

~/archlinux/khadas-edge2 (main)$ sudo mount -t btrfs -o subvol=@ /dev/loop0p2 /mnt/breados

~/archlinux/khadas-edge2 (main)$ mount | grep bre
/dev/loop0p2 on /mnt/breados type btrfs (rw,relatime,ssd,discard=async,space_cache=v2,subvolid=256,subvol=/@)

~/archlinux/khadas-edge2 (main)$ sudo mkdir -p /mnt/breados/boot

~/archlinux/khadas-edge2 (main)$ sudo mount /dev/loop0p1 /mnt/breados/boot

/mnt$ mount | grep bread
/dev/loop0p2 on /mnt/breados type btrfs (rw,relatime,ssd,discard=async,space_cache=v2,subvolid=256,subvol=/@)
/dev/loop0p1 on /mnt/breados/boot type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro)

```
