Do you remember how much space was available to the file systems? You
are right, all the space that is available in the whole pool, is
available to all file systems that are created on that pool. Is it good?
Of course! You don't have to carefully calculate how much space you will
need for this particular file system, you just use it! And when you need
more, you just add more disks to the pool and more space is immediately
available to all file systems!

Yes, it's good and very convenient, but... All file systems share the
same pool and those of them who grab their space faster, eventually will
take over the whole pool and no space will remain available to other
file systems who are not that greedy. Yes, you can add disks, but it
takes time and your users don't want to wait. What can be done to
prevent those greedy file systems from hijacking the whole ZFS pool? You
can use quotas to limit their appetite, but there is better solution.
You can do just the opposite to quotas: instead of setting the maximum
space allocation, you can set the minimum, guaranteed space for the most
important file systems.

Imagine you have a Big Boss who wants to have a guaranteed space for his
projects and files in a file system. And you have a Little Boss who is
not that demanding. We will use this example to demonstrate how ZFS can
handle this situation. Let's start with creating a ZFS pool where we are
going to store data from both bosses. First, figure out what disks are
available on your system.

``` console
root@solaris:~# echo | format (this command prevents format from going into interactive mode)
Searching for disks...done
AVAILABLE DISK SELECTIONS:
0. c7d0
/pci@0,0/pci-ide@1,1/ide@0/cmdk@0,0
1. c7d1
/pci@0,0/pci-ide@1,1/ide@0/cmdk@1,0
Specify disk (enter its number): Specify disk (enter its number):
root@solaris:~#
```

OK, two disks are installed in the system. Which one is taken by the
root pool and which one is available to create another ZFS pool?

``` console
root@solaris:~# zpool status
pool: rpool
state: ONLINE
scan: none requested
config:
NAME STATE READ WRITE CKSUM
rpool ONLINE 0 0 0
c7d0 ONLINE 0 0 0
errors: No known data errors
```

That means that `c7d0` is taken by rpool and we can use `c7d1` to create
an additional ZFS pool. Let's do that.

    root@solaris:~# zpool create labpool c7d1

Now we have our own pool; let's create a file system for our Little
Boss. He is not very demanding, so we create a default file system.

``` console
root@solaris:~# zfs create labpool/littleboss
root@solaris:~# zfs list
NAME USED AVAIL REFER MOUNTPOINT
labpool 124K 976M 32K /labpool
labpool/littleboss 31K 976M 31K /labpool/littleboss
rpool 5.88G 9.50G 4.90M /rpool
rpool/ROOT 4.07G 9.50G 31K legacy
rpool/ROOT/solaris 4.07G 9.50G 3.77G /
rpool/ROOT/solaris/var 201M 9.50G 196M /var
rpool/VARSHARE 52.5K 9.50G 52.5K /var/share
rpool/dump 792M 9.53G 768M -
rpool/export 868K 9.50G 32K /export
rpool/export/home 836K 9.50G 32K /export/home
rpool/export/home/lab 804K 9.50G 804K /export/home/lab
rpool/swap 1.03G 9.53G 1.00G -
Let's create another file system, this time for Big Boss.
root@solaris:~# zfs create labpool/bigboss
root@solaris:~# zfs list
NAME USED AVAIL REFER MOUNTPOINT
labpool 162K 976M 33K /labpool
labpool/bigboss 31K 976M 31K /labpool/bigboss
labpool/littleboss 31K 976M 31K /labpool/littleboss
. . .
```

You see: available space in both file systems is the same and it's equal
to the space available in the whole pool. Try to create a big file in
Boss' file system:

``` console
root@solaris:~# mkfile 200m /labpool/littleboss/bigfile
root@solaris:~# zfs list
NAME USED AVAIL REFER MOUNTPOINT
labpool 200M 776M 33K /labpool
labpool/bigboss 31K 776M 31K /labpool/bigboss
labpool/littleboss 200M 776M 200M /labpool/littleboss
. . .
```

(it may take a second to get exactly 200M. If the number is smaller, try
zfs list again) You see: Little Boss' file just has taken 200 MB of
space - from both file systems! Big Boss might not like it. He wants to
make sure that he has at least 500 MB of space for his files! OK, let's
use ZFS reservation:

``` console
root@solaris:~# zfs set reservation=500m labpool/bigboss
root@solaris:~# zfs list
NAME USED AVAIL REFER MOUNTPOINT
labpool 700M 276M 33K /labpool
labpool/bigboss 31K 776M 31K /labpool/bigboss
labpool/littleboss 200M 276M 200M /labpool/littleboss
. . .
```

Now Big Boss has the same amount available, 776 MB, but we just have cut
500 MB of space from Little Boss. And this space is reserved for Big
Boss. He is happy now. After you are done with this, destroy both
filesystems to clean up the pool for future exercises.

``` console
root@solaris:~# zfs destroy labpool/bigboss
root@solaris:~# zfs destroy labpool/littleboss
```
