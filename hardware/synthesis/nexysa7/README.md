# saxon on nexysa7

## build software

```sh
cd software/standalone/bootloader/
make RISCV_BIN=/opt/riscv/bin/riscv64-unknown-elf- clean all BSP=NexysA7Linux SPINAL_SIM=no
rm -rf build/src
make RISCV_BIN=/opt/riscv/bin/riscv64-unknown-elf- BSP=NexysA7Linux SPINAL_SIM=yes
cd -

cd software/standalone/machineModeSbi/
make RISCV_BIN=/opt/riscv/bin/riscv64-unknown-elf- clean all BSP=NexysA7Linux
cd -

cd ../u-boot/
CROSS_COMPILE=/opt/riscv/bin/riscv64-unknown-elf- make saxon_defconfig
CROSS_COMPILE=/opt/riscv/bin/riscv64-unknown-elf- make
cd -

cd ../buildroot/
make spinal_saxon_default_defconfig
make -j$(nproc)
dtc -O dtb -o output/images/dtb board/spinal/saxon_default/spinal_saxon_ms6p.dts
cd -

cd ../linux/
cp defconfig arch/riscv/configs/
make ARCH=riscv CROSS_COMPILE=/opt/riscv/bin/riscv64-unknown-elf- defconfig
make ARCH=riscv CROSS_COMPILE=/opt/riscv/bin/riscv64-unknown-elf- -j$(nproc)
/opt/riscv/bin/riscv64-unknown-elf-objcopy -O binary vmlinux Image
mkimage -A riscv -O linux -T kernel -C none -a 0x80080000 -e 0x80080000 -n Linux -d Image uImage
```

## storage

SPI flash
 - 0x00000000: fpga 0x00060000 (384K)
 - 0x00060000: sbi 0x00010000 (64K)
 - 0x00070000: u-boot 0x00070000 (448K)

SD card
 - FAT partition mmc 0:1, which contains uImage and dtb
 - EXT4 partition mmc 0:2, which contains rootfs

## memory

OCM
 - 0x20000000: bootloader 0x00000800 (2K)

SDRAM
 - 0x80000000: sbi 0x00004000 (16K)
 - 0x80004000: u-boot-spl 0x0000c000 (48K)
 - 0x80004000: u-boot 0x00070000 (448K) [alternative]
 - 0x80074000: dtb 0x00001000 (48K)
 - 0x80080000: kernel 0x00580000 (5.5M)

## build saxon rtl

```sh
cd SaxonSoc/
cd ext/SpinalHDL
sbt clean
cd -
sbt "runMain saxon.board.digilent.NexysA7Linux"
```

## build and program hw

```sh
cd hardware/synthesis/nexysa7/
source /opt/Xilinx/Vivado/2019.2/settings64.sh
vivado -mode batch -source build.tcl -tclargs "--rebuild"
vivado -mode batch -source build.tcl -tclargs "--flash"
```

## run

On POR the bootloader in ROM is executed. The bootloader initialkize SDRAM and loads sbi and u-boot-spl binaries to SDRAM. Then it jumps to sbi. The sbi initialize CSR and jumps to u-boot-spl. Finally u-boot-spl loads Linux kernel and device tree from SD to SDRAM and boots Linux.

```sh
minicom -D /dev/ttyUSB1

#u-boot
load mmc 0 8007ffc0 uImage
load mmc 0 80074000 dtb
bootm 8007ffc0 - 80074000
```

## debug (optional)

```sh
#terminal 1
openocd/src/openocd -f interface/ftdi/ft2232h_breakout.cfg -c "set CPU0_YAML $PWD/SaxonSoc/cpu0.yaml" -f target/saxon.cfg -s openocd/tcl

#terminal 2
/opt/riscv/bin/riscv64-unknown-elf-gdb SaxonSoc/software/standalone/machineModeSbi/build/machineModeSbi.elf --eval-command "target remote :3333"
load
restore u-boot/spl/u-boot-spl.bin binary 0x80004000
cont
```
