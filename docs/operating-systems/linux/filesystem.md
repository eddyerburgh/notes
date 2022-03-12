---
layout: default
title: Filesystem
description: Notes on how Linux implements filesystem.
nav_order: 9
parent: Linux
grand_parent: Operating systems
permalink: /operating-systems/linux/filesystem
---

<!-- prettier-ignore-start -->

# Filesystem
{:.no_toc}

The filesystem is a well-known abstraction that is valuable to deeply understand.

This section focusses on the Linux filesystem and how it schedules I/O operations.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## VFS

The VFS (Virtual FileSystem) is a subsystem that implements the file and filesystem-related interfaces. All filesystems rely on the VFS. VFS enables different filesystems on different media to interoperate {% cite lkd -l 261 %}.

Linux implements an abstraction layer around its low-level filesystem interface. Nothing in the kernel needs to understand the details of the filesystems, except the filesystems themselves.

Consider:

```c
ret = write(fd, buf, len);
```

`write()` writes `len` bytes pointed to by `buf` to the current position in the file represented by the file descriptor `fd`. The implementation of `write()` is implemented in the `sys_write()` system call. This then calls the filesystems' write method, which will write the data to the media (or do whatever the filesystem does on write) {% cite lkd -l 262-3 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/operating-systems/linux/filesystem/data-flow.svg" alt="">
  <figcaption><h4>Figure: The flow of data from a `write()` call {% cite lkd -l 263 %}</h4></figcaption>
</figure>

## Unix filesystems

Unix historically provided four filesystem abstractions:

- Files
- Directory entries
- inodes
- Mount points

A **filesystem** is the method and data structures that an OS uses to control how data is stored and received. Filesystems contain files, directories, and associated information. "In Unix, filesystems are mounted at a specific mount point in a global hierarchy known as a namespace". This enables mounted filesystems to appear as a single tree. This is different from the behavior in Windows, which breaks the filesystem into drive letters such as `C:` {% cite lkd -l 263 %}.

A **file** is an ordered string of bytes. The first byte is the beginning of the file and the last byte is the end of the file. Files are assigned human-readable names for identification {% cite lkd -l 263-4 %}.

A directory is an organizational structure that can contain files and subdirectories. In Unix, directories are files that list metadata about the files contained in the directory {% cite lkd -l 264 %}.

Unix systems separate the concept of a file from its metadata, which is stored in a separate data structure: an inode (short for index node).

These parts combine with the file system's control information, which is stored in a superblock. The **superblock** data structure contains information about the entire filesystem, sometimes known as the filesystem metadata {% cite lkd -l 264 %}.

A **mount point** is a directory in a currently accessible filesystem tree in which an additional filesystem is attached.

VFS works based on these concepts, but it doesn't require the filesystem to implement them on-disk. VFS works with non-Unix filesystems like FAT by having the FAT filesystem code generate the data structures for inodes and files in-memory from the data that's stored physically {% cite lkd -l 264 %}.

## VFS Objects

The primary object types of VFS are:

- `superblock`
- `inode`
- `dentry`
- `file`

Each of these primary objects contains an operations object, which describe the methods the kernel can operate against the primary objects:

- `super_operations` contains methods that the kernel can execute on a filesystem, like `write_inode` and `sync_fs`.
- `inode_operations` contains methods the kernel can operate on a file, like `create` and `link`.
- `dentry_operations` contains methods the kernel can invoke on a directory entry, like `d_compare` and `d_delete`.
- `file_operations` contains methods the kernel can invoke on an open file, like `read` and `write`.

The operations objects are implemented as a struct that contains pointers to the functions that can operate on the parent object {% cite lkd -l 265 %}.

Each registered filesystem is represented by the `file_system_type` structure. Each mount point is represented by the `vfsmount` structure {% cite lkd -l 266 %}.

Two per-process structures describe the filesystems and files associated with the process: `fs_struct` and `file`.

## The superblock object

The superblock object contains the filesystem metadata. The superblock normally relates to the filesystem superblock that's stored in a special sector on disk. Filesystems that aren't disk-based generate the superblock at runtime and store it in memory {% cite lkd -l 266 %}.

The superblock is represented by the `super_block` struct:

```c
struct super_block {
  struct list_head s_list; // list of all superblocks
  dev_t s_dev; // identifier
  unsigned long s_blocksize; // block size in bytes
  unsigned char s_blocksize_bits; // block size in bits
  unsigned char s_dirt; // dirty flag
  unsigned long long s_maxbytes; // max file size
  struct file_system_type s_type; // filesystem type
  struct super_operations s_op; // superblock methods
  struct dquot_operations *dq_op; // quota methods
  struct quotactl_ops *s_qcop; // quota control methods
  struct export_operations *s_export_op; // export methods
  unsigned long s_flags; // mount flags
  unsigned long s_magic; // filesystem’s magic number
  struct dentry *s_root; // directory mount point
  struct rw_semaphore s_umount; // unmount semaphore
  struct semaphore s_lock; // superblock semaphore
  int s_count; // superblock ref count
  int s_need_sync; // not-yet-synced flag
  atomic_t s_active; // active reference count
  void *s_security; // security module
  struct xattr_handler **s_xattr; // extended attribute handlers
  struct list_head s_inodes; // list of inodes
  struct list_head s_dirty; // list of dirty inodes
  struct list_head s_io; // list of writebacks
  struct list_head s_more_io; // list of more writeback
  struct hlist_head s_anon; // anonymous dentries
  struct list_head s_files; // list of assigned files
  struct list_head s_dentry_lru; // list of unused dentries
  int s_nr_dentry_unused; // number of dentries on list
  struct block_device *s_bdev; // associated block device
  struct mtd_info *s_mtd; // memory disk information
  struct list_head s_instances; // instances of this fs
  struct quota_info s_dquot; // quota-specific options
  int s_frozen; // frozen status
  wait_queue_head_t s_wait_unfrozen; // wait queue on freeze
  char s_id[32]; // text name
  void *s_fs_info; // filesystem-specific info
  fmode_t s_mode; // mount permissions
  struct semaphore s_vfs_rename_sem; // rename semaphore
  u32 s_time_gran; // granularity of timestamps
  char *s_subtype; // subtype name
  char *s_options; // saved mount options
}
```

The code for creating and managing superblocks is in [fs/super.c](https://elixir.bootlin.com/linux/v2.6.39.4/source/fs/super.c). A superblock is created and initialized with the `alloc_super()` function. When a filesystem is mounted, it reads its superblock off the disk, and fills in its superblock object {% cite lkd -l 267 %}.

## Superblock Operations

`s_op` is a pointer to the superblock operations table, represented by the `super_operations` struct:

```c
struct super_operations {
  struct inode *(*alloc_inode)(struct super_block *sb);
  void (*destroy_inode)(struct inode *);
  void (*dirty_inode) (struct inode *);
  int (*write_inode) (struct inode *, int);
  void (*drop_inode) (struct inode *);
  void (*delete_inode) (struct inode *);
  void (*put_super) (struct super_block *);
  void (*write_super) (struct super_block *);
  int (*sync_fs)(struct super_block *sb, int wait);
  int (*freeze_fs) (struct super_block *);
  int (*unfreeze_fs) (struct super_block *);
  int (*statfs) (struct dentry *, struct kstatfs *);
  int (*remount_fs) (struct super_block *, int *, char *);
  void (*clear_inode) (struct inode *);
  void (*umount_begin) (struct super_block *);
  int (*show_options)(struct seq_file *, struct vfsmount *);
  int (*show_stats)(struct seq_file *, struct vfsmount *);
  ssize_t (*quota_read)(struct super_block *, int, char *, size_t, loff_t);
  ssize_t (*quota_write)(struct super_block *, int, const char *, size_t, loff_t);
  int (*bdev_try_to_free_page)(struct super_block*, struct page*, gfp_t);
}
```

Each field in the structure is a pointer to a function that operates on a superblock object.

Some common methods:

- `alloc_inode()` creates and initializes a new inode object.
- `destroy_inode()` deallocates an inode.
- `write_inode()` writes the inode to disk.
- `put_super()` releases the given superblock object.
- `write_super()` updates the on-disk superblock with the specified superblock.
- `sync_fs()` synchronizes the filesystem metadata with the on-disk filesystem.
- `statfs()` returns filesystem statistics.
- `clear_inode()` clears the specified inode and any pages containing related data.
- `umount_begin()` called by VFS to interrupt a mount operation.

{% cite lkd -l 268-9 %}

## Inodes

An **inode** describes a filesystem object, like a file or a directory. Inodes are constructed in memory by a filesystem when it's mounted.

An inode is represented by the `inode` struct:

```c
struct inode {
  struct hlist_node i_hash; // hash list
  struct list_head i_list; // list of inodes
  struct list_head i_sb_list; // list of superblocks
  struct list_head i_dentry; // list of dentries
  unsigned long i_ino; // inode number
  atomic_t i_count; // reference counter
  unsigned int i_nlink; // number of hard links
  uid_t i_uid; // user id of owner
  gid_t i_gid; // group id of owner
  kdev_t i_rdev; // real device node
  u64 i_version; // versioning number
  loff_t i_size; // file size in bytes
  seqcount_t i_size_seqcount; // serializer for i_size
  struct timespec i_atime; // last access time
  struct timespec i_mtime; // last modify time
  struct timespec i_ctime; // last change time
  unsigned int i_blkbits; // block size in bits
  blkcnt_t i_blocks; // file size in blocks
  unsigned short i_bytes; // bytes consumed
  umode_t i_mode; // access permissions
  spinlock_t i_lock; // spinlock
  struct rw_semaphore i_alloc_sem; // nests inside of i_sem
  struct semaphore i_sem; // inode semaphore
  struct inode_operations *i_op; // inode ops table
  struct file_operations *i_fop; // default inode ops
  struct super_block *i_sb; // associated superblock
  struct file_lock *i_flock; // file lock list
  struct address_space *i_mapping; // associated mapping
  struct address_space i_data; // mapping for device
  struct dquot *i_dquot[MAXQUOTAS]; // disk quotas for inode
  struct list_head i_devices; // list of block devices
  union {
    struct pipe_inode_info *i_pipe; // pipe information
    struct block_device *i_bdev; // block device driver
    struct cdev *i_cdev; // character device driver
  };
  unsigned long i_dnotify_mask; // directory notify mask
  struct dnotify_struct *i_dnotify; // dnotify
  struct list_head inotify_watches; // inotify watches
  struct mutex inotify_mutex; // protects inotify_watches
  unsigned long i_state; // state flags
  unsigned long dirtied_when; // first dirtying time
  unsigned int i_flags; // filesystem flags
  atomic_t i_writecount; // count of writers
  void *i_security; // security module
  void *i_private; // fs private pointer
}
```

The `inode_operations` member points to an object containing the operations that can be performed on an inode:

```c
struct inode_operations {
  int (*create) (struct inode *,struct dentry *,int, struct nameidata *);
  struct dentry * (*lookup) (struct inode *,struct dentry *, struct nameidata *);
  int (*link) (struct dentry *,struct inode *,struct dentry *);
  int (*unlink) (struct inode *,struct dentry *);
  int (*symlink) (struct inode *,struct dentry *,const char *);
  int (*mkdir) (struct inode *,struct dentry *,int);
  int (*rmdir) (struct inode *,struct dentry *);
  int (*mknod) (struct inode *,struct dentry *,int,dev_t);
  int (*rename) (struct inode *, struct dentry *,
  struct inode *, struct dentry *);
  int (*readlink) (struct dentry *, char __user *,int);
  void * (*follow_link) (struct dentry *, struct nameidata *);
  void (*put_link) (struct dentry *, struct nameidata *, void *);
  void (*truncate) (struct inode *);
  int (*permission) (struct inode *, int);
  int (*setattr) (struct dentry *, struct iattr *);
  int (*getattr) (struct vfsmount *mnt, struct dentry *, struct kstat *);
  int (*setxattr) (struct dentry *, const char *,const void *,size_t,int);
  ssize_t (*getxattr) (struct dentry *, const char *, void *, size_t);
  ssize_t (*listxattr) (struct dentry *, char *, size_t);
  int (*removexattr) (struct dentry *, const char *);
  void (*truncate_range)(struct inode *, loff_t, loff_t);
  long (*fallocate)(struct inode *inode, int mode, loff_t offset,
  loff_t len);
  int (*fiemap)(struct inode *, struct fiemap_extent_info *, u64 start,
  u64 len);
};
```

Some of the functions contained by the object are:

- `create()` creates a new inode from a given dentry object.
- `find()` searches a directory for an inode corresponding to the filename specified by the dentry.
- `link()` creates a hardlink between an old dentry with a new dentry.
- `unlink()` removes the inode specified by dentry from the directory `dir`.
- `symlink()` creates a symbolic link to the file represented by dentry.
- `mkdir()` called from the `mkdir` syscall, creates a new directory.
- `rmdir()` called from the `rmdir` syscall, removes a new directory.
- `mknod()` called by the `mknod` syscall, creates a special file (like a socket or a pipe).
- `follow_link()` translates a symbolic link to the inode it points to.
- `truncate()` modifies the size of a given field.
- `permission()` checks whether specified access is allowed for the given inode.

{% cite lkd -l 272-4 %}

## Dentries

A **dentry** (directory entry) represents a specific component in a path.

e.g. in the path `/bin/vi`, `/`, `bin`, and `vi` would be represented by dentry objects. Dentries facilitate path traversal and other operations.

Dentry objects are represented by the `dentry` struct:

```c
struct dentry {
  atomic_t d_count; // usage count
  unsigned int d_flags; // dentry flags
  spinlock_t d_lock; // per-dentry lock
  int d_mounted; // is this a mount point?
  struct inode *d_inode; // associated inode
  struct hlist_node d_hash; // list of hash table entries
  struct dentry *d_parent; // dentry object of parent
  struct qstr d_name; // dentry name
  struct list_head d_lru; // unused list
  union {
    struct list_head d_child; // list of dentries within
    struct rcu_head d_rcu; // RCU locking
  } d_u;
  struct list_head d_subdirs; // subdirectories
  struct list_head d_alias; // list of alias inodes
  unsigned long d_time; // revalidate time
  struct dentry_operations *d_op; // dentry operations table
  struct super_block *d_sb; // superblock of file
  void *d_fsdata; // filesystem-specific data
  unsigned char d_iname[DNAME_INLINE_LEN_MIN]; // short name
};
```

The dentry object doesn't correspond to an on-disk structure. The VFS creates dentry objects at runtime from a string representation of a path name {% cite lkd -l 275 %}.

A dentry object can be in one of three states:

1. Used
2. Unused
3. Negative

A used dentry corresponds to a valid inode (`d_inode` points to an inode). A used dentry object cannot be discarded {% cite lkd -l 276 %}.

An unused dentry object points to a valid inode, but the VFS is not currently using the dentry. The dentry is kept in the dentry cache unless it's needed again {% cite lkd -l 276 %}.

A negative dentry does not point to a valid inode. This is either because the inode was deleted or because it never existed to begin with. A negative dentry is also cached but will be deallocated if free memory is low {% cite lkd -l 276 %}.

The dentry cache (dcache) is used to minimize future work to resolve paths into dentries. It consists of three parts:

1. A used dentries list, linked from their associated inode object.
2. A doubly linked LRU list of unused and negative dentries. When the kernel needs to free memory, it removes entries from the tail.
3. A hash table used to resolve a path into a dentry.

{% cite lkd -l 276 %}

## The file object

The `file` object represents a file opened by a process. `file` is an in-memory representation of an open file. It's created in response to the `open()` system call and is destroyed in response to the `close()` system call {% cite lkd -l 279 %}.

There can be multiple file objects for the same file that exists for different procceses. The object points to a `dentry`, which points to an `inode`. Both objects are unique to a file {% cite lkd -l 279 %}.

A file is represented by the `file` struct:

```c
struct file {
  union {
    struct list_head fu_list; // list of file objects
    struct rcu_head fu_rcuhead; // RCU list after freeing
  } f_u;
  struct path f_path; // contains the dentry
  struct file_operations *f_op; // file operations table
  spinlock_t f_lock; // per-file struct lock
  atomic_t f_count; // file object’s usage count
  unsigned int f_flags; // flags specified on open
  mode_t f_mode; // file access mode
  loff_t f_pos; // file offset (file pointer)
  struct fown_struct f_owner; // owner data for signals
  const struct cred *f_cred; // file credentials
  struct file_ra_state f_ra; // read-ahead state
  u64 f_version; // version number
  void *f_security; // security module
  void *private_data; // tty driver hook
  struct list_head f_ep_links; // list of epoll links
  spinlock_t f_ep_lock; // epoll lock
  struct address_space *f_mapping; // page cache mapping
  unsigned long f_mnt_write_state; // debugging state
};
```

The file object contains a pointer to a `file_operations` object:

```c
struct file_operations {
  struct module *owner;
  loff_t (*llseek) (struct file *, loff_t, int);
  ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
  ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
  ssize_t (*aio_read) (struct kiocb *, const struct iovec *,
  unsigned long, loff_t);
  ssize_t (*aio_write) (struct kiocb *, const struct iovec *,
  unsigned long, loff_t);
  int (*readdir) (struct file *, void *, filldir_t);
  unsigned int (*poll) (struct file *, struct poll_table_struct *);
  int (*ioctl) (struct inode *, struct file *, unsigned int,
  unsigned long);
  long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
  long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
  int (*mmap) (struct file *, struct vm_area_struct *);
  int (*open) (struct inode *, struct file *);
  int (*flush) (struct file *, fl_owner_t id);
  int (*release) (struct inode *, struct file *);
  int (*fsync) (struct file *, struct dentry *, int datasync);
  int (*aio_fsync) (struct kiocb *, int datasync);
  int (*fasync) (int, struct file *, int);
  int (*lock) (struct file *, int, struct file_lock *);
  ssize_t (*sendpage) (struct file *, struct page *,
    int, size_t, loff_t *, int);
  unsigned long (*get_unmapped_area) (struct file *,
    unsigned long,
    unsigned long,
    unsigned long,
    unsigned long);
  int (*check_flags) (int);
  int (*flock) (struct file *, int, struct file_lock *);
  ssize_t (*splice_write) (struct pipe_inode_info *,
    struct file *,
    loff_t *,
    size_t,
    unsigned int);
  ssize_t (*splice_read) (struct file *,
    loff_t *,
    struct pipe_inode_info *,
    size_t,
    unsigned int);
  int (*setlease) (struct file *, long, struct file_lock **);
};
```

- `llseek()` updates the file pointer for a given offset.
- `read()` counts bytes from a given file position into a given buffer.
- `aio_read()` asynchronously reads bytes into a buffer.
- `write()` writes from the given buffer into the file, starting at a given offset.
- `poll()` sleeps, waiting for activity on the given file.
- `ioctl()` sends a command and argument to a device. It's used if the file is an open device node.
- `mmap()` memory maps the given file onto the given address space.
- `open()` creates a new file object and links it to the corresponding inode object.
- `flush()` called by the VFS when the reference count of an open file decreases.
- `release()` called by the VFS when the last reference to the file is destroyed.
- `fsync()` writes cached file data to disk.
- `get_unmapped_area()` gets unused address space to map a file.

{% cite lkd -l 281-4 %}

## Filesystem data structures

The `file_system_type` data structure describes the capabilities and behavior of a filesystem:

```c
struct file_system_type {
  const char *name;
  int fs_flags; // filesystem type flags
  // the following is used to read the superblock off the disk
  struct super_block *(*get_sb) (struct file_system_type *, int,
  char *, void *);
  // the following is used to terminate access to the superblock
  void (*kill_sb) (struct super_block *);
  struct module *owner; // module owning the filesystem
  struct file_system_type *next; // next file_system_type in list
  struct list_head fs_supers; // list of superblock objects
  // the remaining fields are used for runtime lock validation
  struct lock_class_key s_lock_key;
  struct lock_class_key s_umount_key;
  struct lock_class_key i_lock_key;
  struct lock_class_key i_mutex_key;
  struct lock_class_key i_mutex_dir_key;
  struct lock_class_key i_alloc_sem_key;
}
```

There is only one `file_system_type` per filesystem. When a filesystem is mounted, the `vfsmount` struct is created.

```c
struct vfsmount {
  struct list_head mnt_hash; // hash table list
  struct vfsmount *mnt_parent; // parent filesystem
  struct dentry *mnt_mountpoint; // dentry of this mount point
  struct dentry *mnt_root; // dentry of root of this fs
  struct super_block *mnt_sb; // superblock of this filesystem
  struct list_head mnt_mounts; // list of children
  struct list_head mnt_child; // list of children
  int mnt_flags; // mount flags
  char *mnt_devname; // device file name
  struct list_head mnt_list; // list of descriptors
  struct list_head mnt_expire; // entry in expiry list
  struct list_head mnt_share; // entry in shared mounts list
  struct list_head mnt_slave_list; // list of slave mounts
  struct list_head mnt_slave; // entry in slave list
  struct vfsmount *mnt_master; // slave’s master
  struct mnt_namespace *mnt_namespace; // associated namespace
  int mnt_id; // mount identifier
  int mnt_group_id; // peer group identifier
  atomic_t mnt_count; // usage count
  int mnt_expiry_mark; // is marked for expiration
  int mnt_pinned; // pinned count
  int mnt_ghosts; // ghosts count
  atomic_t __mnt_writers; // writers count
};
```

Each process on the system has its own list of open files, root filesystem, current working directory, and mount points. This is held in the `files_struct` struct:

```c
struct files_struct {
  atomic_t count; // usage count
  struct fdtable *fdt; // pointer to other fd table
  struct fdtable fdtab; // base fd table
  spinlock_t file_lock; // per-file lock
  int next_fd; // cache of next available fd
  struct embedded_fd_set close_on_exec_init; // list of close-on-exec fds
  struct embedded_fd_set open_fds_init // list of open fds
  struct file *fd_array[NR_OPEN_DEFAULT]; // base files array
};
```

The `fd_array` is a list of the open file descriptors. `NR_OPEN_DEFAULT` is equal to `BITS_PER_LONG`, which is `64` on a 64-bit architecture. If a file opens more than 64 objects then the kernel will allocate a new arrary and point `fdt` to it {% cite lkd -l 287 %}.

The `fs_struct` struct contains filesystem information that's related to a process:

```c
struct fs_struct {
  int users; // user count
  rwlock_t lock; // per-structure lock
  int umask; // umask
  int in_exec; // currently executing a file
  struct path root; // root directory
  struct path pwd; // current working directory
};
```

Per-process namespaces "enable each process to have a unique view of the mounted filesystems on the system—not just a unique root directory, but an entirely unique filesystem hierarchy" {% cite lkd -l 287 %}.

The `mnt_namespace` struct has details on the process namespace:

```c
struct mnt_namespace {
  atomic_t count; // usage count
  struct vfsmount *root; // root directory
  struct list_head list; // list of mount points
  wait_queue_head_t poll; // polling waitqueue
  int event; // event count
};
```

For most processes, the process descriptors point to unique `files_struct` and `fs_struct` objects. However, for processes created with clone flags, they will be shared. "The count member of each structure provides a reference count to prevent destruction while a process is still using the structure" {% cite lkd -l 288 %}.

By default, all processes share the same `namespace`. Only when the `CLONE_NEWNS` flag is specified during `clone()` is the process given a unique copy of the `namespace` structure {% cite lkd -l 288 %}.

## Block devices

**Block devices** are devices that offer random access to fixed-size blocks of memory. There are many types of block devices, like hard disks, floppy drives, and flash memory. These are all devices on which you mount a filesystem {% cite lkd -l 289 %}.

The **block I/O layer** is the kernel subsytem that manages block devices.

A **sector** is the smallest addressable unit on a block device. Most devices have 512-byte sectors {% cite lkd -l 290 %}.

The smallest logically addressable unit is the block. A block is a filesystem abstraction, and the kernel performs all disk operations using blocks. The block size can't be any smaller than a sector, and must be a multiple of a sector {% cite lkd -l 290 %}.

### Buffers and buffer heads

When a block is stored in memory, it’s stored in a buffer: an object that represents a disk block in memory {% cite lkd -l 291 %}.

A buffer is represented by the `buffer_head` struct:

```c
struct buffer_head {
  unsigned long b_state; // buffer state flags
  struct buffer_head *b_this_page; // list of page’s buffers
  struct page *b_page; // associated page
  sector_t b_blocknr; // starting block number
  size_t b_size; // size of mapping
  char *b_data; // pointer to data within the page
  struct block_device *b_bdev; // associated block device
  bh_end_io_t *b_end_io; // I/O completion
  void *b_private; // reserved for b_end_io
  struct list_head b_assoc_buffers; // associated mappings
  struct address_space *b_assoc_map; // associated address space
  atomic_t b_count; // use count
};
```

The `b_state` field contains the buffer state. The buffer can be in one of the following states:

<!-- prettier-ignore-start -->

| Status Flag      | Meaning                                                                                           |
| --------------   | ------------------------------------------------------------------------------------------------- |
| `BH_Uptodate`    | Buffer contains valid data.                                                                       |
| `BH_Dirty`       | Buffer is dirty. (The contents of the buffer are newer than the contents of the block on disk and therefore the buffer must eventually be written back to disk.) |
| `BH_Lock`        | Buffer is undergoing disk I/O and is locked to prevent concurrent access.                         |
| `BH_Req`         | Buffer is involved in an I/O request.                                                             |
| `BH_Mapped`      | Buffer is a valid buffer mapped to an on-disk block.                                              |
| `BH_New`         | Buffer is newly mapped via `get_block` and not yet accessed.                                      |
| `BH_Async_Read`  | Buffer is undergoing asynchronous read I/O via `end_buffer_async_read`.                           |
| `BH_Async_Write` | Buffer is undergoing asynchronous write I/O via `end_buffer_async_write`.                         |
| `BH_Delay`       | Buffer does not yet have an associated on-disk block (delayed allocation).                        |
| `BH_Boundary`    | Buffer forms the boundary of contiguous blocks—the next block is discontinuous.                   |
| `BH_Write_EIO`   | Buffer incurred an I/O error on write.                                                            |
| `BH_Ordered`     | Ordered write.                                                                                    |
| `BH_Eopnotsupp`  | Buffer incurred a “not supported” error.                                                          |
| `BH_Unwritten`   | Space for the buffer has been allocated on disk but the actual data has not yet been written out. |
| `BH_Quiet`       | Suppress errors for this buffer.                                                                  |

<!-- prettier-ignore-end -->

{% cite lkd -l 292 %}

The `b_count` field is the buffer's usage count. This must be incremented each time the buffer head is manipulated, to ensure the buffer head is not deallocated while it's being manipulated {% cite lkd -l 292 %}.

The physical block on disk to which a given buffer corresponds is the `b_blocknr`th logical block on the block device describe by `b_bdev`.

`b_page` is the physical page in memory to which a given buffer corresponds {% cite lkd -l 293 %}.

The buffer head describes the mapping between an on-disk block and the physical in-memory buffer {% cite lkd -l 293 %}.

## The `bio` struct

The `bio` struct represents active I/O operations as a list of segements, where a segment is a chunk of a buffer that is contiguous in memory. Individual buffers don't need to be contiguous in memory, because the buffers can be described in chunks that make up different memory locations {% cite lkd -l 294 %}.

You can see the `bio` struct:

```c
struct bio {
  sector_t bi_sector; // associated sector on disk
  struct bio *bi_next; // list of requests
  struct block_device *bi_bdev; // associated block device
  unsigned long bi_flags; // status and command flags
  unsigned long bi_rw; // read or write?
  unsigned short bi_vcnt; // number of bio_vecs off
  unsigned short bi_idx; // current index in bi_io_vec
  unsigned short bi_phys_segments; // number of segments
  unsigned int bi_size; // I/O count
  unsigned int bi_seg_front_size; // size of first segment
  unsigned int bi_seg_back_size; // size of last segment
  unsigned int bi_max_vecs; // maximum bio_vecs possible
  unsigned int bi_comp_cpu; // completion CPU
  atomic_t bi_cnt; // usage counter
  struct bio_vec *bi_io_vec; // bio_vec list
  bio_end_io_t *bi_end_io; // I/O completion method
  void *bi_private; // owner-private method
  bio_destructor_t *bi_destructor; // destructor method
  struct bio_vec bi_inline_vecs[0]; // inline bio vectors
};
```

The most important `bio` fields are:

- `bi_io_vec`
- `bi_vcnt`
- `bi_idx`

You can see the relationship between each of these structures in the following figure:

<figure>
  <img src="{{site.baseurl}}/assets/img/operating-systems/linux/filesystem/bio.svg" alt="">
  <figcaption><h4>Figure: Relationship between `bio`, `bio_vec`, and `page` {% cite lkd -l 295 %}</h4></figcaption>
</figure>

The `bio_vec` field points to an array of `bio_vec` structs. A `bio_vec` struct contains `page`, `offset`, and `length` values that describe a segment.

The `bio_vec` struct is defined in \<linux/bio.h\>:

```c
struct bio_vec {
  // pointer to the physical page on which this buffer resides
  struct page *bv_page;

  // the length in bytes of this buffer
  unsigned int bv_len;

  // the byte offset within the page where the buffer resides
  unsigned int bv_offset;
};
```

There are `bi_vcnt` vectors in an I/O operation, starting at `bi_io_vec`. The `bi_idx` is used to point to the current index into the array {% cite lkd -l 295 %}.

"The bio structure maintains a usage count in the `bi_cnt` field. When this field reaches zero, the structure is destroyed and the backing memory is freed" {% cite lkd -l 296 %}.

The difference between buffer heads and bio operations is that bio represents an I/O operation, which can include multiple pages of memory. The `buffer_head` represents a single block on a disk. `buffer_head` structs are used to keep information about buffers, while the `bio` describes active I/O operations {% cite lkd -l 296-7 %}.

## Request Queues

Block devices maintain request queues that store pending I/O requests. The request queue contains a linked list of requests, and control information. While the request queue has items in it, the block-device driver associated with the queue will get the request from the head of the queue, and submit it to the associated block device {% cite lkd -l 297 %}.

Request queues are represented with a `request_queue` struct, and individual requests are represented with a `request` struct, defined in \<linux/blkdev.h\>. A request can be composed of one or more `bio` structures {% cite lkd -l 297 %}.

## I/O Schedulers

I/O schedulers merge and sort I/O requests to ensure that disk operations are performed efficiently. Seeks are one of the slowest operations in computing, so organizing the disk operations to minimize them is crucial for good performance {% cite lkd -l 297 %}.

An I/O scheduler decides the order and the time that requests should be sent to block devices. The primary actions are sorting and merging. Merging combines two or more requests into one. Sorting is where requests are sorted according to the sectors they are operating on {% cite lkd -l 298-9 %}.

I/O schedulers are often called elevators, because they minimize disk seeks in the same way that an elevator visits floors—fist in one direction, before reversing and visiting floors in the opposite direction {% cite lkd -l 299 %}.

### The Deadline I/O Scheduler

The Deadline I/O scheduler was introduced to solve problems with the Linus elevator scheduler. The Linus elevator could lead to starvation of requests to one part of the disk in the case of many requests to a different part of the disk.

The Deadline I/O scheduler works on the premise that write operations can happen asynchronously in the background without affecting the user, whereas read operations must be fast. Read requests tend to be dependant on each other, so read latency can have a knock-on affect {% cite lkd -l 300 %}.

Each read request in the Deadline scheduler has a 500ms expiration time by default, and each write request has a 5000ms expiration {% cite lkd -l 301 %}.

The Deadline scheduler maintains a request queue, sorted by the physical location on disk. This queue is known as the sorted queue. When a new request is submitted to the sorted queue:

1. If a request to an adjacent on-disk sector is in the queue, the existing request and the new request are merged into a single request.
2. If a request in the queue is sufficiently old, the new request is inserted at the tail of the queue to prevent starvation of older requests.
3. If a suitable location sector-wise is in the queue, the new request is inserted there. This keeps the queue sorted by physical location on disk.
4. If no such suitable insertion point exists, the request is inserted at the queue tail.

{% cite lkd -l 299-301 %}

The scheduler also inserts requests into a second queue. Read requests are inserted into a FIFO read queue, and write requests are inserted into a FIFO write queue. Under normal operation the deadline queue pulls requests from the head of the sorted queue into the dispatch queue, which is then fed to the disk drive {% cite lkd -l 301 %}.

If the request at the head of the read queue or the write queue expires, the Deadline scheduler will begin taking jobs from the head of the FIFO queue.

<figure>
  <img src="{{site.baseurl}}/assets/img/operating-systems/linux/filesystem/dispatch-queue.svg" alt="">
  <figcaption><h4>Figure: The three queues of the Deadline I/O scheduler {% cite lkd -l 301 %}</h4></figcaption>
</figure>

Although the Deadline scheduler does not make strict guarantees that requests will be comitted before their expiration, it generally keeps commits below their expiration. This favors reads over writes and ensures that requests are not starved {% cite lkd -l 301 %}.

### The Anticaptory I/O scheduler

The Anticaptory scheduler is based on the Deadline scheduler. The differnce is that it has an added anticapatory heuristic.

The anticapatory scheduler attempts to minimize seeks that can occur when there are queued writes, and frequent read requests. The Anticapatory scheduler does not immediately seek back after read requests, instead it delays handling writes at a different section of disk for a certain ammount of time (by default 6ms). Any requests made to an adjacent area of the disk will be served immediately. There's a good chance that there will be another read request in those few miliseconds, and so the disk avoids an expensive seek operation {% cite lkd -l 302 %}.

### The Complete Fair Queuing I/O Scheduler

The Complete Fair Queuing (CFQ) I/O scheduler assigns incoming I/O requests to specific queues depending on the process that originated the request. The requests are sorted sectorwise, and adjacent requests are merged like in other requests {% cite lkd -l 303 %}.

The CFQ picks requests from the different process queues round-robin style. It picks a specified number (4 by default) of requests from each queue before moving onto the next one {% cite lkd -l 303 %}.

The CFQ is now the default I/O queue in Linux {% cite lkd -l 303 %}.

### The Noop I/O scheduler

The Noop I/O scheduler is intended for devices with truly random memory access, like flash memory, that doesn't have a seek time.

The benefit of the Noop I/O scheduler is that it doesn't need to implemented complex algoritms to sort requests, because there's either little or no seek penalty for some storage devices. The Noop I/O scheduler doesn't sort, but it still merges requests {% cite lkd -l 303 %}.

## References

{% bibliography --cited_in_order %}
