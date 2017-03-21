**Task:** A user has accidentally deleted her file. How to restore it
without getting to the tape backup?

**Lab:** Create a snapshot of our archive filesystem:

``` console
root@solaris:~# zfs snapshot -r labpool/archive@snap1 
```

Note the `-r` parameter telling ZFS that we want to snapshot all
dependent filesystems as well. Check your work:

``` console
root@solaris:~# zfs list -r -t all labpool
NAME                      USED  AVAIL  REFER  MOUNTPOINT
labpool                  18.5M  1.10G  47.9K  /labpool
labpool/archive          12.7M  1.10G  52.4K  /labpool/archive
labpool/archive@snap1        0      -  52.4K  -
labpool/archive/a        3.17M  1.10G  3.17M  /labpool/archive/a
labpool/archive/a@snap1      0      -  3.17M  -
labpool/archive/b        3.17M  1.10G  3.17M  /labpool/archive/b
labpool/archive/b@snap1      0      -  3.17M  -
labpool/archive/c        3.17M  1.10G  3.17M  /labpool/archive/c
labpool/archive/c@snap1      0      -  3.17M  -
labpool/archive/d        3.17M  1.10G  3.17M  /labpool/archive/d
labpool/archive/d@snap1      0      -  3.17M  -
labpool/zman             5.41M  1.10G  5.41M  /labpool/zman
```

Imagine user `a` had deleted her archive stored in /labpool/archive/a.

``` console
root@solaris:~# rm /labpool/archive/a/* 
```

And she comes to you asking for help. "Can you restore my archive before
tomorrow?", she asks. Of course, you can! In a matter of seconds, not
hours, her archive files will be back! Just rollback the snapshot!

``` console
root@solaris:~# zfs rollback labpool/archive/a@snap1 
```

You may ask "How often should I make snapshots? Do snapshots take a lot
of space? The answer is here:

``` console
root@solaris:~# zfs list -r -t all labpool 
NAME                    USED  AVAIL  REFER  MOUNTPOINT
labpool                18.5M  1.10G  47.9K  /labpool
labpool/archive        12.7M  1.10G  52.4K  /labpool/archive
labpool/archive@snap1      0      -  52.4K  -
labpool/archive/a      3.17M  1.10G  3.17M  /labpool/archive/a
labpool/archive/b      3.17M  1.10G  3.17M  /labpool/archive/b
labpool/archive/c      3.17M  1.10G  3.17M  /labpool/archive/c
labpool/archive/d      3.17M  1.10G  3.17M  /labpool/archive/d
labpool/zman           5.41M  1.10G  5.41M  /labpool/zman
```

The snapshot uses 0 bytes because we have not changed anything in your
home directory. When you make changes to your filesystem, it will take
more space. Try to change something in the /labpool/archive directory
and check the sizes again. Learn more about how snapshots work from our
OTN technical presentations and articles.

Food for thought: How can snapshots be used in the real life
environment? Backup is the first idea that comes to mind. What else?
