One day you have noticed that you don't have enough free space on your
ZFS file system. Of course, you know how easy it is to expand your ZFS
pool: just add more disks. But you don't have any extra disks available
right now. It's time to use good old method: clean up some garbage. So
you go and look for temporary files, downloaded ISO images, old logs and
other stuff you stored "just temporarily" several months ago and forgot
about it. You delete them all and check your free space again. But...
nothing has changed here. You still have shortage of disk space. Why?

The short answer is: snapshots. Remember you took several snapshots on
your file system? Remember you were told that they don't occupy any disk
space? Yes, that's true. UNTIL you start making changes to your file
system.

Let's perform an experiment. Create a file which will represent a disk
device and then create a ZFS pool, which will automatically create and
mount a new file system.

``` console
root@solaris:~# mkfile 1g /var/tmp/disk1
root@solaris:~# zpool create test1 /var/tmp/disk1
root@solaris:~# zpool list
NAME    SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
rpool  15.6G  10.3G  5.33G  65%  1.00x  ONLINE  -
test1  1016M   152K  1016M   0%  1.00x  ONLINE  -
root@solaris:~# zfs list test1
NAME   USED  AVAIL  REFER  MOUNTPOINT
test1   85K   984M    31K  /test1
```

Now create a file in this file system and check available space.

``` console
root@solaris:~# mkfile 100m /test1/file1
root@solaris:~# zfs list test1
NAME   USED  AVAIL  REFER  MOUNTPOINT
test1  100M   884M   100M  /test1
```

So far so good. Exactly 100 MB of space is taken by our file. Of course,
if we delete this file right now, we get all the space back.

``` console
root@solaris:~# rm /test1/file1
root@solaris:~# zfs list test1
NAME   USED  AVAIL  REFER  MOUNTPOINT
test1  265K   984M    31K  /test1
```

Create the file again and take a snapshot this time.

``` console
root@solaris:~# mkfile 100m /test1/file1
root@solaris:~# zfs list test1
NAME   USED  AVAIL  REFER  MOUNTPOINT
test1  100M   884M   100M  /test1
root@solaris:~# zfs snapshot test1@snap1
```

Check the sizes of both the file system and the snapshot:

``` console
root@solaris:~# zfs list -r -t all test1
NAME         USED  AVAIL  REFER  MOUNTPOINT
test1        100M   884M   100M  /test1
test1@snap1     0      -   100M  -
```

You see: the snapshot's size is exactly zero, as you were told already.
Now delete the file and check the sizes again.

``` console
root@solaris:~# rm /test1/file1
root@solaris:~# ls /test1
root@solaris:~# zfs list -r -t all test1
NAME         USED  AVAIL  REFER  MOUNTPOINT
test1        100M   884M    31K  /test1
test1@snap1  100M      -   100M  -
```

Now the snapshot takes exactly 100 Megabytes--the size of your deleted
file! You don't see the file with `ls(1)` command, but it is still
there, in the snapshot. File is successfully deleted, but your free
space is still the same: 884 Megabytes. It's interesting to note that
`df(1M)` command can produce confusing output in this case:

``` console
oot@solaris:~# df -h /test1
Filesystem             Size   Used  Available Capacity  Mounted on
test1                  984M    31K       884M     1%    /test1
```

From this output it's hard to figure out where 100 Megabytes have gone.
So now you understand why it's recommended to use native `zfs(1M)`
commands when working with ZFS file systems.

It's all good, but we didn't answer the original question: how to get
more free storage space when you need it? In most cases, your file
systems will have snapshots, perhaps many of them. Deleting one or
several snapshots can really help in getting more storage space. In our
case:

``` console
root@solaris:~# zfs destroy test1@snap1
root@solaris:~# zfs list -r -t all test1
NAME   USED  AVAIL  REFER  MOUNTPOINT
test1  110K   984M    31K  /test1
```

All free space is back! Congratulations!

