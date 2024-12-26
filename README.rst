VCT Small ARM BSP startup
=========================

This is mainly a set of tools to (re)create OE build environments.

This first release uses OpenEmbedded metadata layers on ``mickledore``
branches and a KAS_ build configuration.

.. _KAS: https://kas.readthedocs.io/en/latest/command-line.html

References and examples
-----------------------

* https://docs.yoctoproject.org/
* http://meta-raspberrypi.readthedocs.io/en/latest/
* https://www.beagleboard.org/projects/yocto-on-beaglebone-black
* https://github.com/VCTLabs/vct-enclustra-bsp-platform


Tox workflows
-------------

Tox_ automation and virtual environment management is the basis for dev
workflow standardization and repeatability. For a more "general" tutorial
on Tox automation, see the following YouTube video
`Automating Build, Test and Release Workflows with tox`_.

.. _Tox: https://tox.wiki/en/4.21.0/
.. _Automating Build, Test and Release Workflows with tox: https://www.youtube.com/watch?v=PrAyvH-tm8E


Old quick start steps:

* cd somewhere
* clone this repo - default branch is now ``oe-mickledore``
* cd repo/
* create .venv with ``tox -e dev``
* view/edit the KAS configuration file ``layers/meta-small-arm-extra/kas/oe/base.yaml``

Defaults are currently the rpi3-64 machine with both a console image on sysvinit
and an xorg image on systemd as default build configs. User build knobs include:

1. the desired machine key, eg: ``raspberrypi3-64``
2. the desired image target key, eg ``rpi-test-image``
3. the package feed environment variables ``PACKAGE_FEED_IP_PORT`` and
   ``PACKAGE_FEED_TYPE``

* check the contents of ``build/conf/local.conf`` and ``build/conf/bblayers.conf``
  and adjust as needed

* run ``tox -e sdcard`` to build the rpi test image

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
   check/set the desired value for the ``UBOOT_CONFIG`` key

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

