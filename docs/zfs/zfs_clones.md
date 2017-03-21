**Task:** We need to create a copy of our transactional data to do some
analysis and modifications. In other words, we need a writeable
snapshot.

**Lab:** In this lab we will use ZFS cloning feature. Clones are similar
to snapshots, but you can modify them. Similarly to snapshots, it takes
seconds to create them and they take almost no space until you start
changing your files.

Clones can't be created from a live filesystem. To create a clone we
have to have a snapshot first. In this lab we can use a snapshot
'@snap1' we have just created.

``` console
root@solaris:~# zfs clone labpool/archive/a@snap1 labpool/a_work
root@solaris:~# zfs list -r -t all labpool
NAME                      USED  AVAIL  REFER  MOUNTPOINT
labpool                  18.6M  1.10G  49.4K  /labpool
labpool/a_work           26.9K  1.10G  3.17M  /labpool/a_work
labpool/archive          12.8M  1.10G  52.4K  /labpool/archive
labpool/archive@snap1        0      -  52.4K  -
labpool/archive/a        3.20M  1.10G  3.17M  /labpool/archive/a
labpool/archive/a@snap1  26.9K      -  3.17M  -
labpool/archive/b        3.17M  1.10G  3.17M  /labpool/archive/b
labpool/archive/b@snap1      0      -  3.17M  -
labpool/archive/c        3.17M  1.10G  3.17M  /labpool/archive/c
labpool/archive/c@snap1      0      -  3.17M  -
labpool/archive/d        3.17M  1.10G  3.17M  /labpool/archive/d
labpool/archive/d@snap1      0      -  3.17M  -
labpool/zman             5.41M  1.10G  5.41M  /labpool/zman
```

Check if the archive is in place in the clone filesystem:

``` console
root@solaris:~# cd /labpool/a_work
root@solaris:/labpool/a_work# ls
man.tar.gz
```

Unpack the archive and then check the original directory.

``` console
root@solaris:/labpool/a_work# tar xzvf man1.tar.gz 
....................
tar: Removing leading '/' from '/usr/share/man/man1/tracker-services.1'
x usr/share/man/man1/tracker-services.1, 1938 bytes, 4 tape blocks
root@solaris:/labpool/a_work# ls -l 
total 6413
-rw-r--r--   1 root     root     3257177 Dec 13 17:05 man1.tar.gz
drwxr-xr-x   3 root     root           3 Dec 13 18:04 usr
root@solaris:/labpool/a_work# cd ../archive/a
root@solaris:/labpool/archive/a# ls  -l
total 6409
-rw-r--r--   1 root     root     3257177 Dec 13 17:05 man1.tar.gz
```

This powerful cloning feature can be used for your regular data. Oracle
Solaris uses it internally to create boot environments and zone clones.
They will be described in the following lab exercises.
