.. _notes:
.. default-role:: literal

General Notes
=============

Booting from an NFS root filesystem
-----------------------------------

Notes on this here:
http://www.faqs.org/docs/Linux-HOWTO/Diskless-root-NFS-other-HOWTO.html#ss4.1 .

Kernel configuration:

* Build in support for the cient 's network card
    (Network device support -> Select your card driver).
* Build in support for the BOOTP protocol
    (Networking options -> IP: kernel level autoconfiguration ->
    IP: BOOTP support).
* Build in support for NFS and root over NFS
    (File systems -> Network File Systems ->
    NFS file system support and File systems -> Network File Systems ->
    NFS file system support -> Root over NFS).
* Build in support for loopback devices
    (Block devices -> Loopback device support).


Building with `-mthumb`
-----------------------

    ==========  ============    =============
    Component   Size change     Notes
    ==========  ============    =============
    bash        23% smaller
    busybox     26% smaller
    dropbear    17% smaller     But too slow!
    mtd-utils
    nano        25% smaller
    ntp         21% smaller
    openntpd    21% smaller
    portmap     7%  smaller
    screen      27% smaller
    strace      16% smaller
    ==========  ============    =============

We don't use `-mthumb` with dropbear: although the size improvement is
worthwhile, the performance reduction is significant: `time ssh $machine true`
reports 1.3 seconds using thumb, 0.5 with standard architecture!


To Do
-----

* Sort out network configuration.  The main problem here is how do we configure
  the IP address?  At the moment this is done by a nasty hack involving the
  u-boot configuration, but this won't keep.

  Also want to use the `/etc/network/interfaces` system for network
  configuration, as it will be better understood.

* Similarly sort out initialisation of the dropbear machine identity.  At the
  moment we're using a canned identity because it's a pain for the machine id to
  change on each reboot!

  Probably want to create a configuration for automatic initialisation on first
  boot.

* Work on powerpc targets, both MVME 5500 and the Xylinx embedded core.

* Wrap up Colibri.

* Work on Libera.  This should be able to use most of the Colibri work, but
  quite a bit more kernel work is likely to be needed.


Running Read-Only
~~~~~~~~~~~~~~~~~

Need to make a decision about read only vs read write.  Here's an interesting
idea: would it be possible to have a read-write sub-mount of a file system
mounted read only?  Difficult.

The main issue is JFFS2: the argument against a dedicated read-write partition
is the small number of erase blocks.  The choices seem to be:

* Forget about it, run read write and stop worrying about it.
* Create a dedicated writeable partition, and worry about wear on the flash
  file system.
* Run read only, but create a process with operates as follows::

    mount -o rw,remount /
    <perform write>
    mount -o ro,remount /

  Tempting...



Wrapping up Colibri
~~~~~~~~~~~~~~~~~~~

To wrap up the Colibri work still need to somehow wrap everything up and
generate a distribution to place on each board.  Actually creating such a
distribution is a bit involved, as the following components are all required:

* Cross compiler toolchain.  This is currently generated using `crosstool-ng`,
  which is a very satisfactory solution.  The following files record the current
  state:

  `$SVN_ROOT/diamond/trunk/tools/crosstool-ng`
    This builds using version 1.2.2 of `crosstool-ng`, and we're currently using
    target `arm-diamond-linux-gnueabi`.

  `/dls_sw/work/common/x-tools/arm-diamond-linux-gnueabi`
    This is where the toolchain above is currently installed.  This probably
    needs to move into a more permanent home, probably in
    `/dls_sw/targetOS/linux-arm`.

  `/dls_sw/prod/common/sources`::
    All the sources required by `crosstool-ng`, as well as all the other builds
    here, are placed here.

* U-boot.  Currently building version 1.1.6 with extensive patches for Colibri
  support and a small patch to ensure sensible operation with the power supply
  controller.  The following files are involved:

  `$SVN_ROOT/diamond/trunk/tools/u-boot`
    This builds the appropriate version of `u-boot`, instructions are in the
    `README` file::

        make TARGET=colibri
        make install TARGET=colibri

    unfortunately the build is not perfectly repeatable (the build date is
    encoded in the `u-boot.bin` file), and the build doesn't have a final home
    as yet.  at present the files below are involved.

  `/scratch/tmp/u-boot/colibri/{src,build}`
    Temporary build directory: this is where the sources are extracted and
    patched and built -- unfortunately, as patching is target specific, it isn't
    practical to share the `src` directory.

  `~mga83/tools/u-boot/u-boot/colibri/{u-boot.bin,tools/mkimage}`::
    This is where the default installation is.  the `mkimage` tool is needed by
    the kernel build and for other loadable images, and the `u-boot.bin` image
    is what needs to be written to devices.

  `pc50:/scratch/git/u-boot/.git`::
    This is the current git repository for our local u-boot development work.
    this is not backed up anywhere.  the two branches currently of interest are
    `colibri` and `xcep`.

* Kernel.

* Rootfs.





Issues Arising
^^^^^^^^^^^^^^

* The `u-boot` build really needs to be exported to the build server somehow.
  After discussion with Nick we'll move the four tools components
  `crosstool-ng`, `u-boot`, `kernel`, `rootfs` to a `targetOS` path and prepare
  a structure in `/dls_sw/targetOS`.
