---
layout: default
title: Devices and modules
description: Notes on how Linux handles devices and modules.
nav_order: 12
parent: Linux
grand_parent: Operating systems
permalink: /operating-systems/linux/devices-and-modules
---

<!-- prettier-ignore-start -->

# Devices and Modules
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Device types

There are four main classes of device on Linux:

- Block devices
- Character devices
- Network devices
- Miscellaneous devices

Block devices are addressable by block, and generally support seeking (random access of memory). Examples include hard drives and flash memory. Block devices are accessed with a block device node, and are generally mounted as a filesystem {% cite lkd -l 337 %}.

Character devices are not normally addressable. Instead, they provide access to a stream of characters (bytes). Examples include keyboards and mice. Character devices are usually accessed via a character device node. Applications interact with character devices directly through their device node {% cite lkd -l 337 %}.

Network devices provide access to a network via a physical adapter and a network protocol. Network devices aren't accessed with a device node, instead they are accessed with the socket API {% cite lkd -l 338 %}.

Miscellaneous devices are a simplified form of character devices. They enable a driver author to represent simple devices easily {% cite lkd -l 338 %}.

Pseudo-devices are device drivers that represent virtual devices rather than physical devices. Common pseudo-devices include the kernel random number generator, the null device, and the memory device {% cite lkd -l 338 %}.

## Modules

Modules are a single binary image of subroutines, data, entry, and exit points. The Linux kernel supports dynamic addition and deletion of code from itself at runtime via modules. Modules also allow for the loading of new drivers in response to the hot plugging of new devices {% cite lkd -l 338 %}.

Modules are defined using helper functions from \<linux/module.h\>, below is a simple example of a module:

```c
/*
* hello.c – The Hello, World! Kernel Module
*/
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

/*
* hello_init – the init function, called when the module is loaded.
* Returns zero if successfully loaded, nonzero otherwise.
*/
static int hello_init(void)
{
  printk(KERN_ALERT "I bear a charmed life.\n");
  return 0;
}

/*
* hello_exit – the exit function, called when the module is removed.
*/
static void hello_exit(void)
{
  printk(KERN_ALERT "Out, out, brief candle!\n");
}

module_init(hello_init);
module_exit(hello_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Shakespeare");
MODULE_DESCRIPTION("A Hello, World Module");
```

You can build modules using the kbuild build system. If your work is an official part of Linux, then it can live inside the Linux source tree.

"Compiled modules are installed into /lib/modules/version/kernel/, where each directory under kernel/ corresponds to the module’s location in the kernel source tree". For example, with a kernel version of 2.6.34, a compiled fishing module would live at /lib/modules/2.6.34/kernel/drivers/char/fishing.ko, if it was written in drivers/char/ {% cite lkd -l 342 %}.

You can load modules with the `modprobe` utility {% cite lkd -l 343 %}.

You define module parameters with the `module_param()` macro:

```c
module_param(name, type, perm);
```

`name` is the name of the parameter exposed to the user, and the variable inside your module. `type` is the parameters data type. `perm` specifies permissions of the corresponding file in sysfs {% cite lkd -l 346 %}.

`module_param()` does not declare the variable for you. You must do that before using the macro. Typical use might look like the following:

```c
/* module parameter controlling the capability to allow live bait on the pole */
static int allow_live_bait = 1; /* default to on */
module_param(allow_live_bait, bool, 0644); /* a Boolean type */
```

{% cite lkd -l 346 %}

## The device model

The device model is a single mechanism for representing devices and their topology in the system {% cite lkd -l 348 %}.

The motivation for the device model was to be able to walk devices from the leaves to the roots in order to power devices down in the correct order {% cite lkd -l 349 %}.

### Kobjects

Kernel objects (kobjects) are fundamental parts of the device model.

Kernel objects are represented with the `kobject` struct:

```c
struct kobject {
  const char *name;
  struct list_head entry;
  struct kobject *parent;
  struct kset *kset;
  struct kobj_type *ktype;
  struct sysfs_dirent *sd;
  struct kref kref;
  unsigned int state_initialized:1;
  unsigned int state_in_sysfs:1;
  unsigned int state_add_uevent_sent:1;
  unsigned int state_remove_uevent_sent:1;
  unsigned int uevent_suppress:1;
};
```

`parent` points to the kobject's parent. This is used to build a hierarchy that can be used to power down leaf devices before their parents.

sysfs, the pseudo file system, is simply "a user space filesystem representation of the kobject object hierarchy inside the kernel". `sd` points to a `sysfs_dirent` structure that represents the kobject in sysfs {% cite lkd -l 350 %}.

Kobjects are normally embedded in other structures, like the `cdev` struct:

```c
/* cdev structure - object representing a character device */
struct cdev {
  struct kobject kobj;
  struct module *owner;
  const struct file_operations *ops;
  struct list_head list;
  dev_t dev;
  unsigned int count;
};
```

{% cite lkd -l 350 %}

The embedded kobject enables the `cdev` to be represented in the object hierarchy {% cite lkd -l 350 %}.

### Ktypes

Kobjects are associated with a ktype (kernel object type).

Ktypes are represented with a `kobj_type` struct:

```c
struct kobj_type {
  void (*release)(struct kobject *);
  const struct sysfs_ops *sysfs_ops;
  struct attribute **default_attrs;
};
```

Ktypes describe the behavior for a family of kobjects. `release` points to a function that's called when a kobject's reference count reaches zero. `sysfs_ops` points to a `sysfs_ops` operation table. `default_attrs` points to an array of `attributes` for the kobject {% cite lkd -l 350-1 %}.

### Ksets

Kernel object sets (ksets) are collections of kobjects. Ksets hold all kobjects of a related type, such as all block devices.

The `kset` pointer in a `kobject` points at its associated kset. ksets are represented by the `kset` struct:

```c
struct kset {
  struct list_head list;
  spinlock_t list_lock;
  struct kobject kobj;
  struct kset_uevent_ops *uevent_ops;
};
```

{% cite lkd -l 351 %}

`list` is a linked list of all kobjects in the set, `k_obj` represents the base class for the set, `uevent_ops` is an operation table used for hot plugging of the kobjects in this kset.

### Interrelation of kobjects, ktypes, and ksets

Kobjects, ktypes, and ksets are all interrelated. The key object is the kobject, which must be embedded in other structs in order to be useful. A ktype defines default properties of related kobjects. Kobjects are then grouped into ksets which aggregate together related kobjects. "Related directories—say, perhaps all subdirectories of a given directory—might be in the same kset" {% cite lkd -l 351-2 %}.

You can see a visual representation of the relationship between these parts in the following image:

<figure>
  <img src="{{site.baseurl}}/assets/img/operating-systems/linux/devices-and-modules/kobjects-ksets.svg" alt="">
  <figcaption><h4>Figure: Relation of kobjects, ktypes, and ksets {% cite lkd -l 352 %}</h4></figcaption>
</figure>

### Reference Counts

A kobject's reference count is set to 1 after it is initialized. As long as the reference count is nonzero, the object will exist in memory. Any code that holds a reference to a kobject must first increment the reference count. When the code is finished with the object, it must decrement the reference count. "When the reference count reaches zero, the object can be destroyed and any associated memory freed" {% cite lkd -l 353 %}.

## sysfs

sysfs is an in-memory filesystem that provides a view of the kobject hierarchy.

You can see an example of sysfs:

```
/sys
|-- block
| |-- loop0 -> ../devices/virtual/block/loop0
| |-- md0 -> ../devices/virtual/block/md0
| |-- nbd0 -> ../devices/virtual/block/nbd0
| |-- ram0 -> ../devices/virtual/block/ram0
| `-- xvda -> ../devices/vbd-51712/block/xvda
|-- bus
| |-- platform
| |-- serio
|-- class
| |-- bdi
| |-- block
| |-- input
| |-- mem
| |-- misc
| |-- net
| |-- ppp
| |-- rtc
| |-- tty
| |-- vc
| `-- vtconsole
|-- dev
| |-- block
| `-- char
|-- devices
| |-- console-0
| |-- platform
| |-- system
| |-- vbd-51712
| |-- vbd-51728
| |-- vif-0
| `-- virtual
|-- firmware
|-- fs
| |-- ecryptfs
| |-- ext4
| |-- fuse
| `-- gfs2
|-- kernel
| |-- config
| |-- dlm
| |-- mm
| |-- notes
| |-- uevent_helper
| |-- uevent_seqnum
| `-- uids
`-- module
 |-- ext4
 |-- i8042
 |-- kernel
 |-- keyboard
 |-- mousedev
 |-- nbd
 |-- printk
 |-- psmouse
 |-- sch_htb
 |-- tcp_cubic
 |-- vt
 `-- xt_recent
```

{% cite lkd -l 356 %}

Initialized kobjects are not automatically exported to sysfs. To represent a kobject in sysfs, you use `kobject_add()`:

```c
int kobject_add(struct kobject *kobj, struct kobject *parent, const char *fmt, ...);
```

The location of a kobject in sysfs depends on the kobject's location in the object hierarchy. "If the kobject’s parent pointer is set, the kobject maps to a subdirectory in sysfs inside its parent. If the parent pointer is not set, the kobject maps to a subdirectory inside `kset->kobj`". `fmt` is the name of the directory representing the kobject
in sysfs {% cite lkd -l 357-8 %}.

The `kobj_type` field in kobjects and ksets provides a default set of files used to populate the sysfs directories {% cite lkd -l 358 %}.

The `default_attrs` field on `kobj_type` is an array of `attribute` structs, which map kernel data to files in sysfs. The `attribute` struct is as follows:

```c
/* attribute structure - attributes map kernel data to a sysfs file */
struct attribute {
  const char *name; /* attribute’s name */
  struct module *owner; /* owning module, if any */
  mode_t mode; /* permissions */
};
```

The `name` field will be the filename of the file created from the attribute. `mode` points to the permissions of the file.

The `sysfs_ops` table describes how to use the attributes:

```c
struct sysfs_ops {
  /* method invoked on read of a sysfs file */
  ssize_t (*show) (struct kobject *kobj,
    struct attribute *attr,
    char *buffer);

  /* method invoked on write of a sysfs file */
  ssize_t (*store) (struct kobject *kobj,
    struct attribute *attr,
    const char *buffer,
    size_t size);
};
```

{% cite lkd -l 359 %}

`show()` is invoked when the sysfs entry is read from user space. "It must copy the value of the attribute given by attr into the buffer provided by buffer. The buffer is `PAGE_SIZE` bytes in length; on x86, `PAGE_SIZE` is 4096 bytes. The function should return the size in bytes of data actually written into buffer on success or a negative error code on failure" {% cite lkd -l 359 %}.

`store()` is "invoked on write. It must read the size `bytes` from buffer into the variable represented by the attribute `attr`. The size of the buffer is always `PAGE_SIZE` or smaller. The function should return the size in bytes of data read from buffer on success or a negative error code on failure." {% cite lkd -l 359 %}.

## References

{% bibliography --cited_in_order %}
