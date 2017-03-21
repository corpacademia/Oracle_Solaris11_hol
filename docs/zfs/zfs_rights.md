Remember, in our previous ZFS lab we created file system snapshots and
then used them to restore files we have "accidentally" deleted? It would
be great if it was possible to give your users rights to create and
restore snapshots on their own, without distracting you, sysadmin, from
more important tasks?

Yes, it's possible! You can delegate these rights to your users. Let's
create a user Joe and give him rights to manage his own home directory,
i.e. file system (remember, in Solaris 11 `useradd` operation creates a
ZFS file system for the user, not just a home directory!).

``` console
root@solaris:~# useradd -c "Joe User" -m joe
80 blocks
root@solaris:~# passwd joe
New Password:
Re-enter new Password:
passwd: password successfully changed for joe
root@solaris:~# zfs allow joe create,destroy,mount,snapshot rpool/export/home/joe
```

Now become Joe and create a file. After that, create a snapshot and
"accidentally" delete the file you have just created:

``` console
root@solaris:~# su - joe
Oracle Corporation SunOS 5.11 11.1 September 2012
joe@solaris:~$
joe@solaris:~$ vi firstfile.txt
joe@solaris:~$ cat firstfile.txt
This is my first file.
joe@solaris:~$ pwd
/export/home/joe
joe@solaris:~$ zfs snap rpool/export/home/joe@snap1
joe@solaris:~$ rm firstfile.txt
joe@solaris:~$ cat firstfile.txt
cat: cannot open firstfile.txt: No such file or directory
```

Yes, the file is gone. But Joe is a smart guy, he has taken a snapshot
after he created the file. But he just forget the name of the
snapshot... Let's figure it out:

``` console
joe@solaris:~$ zfs list -t all | grep joe
rpool/export/home/joe 56K 8.52G 35.5K /export/home/joe
rpool/export/home/joe@snap1 20.5K - 35.5K -
```

OK, now Joe knows the name and tries to rollback the snapshot:

``` console
joe@solaris:~$ zfs rollback rpool/export/home/joe@snap1
cannot rollback 'rpool/export/home/joe': permission denied
```

What? A-ha, we forgot to add rollback to the list of rights for Joe.
Let's fix that:

``` console
joe@solaris:~$ exit
logout
root@solaris:~# zfs allow joe rollback rpool/export/home/joe
root@solaris:~# su - joe
Oracle Corporation SunOS 5.11 11.1 September 2012
joe@solaris:~$ zfs rollback rpool/export/home/joe@snap1
joe@solaris:~$ ls
firstfile.txt local.cshrc local.login local.profile
joe@solaris:~$ cat firstfile.txt
This is my first file.
```

What a relief for Joe! And what a relief for you--now your users can
manage their filesystems on their own! Joe can even create new file
systems under his home directory. Try this as Joe to test if it's
possible.
