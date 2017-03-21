Another important feature of IPS is the ability to perform all package
related operations not only on the current boot environment, but also on
a mounted one. Imagine you want to update your system, but you want to
keep your current state untouched to be able to return back to safety in
case something goes wrong. Also you want to minimize the downtime.

Create a new boot environment for the updated system:

``` console
root@solaris:~# beadm create solaris-updated
```

In real life you might want to use some naming policy for the BEs, like
timestamping them.

Now mount this boot environment in your file system:

``` console
root@solaris:~# beadm mount solaris-updated /mnt
```

Now you can perform any package operations with this mounted boot
environment. As we don't have updates in our repository, we just install
a package (the same iperf package), check that it's not available in our
current BE and then reboot the system with the updated BE and make sure
the package is installed there.

``` console
root@solaris:~# pkg -R /mnt install iperf
```

Now make sure the new boot environment is active on reboot:

``` console
root@solaris:~# beadm activate solaris-updated
```

Then reboot the system and check if `iperf` is available.

Imagine how much downtime you can save when using this method to update
your systems!

