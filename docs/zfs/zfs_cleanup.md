After we have finished this ZFS Lab, let's clean up what we have created
so far. After all, our ZFS skill set won't be complete without knowing
how to destroy ZFS pools.

``` console
root@solaris:~# zpool destroy ddpool
root@solaris:~# zpool destroy labpool
root@solaris:~# zpool destroy backuppool
```

Take a note how easy it is to destroy a ZFS pool. Use extreme care when
using the "destroy" command.
