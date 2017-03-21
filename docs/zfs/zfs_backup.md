No data management system is complete without proper backup and restore
facility. Let's see what's available in ZFS.

Create a new file system and copy some data into it. In our case we'll
use system manuals again.

``` console
root@solaris:~# zfs create -o compression=lz4 -o mountpoint=/data rpool/data
root@solaris:~# cp -rp /usr/share/man/ /data/
root@solaris:~# zfs list rpool/data
NAME         USED  AVAIL  REFER  MOUNTPOINT
rpool/data  94.5M  19.8G  94.5M  /data
```

Try `ls -R /data` and see a lot of manual files listed in this
directory. Everything is good.

Before creating a backup we have to take a snapshot of our data. (Note
that we don't have to stop applications that might be accessing the
data.) In the command below we use the '`date(1)`' command to produce a
timestamp on the backup snapshot. Feel free to use the date format which
is suitable for you.

``` console
root@solaris:~# zfs snapshot rpool/data@backup-`date +%Y-%m-%d`
root@solaris:~# zfs list -r -t filesystem,snapshot rpool/data
NAME                           USED  AVAIL  REFER  MOUNTPOINT
rpool/data                    94.5M  19.5G  94.5M  /data
rpool/data@backup-2016-03-28      0      -  94.5M  -
```

Now let's create a separate ZFS pool to store our backups. In our lab
environment we'll use plain files instead of actual disks, but in real
life most likely you will use a separate storage array.

``` console
root@solaris:~# cd /devdsk
root@solaris:/devdsk# mkfile 300M backupdisk
root@solaris:/devdsk# cd
root@solaris:~# zpool create backuppool /devdsk/backupdisk 
```

And now we are ready to send the snapshot we've just created to that
"separate storage array".

``` console
root@solaris:~# zfs send rpool/data@backup-2016-03-28 | zfs recv backuppool/backup
root@solaris:~# zfs list -r -t filesystem,snapshot backuppool/backup
NAME                                 USED  AVAIL  REFER  MOUNTPOINT
backuppool/backup                    161M   101M   161M  /backuppool/backup
backuppool/backup@backup-2016-03-28     0      -   161M  -
```

This is all good, but why the backup file system takes more space than
the original? Of course, we didn't specify the compression option! Can
we do it while receiving the snapshot? Of course! Repeat the command but
now with the compression option (don't forget to destroy the backup
first):

``` console
root@solaris:~# zfs destroy -r backuppool/backup
root@solaris:~# zfs send rpool/data@backup-2016-03-28 | zfs recv -o compression=lz4 backuppool/backup
root@solaris:~# zfs list -r -t filesystem,snapshot backuppool/backup
NAME                                  USED  AVAIL  REFER  MOUNTPOINT
backuppool/backup                    94.3M   167M  94.3M  /backuppool/backup
backuppool/backup@backup-2016-03-28      0      -  94.3M  -
```

Now we'll use this backup to restore our file system. Take a look at
your data one last time and destroy the file system.

``` console
root@solaris:~# ls -R /data
... (long list of files follows. Feel free to stop it with Ctrl-C)....
root@solaris:~# zfs destroy -r rpool/data
root@solaris:~# ls -R /data
/data:
```

Now receive your data back from the backup and check if everything is
OK.

``` console
root@solaris:~# zfs send backuppool/backup@backup-2016-03-28 | zfs recv -o compression=lz4 rpool/data
root@solaris:~# ls -R /data
/data:
```

What? Where is our data?? Try 'zfs list':

``` console
root@solaris:~# zfs list rpool/data
NAME         USED  AVAIL  REFER  MOUNTPOINT
rpool/data  94.3M  19.2G  94.3M  /rpool/data
```

Of course! The file system is there, but it's not mounted under /data as
it was before. Let's receive the backup again, but now specify the
mountpoint.

``` console
root@solaris:~# zfs destroy -r rpool/data
root@solaris:~# zfs send backuppool/backup@backup-2016-03-28 | zfs recv -o compression=lz4 -o mountpoint=/data rpool/data
root@solaris:~# ls -R /data
...(a lot of files)....
```

Well, now everything is back!

You might have noticed that we used Unix _pipe_ to send and receive ZFS
datasets. That means that `zfs send` command produces a stream and
sends it to standard output. So, instead of "piping" that stream into
another command we can just redirect it to a file, like this:

``` console
root@solaris:~# zfs send rpool/data@backup-2016-03-28 > backup
root@solaris:~# file backup
backup:     ZFS snapshot stream
root@solaris:~# ls -lh backup
-rw-r--r--   1 root     root        181M Mar 29 12:26 backup
```

What can we do with this backup file? We can store it in some safe
location, we can copy it and store in several locations, just in case.
When we need to restore from it, we use `zfs recv` command and send this
file to its standard input. It's pretty easy and very much in "Unix
spirit", isn't it?

``` console
root@solaris:~# zfs destroy -r rpool/data
root@solaris:~# zfs recv -o compression=lz4 -o mountpoint=/data rpool/data < backup
```

Also that means that we can send the backup stream to another machine
via ssh tunnel. If you have another system available (as in our Oracle
Solution Center lab environment), try to send it there:

``` console
root@solaris:~# zfs send rpool/data@backup-2016-03-28 | ssh  zfs recv -o compression=lz4 backuppool/backup
```

There are a lot more topics in ZFS sending and receiving: full and
incremental streams, stream packages, recursive stream packages, etc.
Feel free to open "Managing ZFS File Systems in Oracle Solaris" manual
in the official documentation set and practice using your VirtualBox lab
environment.
