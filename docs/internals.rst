.. _internals:
.. default-role:: literal

Notes on the Internals of Rootfs
================================


Files in `scripts` directory
----------------------------
Most of the files used to coordinate the operation of rootfs are in the
`scripts` directory organised by function.

Make format files
~~~~~~~~~~~~~~~~~
Files in make format are named in upper case.

`ALL_PACKAGES`
    Make file invoked by `rootfs all`.  Simply invokes `PACKAGE_MAKE` for all
    configured packages.  Includes `COMMON` and `TARGET_COMMON`.

`COMMON`
    Make definitions common to all rootfs make files.  Sets up core rootfs path
    definitions: `ROOTFS_ROOT`, `SOURCE_ROOT`, `TAR_DIRS` and toolkit
    definitions and includes the selected `CONFIG` file.  Also defines a number
    of helpful make "functions":

    `SAFE_QUOTE`
        Invoked as `$(call SAFE_QUOTE,string to expand)`, this puts shell quotes
        around make text.  Designed for passing a single value to a shell
        script.

    `EXPORT`
        Invoked as `$(call EXPORT,list of symbols)`, this converts the list of
        symbols into a list of the form '<symbol-name>'`=`'<quoted-value>'.
        Designed for passing a list of symbols to a shell script, for example::

            target:
                    $(call EXPORT,A B C) script

        This invokes `script` with symbols `A`, `B`, `C` exported with their
        current make file values.

    `COMPUTE_PATH`
        Used internally for symbols which can be specified with or without a
        path component, such as `TARGET` and `PACKAGE`.

`FAKEROOT_COMMON`
    Definitions shared in common between `FAKEROOT_MAKE` which assembles the
    final rootfs directory tree and `PACKAGE_INSTALL` which installs individual
    packages into the target tree.

`FAKEROOT_MAKE`

`PACKAGE_COMMON`, `PACKAGE_INSTALL`
    `PACKAGE_INSTALL` is invoked to install a single package and pulls in the
    following definitions in turn: `COMMON`, `FAKEROOT_COMMON`, `PACKAGE_COMMON`
    and finally the package `CONFIG`.

`PACKAGE_MAKE`, `PACKAGE_RULES`
    `PACKAGE_MAKE` is invoked to build a single package, and `PACKAGE_RULES`
    contains the actual rule definitions used in the build: this separation is
    just a programming convenience.  These files, together with `TARGET_COMMON`,
    define the complete build environment seen when building a target package.

`ROOTFS_MAKE`
    Coordinates building of rootfs.

`TARGET_COMMON`
    Make definitions common to all rootfs make files except for those involved
    in building the toolkit: requires a `TARGET` symbol to be defined.  Defines
    `TARGET_ROOT` and `OBJECT_ROOT` and pulls in target specific definitions.

`TOOLKIT_DEFS`, `TOOLKIT_MAKE`
    `TOOLKIT_MAKE` is invoked to build the rootfs toolkit dependencies, and
    `TOOLKIT_DEFS` contains the individual toolkit component definitions.


Supporting Scripts
~~~~~~~~~~~~~~~~~~

The following scripts are mostly used during the assembly of the final rootfs.
Several scripts are designed only to be called directly from make files with
one or more of the following environment variables exported: `sysroot`,
`COMPILER_PREFIX`, `BINUTILS_DIR`, `LIB_PREFIX`.

`crypt`
    Creates encoded password string suitable for use in `/etc/passwd` or
    `/etc/shadow`, used by `useradd` to create specified user.

`extract-tar`
    Searches for a source package with the given name and with the specified md5
    checksum and extracts it.  Used by `PACKAGE_RULES` to implement the `untar`
    target.

`first-time`
    Configures initialisation only scripts.

`functions`
    A handful of common functions shared among these scripts.

`groupadd`
    Adds specified group to rootfs.

`install`
    Installs files in rootfs.

`install-files`
    Used to batch install files and device nodes in the rootfs.

`ldconfig`
    Manages installation or configuration of `ldconfig`.  Long and annoying
    story here.

`populate`
    Populates libraries on the target rootfs.

`startup`
    Configures startup scripts.

`useradd`
    Adds specified user to rootfs.



Include Dependencies
--------------------

::

    COMMON:
        -include $(CONFIG)

    TARGET_COMMON:
        include $(configdir)/PACKAGES

    SYSROOT_COMMON:
        include $(makefiles)/TARGET_COMMON

    ALL_PACKAGES:
        include $(TOP)/scripts/makefiles/COMMON
        include $(makefiles)/TARGET_COMMON

    PACKAGE_COMMON:

    PACKAGE_RULES:

    TOOLKIT_DEFS:

    PACKAGE_INSTALL:
        include $(TOP)/scripts/makefiles/COMMON
        include $(makefiles)/SYSROOT_COMMON
        include $(makefiles)/PACKAGE_COMMON
        include $(packagedir)/CONFIG

    PACKAGE_MAKE:
        include $(TOP)/scripts/makefiles/COMMON
        include $(makefiles)/TARGET_COMMON
        include $(makefiles)/PACKAGE_COMMON
        include $(packagedir)/CONFIG
        include $(makefiles)/PACKAGE_RULES

    ROOTFS_MAKE:
        include $(TOP)/scripts/makefiles/COMMON
        include $(makefiles)/TARGET_COMMON

    SYSROOT_MAKE:
        include $(TOP)/scripts/makefiles/COMMON
        include $(makefiles)/SYSROOT_COMMON

    TOOLKIT_MAKE:
        include $(TOP)/scripts/makefiles/COMMON
        include $(makefiles)/TOOLKIT_DEFS
