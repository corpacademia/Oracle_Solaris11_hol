**Task:** You have to create home directories for your users; use file
system quota to limit their space.

**Lab:** We'll create a user "`joe`" and set a disk quota for him.

Creating a user is pretty similar to most Unix/Linux systems. What's
different is what's going on behind the scenes.

``` console
root@solaris:~# useradd -m joe 
root@solaris:~# passwd joe 
New Password: oracle1 
Re-enter new Password: oracle1
passwd: password successfully changed for joe
```

In Solaris 11 behind the scenes we create a *separate* ZFS file system
for the user (parameter `-m`) in `/export/home`. Check it:

``` console
root@solaris:~# zfs list
NAME                              USED  AVAIL  REFER  MOUNTPOINT
labpool                           100K  1.06G  44.9K  /labpool
rpool                            8.33G  7.05G  4.97M  /rpool
rpool/ROOT                       4.73G  7.05G    31K  legacy
rpool/ROOT/solaris               4.73G  7.05G  4.22G  /
rpool/ROOT/solaris/var            409M  7.05G   198M  /var
rpool/VARSHARE                    144K  7.05G    50K  /var/share
rpool/VARSHARE/pkg                 63K  7.05G    32K  /var/share/pkg
rpool/VARSHARE/pkg/repositories    31K  7.05G    31K  /var/share/pkg/repositories
rpool/VARSHARE/zones               31K  7.05G    31K  /system/zones
rpool/dump                        792M  7.08G   768M  -
rpool/export                      906K  7.05G    32K  /export
rpool/export/home                 874K  7.05G    33K  /export/home
rpool/export/home/joe              35K  7.05G    35K  /export/home/joe
rpool/export/home/lab             806K  7.05G   806K  /export/home/lab
rpool/repo                       1.78G  7.05G  1.78G  /repo
rpool/swap                       1.03G  7.09G  1.00G  -
```

What does it mean for us, system administrators? That means we can use
all kinds of ZFS features (compression, deduplication, encryption) on a
per-user basis. We can create snapshots and perform rollbacks on a
per-user basis. We can even give users rights to perform those
operations themselves (look into Advanced labs folder). Now we'll set a
disk quota for `joe`'s home directory.

``` console
root@solaris:~# zfs set quota=200m rpool/export/home/joe
```

Now change user to "`joe`" and check how much space you can use:

``` console
root@solaris:# su - joe 
joe@solaris$ mkfile 150m file1 
```

Now check the file system's available space again:

``` console
root@solarislab:~# zfs list rpool/export/home/joe
NAME                   USED  AVAIL  REFER  MOUNTPOINT
rpool/export/home/joe  150M  49.9M   150M  /export/home/joe
and from Joe's perspective:
joe@solarislab:~$ df -h $HOME
Filesystem             Size   Used  Available Capacity  Mounted on
rpool/export/home/joe
                   200M   150M        50M    76%    /export/home/joe
```

Now try to create another file:

``` console
joe@solaris$ mkfile 150m file2 
```

This time we will get an error: "Disk quota exceeded". More than that,
even root can't create another file in `/export/home/joe` directory. Try
it!

Change the quota for `joe` in the other window:

``` console
root@solaris:~# zfs set quota=300m rpool/export/home/joe 
```

Then change back to the `joe`'s window and try again:

``` console
joe@solaris$ rm file2 
joe@solaris$ mkfile 150m file2 
```

Success! As you can see, it's pretty easy to create and manage ZFS
filesystems. Remember, by default Solaris 11 creates a separate ZFS file
system for each user.

