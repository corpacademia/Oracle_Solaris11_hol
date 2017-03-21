Boot Environments Intro
-----------------------

Boot Environments feature helps you to have several separate environments with different Solaris versions (e.g. 11.2.7, 11.3.12. etc.), different package versions, different system configurations. Typically they are used to update Solaris systems (most of the work is done by IPS behind the scenes, but you can learn the mechanics of the process in the [IPS Lab](ips/ips.md).

Boot environments use ZFS snapshot/clone technology so they can be created quickly and they don't consume storage space until you make changes. You can have as many Boot Environments as you want and keep different system configurations in them. By default, every time you update Solaris OS, a new boot environment is created (you can specify its name).  


Protection from Human Errors
-------------------------------

**Task:** You want to make updates to your system, but you want to be
able to return back to the previous state.

**Lab:** We will use a Solaris 11 feature called Boot Environments.
We'll create an extra boot environment as a backup (think saving your
state in a shooting game). Then we'll make some fatal mistakes which
make our system unbootable. After we failed to boot our default boot
environment, we'll boot into the backup BE.

The only command you want to know to work with Boot Environments is
`beadm(1M)`. Start with showing all boot environments in the system:

``` console
root@solaris:~# beadm list
BE      Active Mountpoint Space Policy Created          
--      ------ ---------- ----- ------ -------          
solaris NR     /          6.87G static 2011-11-14 13:13 
```

Create a new boot environment:

``` console
root@solaris:~# beadm create solaris-backup
```

Check the status again:

``` console
root@solaris:~# beadm list
BE          Active Mountpoint Space   Policy Created          
--          ------ ---------- -----   ------ -------          
solaris       NR    /          7.13G  static 2011-11-14 13:13 
solaris-backup      -        126.14M  static 2011-11-15 11:09 
```

Now pretend you are making some changes in the system configuration,
creating and removing directories, and... Somebody has distracted you
and instead of removing a temporary directory, you have typed:

``` console
root@solaris:~# rm -rf /usr 
```

What??? You just have destroyed the whole `/usr` directory! You have
killed your system! Try `ls` or `ps` or any normal command. What do
you see? Even if you try to reboot your system using `init 6`, the
result is the same: `Command not found`. Try to "Power Off" your virtual
machine and reboot it&mdash;you'll see it won't boot. Don't wait too long,
3&ndash;5 minutes is enough to be sure that it doesn't boot. Hint: While
waiting for the system to reboot, press the <kbd>ESC</kbd> key to see the error
messages.

No worries! We have a backup! And you don't have to go and find the
backup tape in your fireproof cabinet, go through all the hassles of
restoring the unbootable system... Just when your VirtualBox VM shows
you the GRUB menu, choose the `solaris-backup` item instead of the
`Oracle Solaris 11.3` which is the default.


!!! note "SPARC version" 
        If you are using a SPARC system for this lab (for
        example, in Oracle Solutions Center we use logical domains for that)
        then you need console access to be able to choose a different Boot
        Environment. In the case of LDoms, you have to login in the Control
        Domain and stop your guest domain with `ldm stop lab0` (or whatever
        domain name you are using). Then make sure the domain will not
        auto-boot: `ldm set-var auto-boot\?=false lab0` and start the domain
        again: `ldm start lab0`. Now you can access the domain's console via
        `telnet localhost 5000`. You will see the familiar `ok` prompt. Then
        use `boot -L` to list existing Boot Environments.

        ``` console
        {0} ok boot -L
        Boot device: /virtual-devices@100/channel-devices@200/disk@0:a  File and args: -L
        1 solaris
        2 solaris-backup
        Select environment to boot: [ 1 - 2 ]: 2

        To boot the selected entry, invoke:
        boot [] -Z rpool/ROOT/solaris-backup

        Program terminated
        {0} ok boot -Z rpool/ROOT/solaris-backup
        ```


It boots again! What a relief!

What now? List your boot environments again:

``` console
root@solaris:~# beadm list
BE          Active Mountpoint Space   Policy Created          
--          ------ ---------- -----   ------ -------          
solaris        R    /          7.13G  static 2011-11-14 13:13 
solaris-backup N      -        126.14M  static 2011-11-15 11:09 
```

What does it mean? It means that you are now running `solaris-backup`
boot environment, but at the next reboot the system will try to boot the
`solaris` boot environment, which is corrupted. Let's fix that. The plan
is: activate your current BE, delete the `solaris` BE (as it's corrupted
anyway), create a new `solaris` BE and activate it.

``` console
root@solaris:~# beadm activate solaris-backup
root@solaris:~# beadm destroy solaris
root@solaris:~# beadm create solaris
root@solaris:~# beadm activate solaris
```

Check your work:

``` console
root@solaris:~# beadm list
BE             Active Mountpoint Space  Policy Created          
--             ------ ---------- -----  ------ -------          
solaris        R      -          6.05G  static 2012-11-14 16:35 
solaris-backup N      /          275.5K static 2012-11-14 16:22 
```

Now you've got everything back. You can safely reboot into `solaris` BE
or you can continue working in `solaris-backup` until next reboot.

Boot Environments have many applications: you can update your system,
installing packages into inactive boot environment; create boot
environment snapshots etc. We leave it for your homework.

