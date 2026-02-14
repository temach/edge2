# edge2
Repo for info on khadas edge2 single-board-computer uses SoC: RK3588S (or tiny upgrade RK3588S2 https://dl.khadas.com/products/edge2/specs/documentation-of-changes-from-edge2-v12-to-v13.pdf )

Ubuntu has the best video acceleration (i.e. actually watching youtube videos normally), on BreadOS (arch based) playback stutters.


# Booting

Useful links:
- https://github.com/edk2-porting/edk2-rk3588?tab=readme-ov-file#supported-oses
- https://wiki.bredos.org/en/install/Installation-of-UEFI
- https://opensource.rock-chips.com/wiki_Boot_option

Edge has two memory storages, one is SPI Flash for default oowow and one is eMMC 5.1 64GB for everything else.
see: https://dl.khadas.com/products/edge2/specs/edge2-specs.pdf

No UEFI on the board.

# Flashing

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


