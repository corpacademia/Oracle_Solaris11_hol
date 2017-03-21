What is ZFS?
------------

ZFS (Zettabyte File System) was developed by Sun Microsystems and introduced in Solaris 10 in 2005. Since then it became very popular&mdash;and not only in Solaris world. The main goal of ZFS design was to make sysadmin's life easier. You can quickly and easily:

- create disk pools and filesystems (it takes less than a second, no matter how big your pool is)
- snapshot and clone your data for backup or dev/test purposes 
- turn on compression and deduplication to save space
- migrate Terabytes of your application data with minimal downtime

In addition to that, ZFS is extremely reliable: everything is covered by checksums, all the way along the path of the data. ZFS is scalable up to the numbers which are hard to imagine: file systems can have up to 256 quadrillion zettabytes of storage, directories can have up to 256 trillion entries.  

You can find more information about ZFS concepts and design strategy in this document: [What is ZFS?](https://docs.oracle.com/cd/E26505_01/html/E37384/zfsover-2.html)

Lab Outline: Situations and Exercises
-------------------------------------

We have collected several practical situations where we can use some ZFS
features. It is not absolutely manadatory to perform all the exercises in the
following order, but sometimes we use results of the preceding tasks, like
using the ZFS pool we created in the first exercise. If you feel you are
missing something, try to look at the preceding exercises. 

|Situation | Exercise|
|----------|---------|
You have some disks to use for your new file system. Create a new disk pool and a file system on top of it.   | [ZFS Pools](zfs_pools.md) 
You have to create home directories for your users; use file system quota to limit their space.    | [ZFS File systems](zfs_fs.md) 
You are becoming low on your disk space. Add a couple more disks to your pool and expand your file system.|[ZFS Compression](zfs_compress.md)
Users tend to keep a lot of similar files in their archives. Is it possible to save space by using deduplication?|[ZFS Deduplication](zfs_dedup.md)
A user has accidentally deleted her file. How to restore it without getting to the backup?|[ZFS Snapshots](zfs_snap.md)
You need a copy of your data to work on it separately (e.g. for test/dev environment). How to create a copy of your multi-terabyte dataset really fast?|[ZFS Clones](zfs_clones.md)
Your users want to be able to create and rollback their own snapshots (instead of asking you to do that|[ZFS Rights Delegation](zfs_rights.md)
How do we backup and restore ZFS file systems?|[ZFS Backup and Restore](zfs_backup.md)
You want to free up some space in your ZFS file system. You find several huge files, you delete them, but you don't see that you gained free space. What's happening?|[ZFS Free Space](zfs_free.md)
All ZFS file systems share the same pool. You want to make sure that some critical file systems are guaranteed to have enough space.|[ZFS Reservations](zfs_reservations.md)
You have received a new, fast storage array and you want to migrate your application data to it. How to do it with minimal downtime?|[ZFS Shadow Migration](zfs_shadow.md)


