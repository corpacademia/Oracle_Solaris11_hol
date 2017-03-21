**Task:** Users tend to keep a lot of similar files in their archives.
Is it possible to save space by using deduplication?

**Lab:** We will create a ZFS file system with deduplication turned on
and see if it helps.

Let's model the following situation: we have a file system which is used
as an archive. We'll create separate file systems for each user and
imagine that they store similar files there.

We will use the ZFS pool called `labpool` that we have created in the
first exercise.

Create a file system with deduplication and compression:

``` console
root@solaris:~# zfs create -o dedup=on -o compression=gzip labpool/archive
```

Create users' file systems (we'll call them a, b, c, d for simplicity):

``` console
root@solaris:~# zfs create labpool/archive/a
root@solaris:~# zfs create labpool/archive/b
root@solaris:~# zfs create labpool/archive/c
root@solaris:~# zfs create labpool/archive/d
```

Check their "dedup" parameter:

``` console
root@solaris:~# zfs get dedup labpool/archive/a
NAME               PROPERTY  VALUE          SOURCE
labpool/archive/a  dedup     on             inherited from labpool/archive
```

Children file systems inherit parameters from their parents.

Create an archive from /usr/share/man/man1, for example.

``` console
root@solaris:~# tar czf /tmp/man1.tar.gz /usr/share/man/man1
```

And copy it four times to the file systems we've just created. Don't
forget to check deduplication rate after each copy.

``` console
root@solaris:~# cd /labpool/archive
root@solaris:/labpool/archive# ls -lh /tmp/man1.tar.gz 
-rw-r--r--   1 root     root        3.2M Oct  3 15:30 /tmp/man1.tar.gz
root@solaris:/labpool/archive# zpool list labpool
NAME      SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
labpool  1.46G  7.99M  1.45G   0%  1.00x  ONLINE  -
root@solaris:/labpool/archive# cp /tmp/man1.tar.gz a/
root@solaris:/labpool/archive# zpool list labpool
NAME      SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
labpool  1.46G  12.6M  1.45G   0%  1.00x  ONLINE  -
root@solaris:/labpool/archive# cp /tmp/man1.tar.gz b/
root@solaris:/labpool/archive# zpool list labpool
NAME      SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
labpool  1.46G  12.7M  1.45G   0%  2.00x  ONLINE  -
root@solaris:/labpool/archive# cp /tmp/man1.tar.gz c/
root@solaris:/labpool/archive# zpool list labpool
NAME      SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
labpool  1.46G  12.7M  1.45G   0%  2.00x  ONLINE  -
root@solaris:/labpool/archive# zpool list labpool
NAME      SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
labpool  1.46G  12.5M  1.45G   0%  3.00x  ONLINE  -
root@solaris:/labpool/archive# cp /tmp/man1.tar.gz d/
root@solaris:/labpool/archive# zpool list labpool
NAME      SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
labpool  1.46G  12.5M  1.45G   0%  4.00x  ONLINE  -
```

It might take a couple of seconds for ZFS to commit those changes and
report the correct dedup ratio. Just repeat the command if you don't see
the results listed above.

Remember, we set compression to "on" as well when we created the file
system? Check the compression ratio:

``` console
root@solaris:/labpool/archive# zfs get compressratio labpool/archive
NAME             PROPERTY       VALUE  SOURCE
labpool/archive  compressratio  1.00x  -
```

The reason is simple: we placed in the file system files that are
compressed already. Sometimes compression can save you some space,
sometimes deduplication can help.

It's interesting to note that ZFS uses deduplication on a block level,
not on a file level. That means if you have a single file but with a lot
of identical blocks, it will be deduplicatied too. Let's check this.
Create a new ZFS pool:

``` console
root@lab0:~# zpool create ddpool raidz /devdsk/c2d8 /devdsk/c2d9 /devdsk/c2d10 /devdsk/c2d11
```

As you remember, when we create a ZFS pool, by default a new ZFS
filesystem with the same name is created and mounted. We just have to
turn deduplication on:

``` console
root@solaris:~# zfs set dedup=on ddpool
```

Now let's create a big file that contains 1000 copies of the same block.
In the following commands we are figuring out the size of a ZFS block
and creating a single file of that size. Then we are copying that file
1000 times into our big file.

``` console
root@solaris:~# zfs get recordsize ddpool
NAME    PROPERTY    VALUE  SOURCE
ddpool  recordsize  128K   default
root@solaris:~# mkfile 128k 128k-file
root@solaris:~# for i in {1..1000} ; do cat 128k-file >> 1000copies-file ; done
```

Now we can copy this file to /ddpool and see the result:

``` console
root@solaris:~# cp 1000copies-file /ddpool
root@solaris:~# zpool list
NAME      SIZE  ALLOC   FREE  CAP     DEDUP  HEALTH  ALTROOT
ddpool    748M   357K   748M   0%  1000.00x  ONLINE  -
labpool  1.46G  12.4M  1.45G   0%     4.00x  ONLINE  -
rpool    15.6G  7.91G  7.72G  50%     1.00x  ONLINE  -
```

How can this help in real life? Imagine you have a policy which requires
creating and storing an archive every day. The archive's content doesn't
change a lot from day to day, but still you have to create it every day.
Most of the blocks in the archive will be identical so it can be
deduplicated very efficiently. Let's demonstrate it using our system's
manual directories.

``` console
root@solaris:~# tar cvf /tmp/archive1.tar /usr/share/man/man1
root@solaris:~# tar cvf /tmp/archive2.tar /usr/share/man/man1 /usr/share/man/man2
```

Clean up our `/ddpool` file system and copy both files there:

``` console
root@solaris:~# rm /ddpool/*
root@solaris:~# cp /tmp/archive* /ddpool
root@solaris:~# zpool list ddpool
NAME    SIZE  ALLOC  FREE  CAP  DEDUP  HEALTH  ALTROOT
ddpool  748M  18.2M  730M   2%  1.90x  ONLINE  -
```

Think about your real life situations where deduplication could help.
Homework exercise: compress both archive files with `gzip`, clean up the
`/ddpool` and copy the compressed files again. Check if it affects
deduplication rate.

