Integration
===========

System configuration
--------------------

Rauc expects the file ``/etc/rauc/system.conf`` to describe the system it runs
on in a way that all relevant information for performing updates and making
decisions are given.

.. note:: For a full reference off the system.conf file, see :ref:`todo`.

Similar to other configuration files used by rauc, the system configuration
uses a key-value syntax (similar to those known from .ini files).

Slot configuration
~~~~~~~~~~~~~~~~~~

The most important step is to describe the slots that rauc should use
when performing updates. Which slots are required and what you have to take
care of when designing you system will be covered in the chapter :ref:`todo`.
This section assumes, you have already decided on a setup and want to describe
it for rauc.

A slot is defined by a slot section. The naming of the section must follow a
simple format: `slot.<slot-class>.<slot-index>` where *slot-class* describes a
group used for redundancy and *slot-index* is the index of the individual slot
starting with 0.
If you have two rootfs slots, for example, one slot section will be named
``[slot.rootfs.0]``, the other will be named ``[slot.rootfs.1]``.
Rauc does not have predefined class names. The only requirement is that the
class names used in the system config match those in the update manifests.

The mandatory settings for each slot are, the ``device`` that holds the
(device) path describing *where* the slot is located, the ``type`` that
provides defines *how* to update the target device, and the ``bootname``
which is the name the bootloader uses to refer to this slot device.

Type
^^^^

A list of common types supported by rauc:

+----------+-------------------------------------------------------------------+
| Type     | Description                                                       |
+----------+-------------------------------------------------------------------+
| raw      | A partition holding no (known) file system. Only raw image copies |
|          | may be performed.                                                 |
+----------+-------------------------------------------------------------------+
| ext4     | A partition holing an ext4 filesystem.                            |
+----------+-------------------------------------------------------------------+
| nand     | A NAND partition.                                                 |
+----------+-------------------------------------------------------------------+
| ubivol   | A NAND partition holding an UBI volume                            |
+----------+-------------------------------------------------------------------+
| ubifs    | A NAND partition holding an UBI volume containing an UBIFS.       |
+----------+-------------------------------------------------------------------+

Yocto
-----

Yocto support for using rauc is provided by the `meta-ptx
<http://git-public.pengutronix.de/?p=meta-ptx.git>`_ layer.

The layer supports building rauc both for the target as well as a host tool.
With the `bundle.bbclass` it provides a mechanism to specify and build bundles
directly with of Yocto.

Target system setup
~~~~~~~~~~~~~~~~~~~

Add the `meta-ptx` layer to your setup::

  git submodule add http://git-public.pengutronix.de/git-public/meta-ptx.git

Add the rauc tool to your image recipe (or package group)::

  IMAGE_INSTALL_append = "rauc"

Append the rauc recipe from your BSP layer (referred to as `meta-your-bsp` in the
following) by creating a ``meta-your-bsp/recipes-core/rauc/rauc_%.bbappend``
with the following content::

  FILESEXTRAPATHS_prepend := "${THISDIR}/files:"
  
  SRC_URI_append := "file://system.conf"

Write a ``system.conf`` for your board and place it in the folder you mentioned
in the recipe (`meta-your-bsp/recipes-core/rauc/files`). This file must provide
a system compatible string to identify your system type, as well as a
definition of all slots in your system. By default, the system configuration
will be placed in `/etc/rauc/system.conf` on your target rootfs.

For a reference of allowed configuration options in system.conf, see `system
configuration file`_.
For a more detailed instruction on how to write a system.conf, see `chapter`_.

Using rauc on the Host system
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The rauc recipe allows to compile and use rauc on your host system.
Having rauc available as a host tool is useful for debugging, testing or for
creating bundles manually.
For the preferred way to creating bundles automatically, see the chapter
`Bundle generation`_. In order to compile rauc for you host system, simply run::

  bitbake rauc-native

This will place a copy of the rauc binary in ``tmp/deploy/tools`` in your
current build folder. To test it, try::

  tmp/deploy/tools/rauc --version

Bundle generation
~~~~~~~~~~~~~~~~~

Bundles can be created either manually by building and using rauc as a native
tool, or by using the ``bundle.bbclass`` that handles most of the basic steps,
automatically.

First, create a bundle recipe in your BSP layer. A possible location for this
could be ``meta-your-pbsp/recipes/core/bundles/update-bundle.bb``.

To create your bundle you first have to inherit the bundle class::

  inherit bundle

To create the manifest file, you may either use the built-in class mechanism,
or provide a custom manifest.

For using the built-in bundle generation, you need to specify some variables:

``RAUC_BUNDLE_COMPATIBLE``
  Sets the compatible string for the bundle. This should match the compatible
  you specified in your ``system.conf`` or, more general, the compatible of the
  target platform you intend to install this bundle on.

``RAUC_BUNDLE_SLOTS``
  Use this to list all slot classes for which bundle should contain images. A
  value of ``"rootfs appfs"`` for example will create a manifest with images
  for two slot classes; rootfs and appfs.

``RAUC_SLOT_<slotclass>``
  For each slot class, set this to the image (recipe) name which build artifact
  you intend to place in it.

``RAUC_SLOT_<slotclass>[type]``
  For each slot class, set this to the *type* of image you intend to place in
  this slot. Possible types are: ``rootfs`` (default), ``kernel``,
  ``bootloader``.

Based on this information, your bundle recipe will build all required
components and generate a bundle from this. The created bundle can be found in
``tmp/deploy/images/<machine>/bundles`` in your build directory.


PTXdist
-------
   * System setup (system conf, keys, ...)
   * Bundle creation

System Boot
-----------
   * Watchdog vs. Confirmation
   * Kernel Command Line: booted slot
   * D-Bus-Service vs. Single Binary
   * Cron

Barebox
-------
   * State/Bootchooser

GRUB
----

   * Grub-Environment
   * Scripting

Backend
-------

Persistent Data
---------------

   * SSH-Keys?
