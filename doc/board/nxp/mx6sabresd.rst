.. SPDX-License-Identifier: GPL-2.0+

mx6sabresd
==========

How to use and build U-Boot on mx6sabresd
-----------------------------------------

The following methods can be used for booting mx6sabresd boards:

1. Booting from SD card

2. Booting from eMMC

3. Booting via Falcon mode (SPL launches the kernel directly)


1. Booting from SD card via SPL
-------------------------------

mx6sabresd_defconfig target supports mx6q/mx6dl/mx6qp sabresd variants.

In order to build it:

.. code-block:: bash

   $ make mx6sabresd_defconfig
   $ make

This will generate the SPL and u-boot-dtb.img binaries.

- Flash the SPL binary into the SD card:

.. code-block:: bash

   $ sudo dd if=SPL of=/dev/sdX bs=1K seek=1 conv=notrunc && sync

- Flash the u-boot-dtb.img binary into the SD card:

.. code-block:: bash

   $ sudo dd if=u-boot-dtb.img of=/dev/sdX bs=1K seek=69 conv=notrunc && sync

2. Booting from eMMC
--------------------

.. code-block:: bash

   $ make mx6sabresd_defconfig
   $ make

This will generate the SPL and u-boot-dtb.img binaries.

- Boot first from SD card as shown in the previous section

In U-Boot change the eMMC partition config::

   => mmc partconf 2 1 0 0

Mount the eMMC in the host PC::

   => ums 0 mmc 2

- Flash SPL and u-boot-dtb.img binaries into the eMMC:

.. code-block:: bash

   $ sudo dd if=SPL of=/dev/sdX bs=1K seek=1 conv=notrunc && sync
   $ sudo dd if=u-boot-dtb.img of=/dev/sdX bs=1K seek=69 conv=notrunc && sync

Set SW6 to eMMC 8-bit boot: 11010110

3. Booting via Falcon mode
--------------------------

.. code-block:: bash

  $ make mx6sabresd_defconfig
  $ make

This will generate the SPL image called SPL and the u-boot-dtb.img.

- Flash the SPL image into the SD card

.. code-block:: bash

   $ sudo dd if=SPL of=/dev/sdX bs=1K seek=1 oflag=sync status=none conv=notrunc && sync

- Flash the u-boot-dtb.img image into the SD card

.. code-block:: bash

   $ sudo dd if=u-boot-dtb.img of=/dev/sdX bs=1K seek=69 oflag=sync status=none conv=notrunc && sync

Create a partition for root file system and extract it there

.. code-block:: bash

   $ sudo tar xvf rootfs.tar.gz -C /media/root

The SD card must have enough space for raw "args" and "kernel".
To configure Falcon mode for the first time, on U-Boot do the following commands:

- Setup the IP server::

   # setenv serverip <server_ip_address>

- Download dtb file::

   # dhcp ${fdt_addr} imx6q-sabresd.dtb

- Download kernel image::

   # dhcp ${loadaddr} uImage

- Write kernel at 2MB offset::

   # mmc write ${loadaddr} 0x1000 0x4000

- Setup kernel bootargs::

   # setenv bootargs "console=ttymxc0,115200 root=/dev/mmcblk1p1 rootfstype=ext4 rootwait quiet rw"

- Prepare args::

   # spl export fdt ${loadaddr} - ${fdt_addr}

- Write args 1MB data (0x800 sectors) to 1MB offset (0x800 sectors)::

   # mmc write 18000000 0x800 0x800

- Press KEY_VOL_UP key, power up the board and then SPL binary will launch the kernel directly.
