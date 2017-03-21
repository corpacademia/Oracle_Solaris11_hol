You may have to migrate your data to a new location. For instance, you
just have connected a new disk array with really fast disks and you want
to move your data from the old array with slow disks. Or, you may want
to turn on compression on the file system and you know that compression
only works for future data, not for existing data. To make all data
compressed you have to re-write all your data. If your dataset is large,
it may take significant time. You don't want to wait, you want to start
using your data as if it was already in the new location. ZFS Shadow
Migration is created exactly for this situation.

For this exercise we will use our manual pages directory. First, we will
create a separate file system and copy all manual pages there. We'll
change the MANPATH variable to make sure we are using manual files from
that file system. After that, we will create a new file system on our
ZFS pool labpool and configure it for shadow migration. We will change
MANPATH again to point to that new file system and check if we can read
system manuals while the data is being migrated. Shadow migration is
created as a separate package and separate service. To use it, we have
to install the package and enable the service.

``` console
root@solaris:~# pkg install shadow-migration
Packages to install: 1
Create boot environment: No
Create backup boot environment: No
Services to change: 1
DOWNLOAD PKGS FILES XFER (MB) SPEED
Completed 1/1 14/14 0.2/0.2 734k/s
PHASE ITEMS
Installing new actions 39/39
Updating package state database Done
Updating image state Done
Creating fast lookup database Done
root@solaris:~# svcadm enable shadowd
Start with creating a file system and copying the manuals there:
root@solaris:~# zfs create rpool/mancopy
root@solaris:~# cp -rp /usr/share/man/* /rpool/mancopy
Set MANPATH and try to read a manual page:
root@solaris:~# export MANPATH=/rpool/mancopy
root@solaris:~# man ls
Reformatting page. Please Wait... done
```

Hint: If you want to be absolutely sure that the man utility uses your
file system instead of the default one, use this powerful script from
DTrace Toolkit (open another terminal window, become root, run the
following command and then in your first terminal window run `man ls`):

``` console
root@solaris:~# /usr/dtrace/DTT/opensnoop -n man
. . .
0 2785 man 5 /rpool/mancopy/man1/ls.1
. . .
```

Now create a new file system on labpool and set it as a shadow of our
mancopy. Before doing that, change rpool/mancopy to read-only. It's a
requirement for shadow migration.

``` console
root@solaris:~# zfs set readonly=on rpool/mancopy
root@solaris:~# zfs create -o shadow=file:///rpool/mancopy labpool/shadowman
Use the 'shadowstat' command to watch the migration progress.
root@solaris:~# shadowstat
EST
BYTES BYTES ELAPSED
DATASET XFRD LEFT ERRORS TIME
labpool/shadowman 18.8M - - 00:01:10
^Croot@solaris:~#
```

And now, before the migration process has finished (we have a little bit
over 100MB to copy), change MANPATH to the new file system and try 'man
ls' again.

``` console
root@solaris:~# export MANPATH=/labpool/shadowman
root@solaris:~# man ls
```

Again, just to check if you are really accessing the new location, in
another window run the `opensnoop` script. You may want to watch the
process using `shadowstat` until you see `No migrations in progress`.

Interesting to note that the new filesystem (`labpool/shadowman`) is not
read-only, you can use it for reading and writing right after it was
created.

``` console
root@solaris:~# zfs get readonly labpool/shadowman
NAME PROPERTY VALUE SOURCE
labpool/shadowman readonly off default
```

You can read more about ZFS Shadow migration here:
http://www.oracle.com/technetwork/articles/servers-storage-admin/howto-migrate-s11-datashadow-%0A1866521.html   
https://blogs.oracle.com/eschrock/entry/shadow_migration   
https://blogs.oracle.com/eschrock/entry/shadow_migration_internals   
