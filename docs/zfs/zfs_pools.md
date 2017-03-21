**Task:** You have several disks to use for your new file system. Create
a new disk pool and a file system on top of it.

**Lab:** We will check the status of disk pools, create our own pool and
expand it.

Our Solaris 11 installation already has a ZFS pool. It's your root file
system. Check this:

``` console
root@solaris:~# zpool list 

NAME    SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
rpool  15.9G  5.64G  10.2G  35%  1.00x  ONLINE  -
```

This is our root system ZFS pool. In Solaris 11 the root file system
must be ZFS created on top of ZFS pool. What do we know about this pool?

``` console
root@solaris:~# zpool status rpool 
pool: rpool
state: ONLINE
scan: none requested
config:

NAME        STATE     READ WRITE CKSUM
rpool       ONLINE       0     0     0
  c1t0d0s1  ONLINE       0     0     0

errors: No known data errors
```

In our typical lab environment in the VirtualBox VM we don't have extra disks
to experiment with. Let's use files instead. We will create a
separate directory and create files in it.

``` console
root@solaris:~# mkdir /devdsk
root@solaris:~# cd /devdsk
root@solaris:/devdsk# mkfile 200m c2d{0..11}
```

Now we have 12 files which *look* like disks and we will use them like
if they were disks. Create a ZFS pool out of 4 disks using RAID-Z
protection:

``` console
root@solaris:/devdsk# zpool create labpool raidz /devdsk/c2d0 /devdsk/c2d1 /devdsk/c2d2 /devdsk/c2d3
```

That was easy, wasn't it? And fast, too! Check our ZFS pools again:

``` console
root@solaris:~# zpool list 
NAME      SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
labpool   748M   158K   748M   0%  1.00x  ONLINE  -
rpool    15.6G  7.79G  7.83G  49%  1.00x  ONLINE  -
```
Check its status:

``` console
root@solaris:~# zpool status labpool
pool: labpool
state: ONLINE
scan: none requested
config:

NAME        STATE     READ WRITE CKSUM
labpool     ONLINE       0     0     0
  raidz1-0  ONLINE       0     0     0
    c1t4d0  ONLINE       0     0     0
    c1t5d0  ONLINE       0     0     0
    c1t6d0  ONLINE       0     0     0
    c1t7d0  ONLINE       0     0     0

errors: No known data errors
```

By the way, the file system was also created and mounted automatically:

``` console
root@solaris:~# zfs list labpool 
NAME      USED  AVAIL  REFER  MOUNTPOINT
labpool  97.2K   527M  44.9K  /labpool
```

Do you need more space? Adding disks to the existing ZFS pool is as easy
as creating it:

``` console
root@lab0:/devdsk# zpool add labpool raidz /devdsk/c2d4 /devdsk/c2d5 /devdsk/c2d6 /devdsk/c2d7
```

Check its size again:

``` console
root@solaris:~# zpool list labpool 
NAME      SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
labpool  1.46G   134K  1.46G   0%  1.00x  ONLINE  -
root@solaris:~# zfs list labpool
NAME     USED  AVAIL  REFER  MOUNTPOINT
labpool  100K  1.06G  44.9K  /labpool
```

Take a note of the increased pool and file system sizes. Why are they
different? What do you think?

Check the pool status:

``` console
root@solaris:~# zpool status labpool
pool: labpool
state: ONLINE
scan: none requested
config:

NAME         STATE     READ WRITE CKSUM
labpool      ONLINE       0     0     0
  raidz1-0   ONLINE       0     0     0
    c1t4d0   ONLINE       0     0     0
    c1t5d0   ONLINE       0     0     0
    c1t6d0   ONLINE       0     0     0
    c1t7d0   ONLINE       0     0     0
  raidz1-1   ONLINE       0     0     0
    c1t8d0   ONLINE       0     0     0
    c1t9d0   ONLINE       0     0     0
    c1t10d0  ONLINE       0     0     0
    c1t11d0  ONLINE       0     0     0

errors: No known data errors
```

Note that there are two disk groups in this pool both protected with
RAID-Z. ZFS has many options to protects your data, you can learn and
experiment with them later. Hint: learn more about RAID-Z2 and RAID-Z3
options and how they can protect your data.

