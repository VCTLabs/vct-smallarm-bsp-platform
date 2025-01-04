VCT Small ARM BSP startup
=========================

This is mainly a set of tools to (re)create OE build environments. Use Tox_
(or your own shell) to run specific build commands in an isolated (Python)
virtual environment. The (shared) tox environment uses several environment
variables to set/adjust various build and/or workflow options and installs
some useful support tools.

This first release uses OpenEmbedded metadata layers on ``mickledore``
branches and a KAS_ build configuration.

.. _KAS: https://kas.readthedocs.io/en/latest/command-line.html

Old quick start steps:

* cd somewhere
* clone this repo - default branch is now ``oe-mickledore``
* cd repo/
* create .venv with ``tox -e dev``
* view/edit the KAS configuration file ``layers/meta-small-arm-extra/kas/oe/base.yaml``

New tox workflows:

Defaults are currently the rpi3-64 machine with both a console image on sysvinit
and an xorg image on systemd as default build configs. User build knobs include:

1. the desired machine key, eg: ``raspberrypi3-64``
2. the desired image target key, eg ``rpi-test-image``
3. the package feed environment variables ``PACKAGE_FEED_IP_PORT`` and
   ``PACKAGE_FEED_TYPE``

* check the contents of ``build/conf/local.conf`` and ``build/conf/bblayers.conf``
  and adjust as needed

* run ``tox -e sdcard`` to build the rpi test image

Dev workflow support
--------------------

In addition to the Yocto build/deploy tasks, we use Tox virtual environments
to facilitate development support for other Yocto-related workflows such as
``tftp`` boot support, ``http`` package feed support, and ``bmap`` copies of
possibly large disk images. Support tools installed via Tox (or just plain
pip) include:

* kas_ - Yocto metadata and build management
* bmaptool_ - The better ``dd`` for embedded projects, based on block maps
* pyserv_ - A collection of simple servers (includes tftp and http server daemons)
* repolite_ - A git repo dependency manager for developers (only used for kas layer)

.. _kas: https://kas.readthedocs.io/en/latest/index.html
.. _bmaptool: https://docs.yoctoproject.org/dev/dev-manual/bmaptool.html
.. _pyserv: https://sarnold.github.io/pyserv/
.. _repolite: https://sarnold.github.io/repolite/

Other dev requirements
----------------------

Some requirements cannot be installed via tox or pip, and must be installed
via OS package manager (eg, ``apt`` or ``emerge``). Some workflows here may
depend on the ``setcap`` tool to provide specific Linux Capabilities (eg, to
run a python tftp server on the default port number expected by u-boot). Note
the default example configuration includes a serial console, so install your
favorite terminal program, eg, picocom_ or putty_.

Follow the `usual Yocto/OE guidance`_ to install build dependencies and add the
appropriate ``libcap`` package to the list of dependencies, eg, ``libcap2-bin``
or ``sys-libs/libcap``. The older `OE and your distro`_ page is also useful.

.. _picocom: https://linux.die.net/man/8/picocom
.. _putty: https://www.instructables.com/How-to-Connect-to-USB-Console-by-Using-PuTTY/
.. _usual Yocto/OE guidance: https://docs.yoctoproject.org/ref-manual/system-requirements.html
.. _OE and your distro: https://www.openembedded.org/wiki/OEandYourDistro


References and examples
-----------------------

* https://docs.yoctoproject.org/
* http://meta-raspberrypi.readthedocs.io/en/latest/
* https://www.beagleboard.org/projects/yocto-on-beaglebone-black
* https://github.com/VCTLabs/vct-enclustra-bsp-platform
* https://github.com/sarnold/u-boot-ATF-manifest


Tox workflows
=============

Tox_ automation and virtual environment management is the basis for dev
workflow standardization and repeatability. For a more "general" tutorial
on Tox automation, see the following YouTube video
`Automating Build, Test and Release Workflows with tox`_.

.. _Tox: https://tox.wiki/en/4.21.0/
.. _Automating Build, Test and Release Workflows with tox: https://www.youtube.com/watch?v=PrAyvH-tm8E

Workflow descriptions
---------------------

The workflow commands described here fall roughly into three categories:

**Workspace workflows**

:sync: clone initial kas config layer in ``.repolite.yml``
:dev: use kas to clone and checkout remaining metadata layers, create virtual
      environment with workflow dependencies.
:clean: Remove ``build/tmp-*`` folder.

**Build workflows**

:sdcard: Build target wic and bmap image files (default is tox IMAGE variable).

**Deployment workflows**

:bmap: Use ``bmaptool`` to burn raw (wic) disk image to an SDCard. Optionally
       apply udev rule to optimize I/O performance.

Big Fat Warning
---------------

.. important:: The above deployment workflows *directly touch* disk devices
               and will *destroy any data* on the ``DISK`` target. Therefore,
               as the workflow user, *you* need to make sure the value
               you provide is the correct ``DISK`` value for your sdcard
               device, eg, ``/dev/mmcblk0`` or ``/dev/sdb``. See below in
               section `Setup micro-SDCard`_ for an example of how to find
               your device name.

Workflow permissions
--------------------

* general Linux development host permissions to install/update host OS packages
* development user added to removable media group, eg, ``disk``
* development user added to ``wheel`` group for polkit rule
* ``libcap`` available for tftp boot workflow support

Workflow support files
----------------------

In terms of development functionality, there is essentially one "support"
file required, that being the kas build config. The main functionality and
development user knobs are contained directly in the parent repo ``tox.ini``
file.

Default options are set as tox environment variables with defaults defining
the yocto build tree and package feeds, machine, and image names::

    IPP = {env:IPP:}               # build server http IP:PORT
    CORE = {env:CORE:oe}           # core metadata (either oe or poky)
    DEPLOY_DIR = {env:DEPLOY_DIR:build/tmp-glibc/deploy/images/{env:MACHINE}}
    DISK = {env:DISK:/dev/mmcblk0}
    IMAGE = {env:IMAGE:rpi-test-image}
    MACHINE = {env:MACHINE:raspberrypi3-64}
    PKGTYPE = {env:PKGTYPE:ipk}


General requirements
====================

* Linux host with yocto build dependencies and tox/libcap packages installed

With at least Python 3.8 and tox installed, clone this repository, then run
the ``sync,dev`` commands to create the yocto build environment. From there,
either use the virtual environment to run kas and/or bitbake commands *or*
run one or more ``tox`` commands to build/deploy specific yocto targets.

Example: install dependencies on Ubuntu build host::

  $ sudo apt-get update
  $ sudo apt-get install gawk wget git diffstat unzip texinfo gcc build-essential \
  chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils \
  iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 \
  xterm python3-subunit mesa-common-dev zstd liblz4-tool libyaml-dev libelf-dev
  $ sudo apt-get install python3-venv python3-distutils tree libgpgme-dev libcap2-bin

On ubuntu 20 or 22, install a newer version of tox into user home::

  $ python3 -m pip install -U pip  # this will install into ~/.local/bin
  $ source ~/.profile
  $ which pip3
  /home/user/.local/bin/pip3
  $ pip3 install tox

Setup micro-SDCard
------------------

We need access to the External Drive to be utilized by the target device.
Run lsblk to help figure out what linux device has been reserved for your
External Drive. To compare state, run ``lsblk`` before inserting the USB
card reader, then run the same command again with the USB device inserted.

Example: for DISK=/dev/sdX

::

  $ lsblk
  NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
  sda      8:0    0 465.8G  0 disk
  ├─sda1   8:1    0   512M  0 part /boot/efi
  └─sda2   8:2    0 465.3G  0 part /                <- Development Machine Root Partition
  sdb      8:16   1   962M  0 disk                  <- microSD/USB Storage Device
  └─sdb1   8:17   1   961M  0 part                  <- microSD/USB Storage Partition

Thus your value is ``DISK=/dev/sdb``

Example: for DISK=/dev/mmcblkX

::

  $ lsblk
  NAME      MAJ:MIN   RM   SIZE RO TYPE MOUNTPOINT
  sda         8:0      0 465.8G  0 disk
  ├─sda1      8:1      0   512M  0 part /boot/efi
  └─sda2      8:2      0 465.3G  0 part /                <- Development Machine Root Partition
  mmcblk0     179:0    0   962M  0 disk                  <- microSD/MMC Storage Device
  └─mmcblk0p1 179:1    0   961M  0 part                  <- microSD/MMC Storage Partition

Thus your value is ``DISK=/dev/mmcblk0`` which is the default workflow value
so may be omitted.

Usage
=====

The commands shown below will clone the required yocto layers, then
build and install the python deps for running the workflow commands.
The install results will end up in a tox virtual environment named
``.venv`` which you can activate for manual use as needed.

The tox/kas commands create two directories to contain the yocto metadata
and build outputs, ie, ``layers`` and ``build`` respectively. Note the Kas_
tool treats both these directories as *transitory*, however, development
workflows should include testing yocto changes inside ``build/conf`` as
well as preserving yocto ``downloads`` and ``sstate_cache`` to speed up
builds.

Tox commands
------------

From inside the repository checkout, use  ``tox list`` to view the list of
workflow environment descriptions::

  $ tox list
  default environments:
  dev     -> Create a kas build virtual environment with managed deps
  bmap    -> Burn the wic image to sdcard device (default: /dev/mmcblk0)
  sdcard  -> Build the (wic) sdcard boot target
  sync    -> Install repolite and use it for cloning workflow deps
  do      -> Run a cmd following "--" from the sync .env, e.g. "tox -e do -- repolite --show"

  additional environments:
  changes -> [no description]
  clean   -> [no description]


.. note:: The default DISK value shown below is at least somewhat "safe"
          as it is not likely to be critical on most development hardware.


Also note the primary tox commands given here are order-dependent, eg::

  $ tox -e sync                   # first time setup only
  $ tox -e dev                    # checkout/refresh yocto layers and build config
  $ IPP="192.168.7.122:8080" tox -e sdcard  # USE YOUR BUILD HOST IP:PORT
  # <insert USB card reader or sdcard>
  $ DISK=/dev/sdX tox -e bmap     # USE YOUR SDCARD DEVICE
                                  # REPLACE sdX with your actual sdcard device

Additional Tox environment commands include::

  $ tox -e changes    # generate a changelog
  $ tox -e clean      # clean build artifacts/tmp dir


.. important:: When running tox commands using an existing build tree, it is
               sometimes advisable to run ``tox -e clean`` before (re)building.


Run KAS directly without Tox
============================

1. create a Python virtual environment in this checkout, activate it, and
   install kas:

::

   $ python -m venv .venv
   $ source .venv/bin/activate
   (.venv) $ python -m pip install kas bmaptool

2. clone the "config" layer (where the new kas base.yaml lives):

::

   (.venv) $ mkdir layers
   (.venv) $ git clone https://github.com/VCTLabs/meta-small-arm-extra.git -b mickledore layers/meta-small-arm-extra

3. view/edit the kas file ``layers/meta-small-arm-extra/kas/oe/base.yaml`` and
   check/set the desired values for the package feed keys

4. fetch the required metadata layers and build default devel image:

::

   (.venv) $ kas checkout layers/meta-small-arm-extra/kas/oe/sysvinit.yaml
   (.venv) $ kas build layers/meta-small-arm-extra/kas/oe/sysvinit.yaml


The first command in step 4 above will populate the ``layers`` folder with
the cloned layers and create a build folder creatively named ``build``.

By default all of the downloaded sources and locally created sstate
cache files are also in the ``build`` folder but can be relocated to a
more convenient/shared location by using some `environment variables`_
as shown below; set them before running the ``build`` command::

  (.venv) $ export DL_DIR="${HOME}/shared/downloads"
  (.venv) $ export SSTATE_DIR="${HOME}/shared/oe/sstate-cache"

.. note:: You may need to create the above directories manually before
          starting a new build.

The (yocto) build config files can be found in the usual place in the
``build`` folder, ie::

  (.venv) $ ls build/conf/
  bblayers.conf  local.conf  templateconf.cfg

Note that changes made to the config files inside ``build/conf/`` are only
temporary as Kas treats everything in the build folder as transitory. Any
changes you wish to keep should be migrated to a Kas config file.

.. _environment variables: https://kas.readthedocs.io/en/latest/command-line.html#variables-glossary

.. important:: *Do not* delete the build folder to start a fresh build,
              rather *do* remove ``build/tmp-glibc`` for that very purpose.

The initial build must fetch and build a large number of components, including
several *very* large git repositories, so the first build can take several hours.

When finished, check the results::

    (.venv) $ $ ls -1 build/tmp-glibc/deploy/images/raspberrypi3-64/ | grep rpi-test-image
    rpi-test-image.env
    rpi-test-image-raspberrypi3-64-20241225194341.rootfs.ext3
    rpi-test-image-raspberrypi3-64-20241225194341.rootfs.manifest
    rpi-test-image-raspberrypi3-64-20241225194341.rootfs.tar.bz2
    rpi-test-image-raspberrypi3-64-20241225194341.rootfs.wic
    rpi-test-image-raspberrypi3-64-20241225194341.rootfs.wic.bmap
    rpi-test-image-raspberrypi3-64-20241225194341.rootfs.wic.bz2
    rpi-test-image-raspberrypi3-64-20241225194341.rootfs.wic.xz
    rpi-test-image-raspberrypi3-64-20241225194341.testdata.json
    rpi-test-image-raspberrypi3-64.ext3
    rpi-test-image-raspberrypi3-64.manifest
    rpi-test-image-raspberrypi3-64.tar.bz2
    rpi-test-image-raspberrypi3-64.testdata.json
    rpi-test-image-raspberrypi3-64.wic
    rpi-test-image-raspberrypi3-64.wic.bmap
    rpi-test-image-raspberrypi3-64.wic.bz2
    rpi-test-image-raspberrypi3-64.wic.xz

Since it already has all of the important bits, the main file(s) of interest
in the listing above are the files ending in ``*.wic[.bmap]`` which are
"raw" disk images used to flash MMC devices. Use these to create a bootable
SDCard or USB stick.

In the full file listing of the image deploy directory, many of the items
are symlinks, but mainly there should be some obvious file types:

* yocto build image files
* kernel image, modules, and device tree files
* u-boot image, boot script, and env files



Host Requirements
-----------------

Host Operating System:

This reference build was tested on following operating systems:

* Ubuntu 22.04
* Gentoo

Required Packages:

The following packages are required for building OE/Yocto-based images on Ubuntu::

  libcap2-bin gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat cpio \
  python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git \
  python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 xterm python3-subunit \
  mesa-common-dev zstd liblz4-tool libyaml-dev libelf-dev python3-distutils

