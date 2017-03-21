**Task:** You want to find and install a package from Solaris
repository.

**Lab:** We will learn some basic IPS commands used to look for a
package, inquire about its content etc. When we found the package we
needed, we install it.

Let's imagine we want to do some network load testing, so we want to run
the utility called `iperf`. Try to run this command and find out that
it's not installed (don't be surprised by `command not found` message:

``` console
root@solaris:~# iperf
```

So the first thing we do is show our current publisher. The publisher is
where the IPS repository is located. It can be a local directory, an NFS
mount point, an internal http ot https server, or an Internet repository
like http://pkg.oracle.com/solaris. You system can have several
publishers configured.

``` console
root@solaris:~# pkg publisher
```

OK, it seems we are going to use our local publisher installed in our
Solaris system.

Now we list our installed packages

``` console
root@solaris:~# pkg list | more
```

Let's see how many packages are installed

``` console
root@solaris:~# pkg list | wc
```

Now let's see how many packages are available in the repository

``` console
root@solaris:~# pkg list -a | wc
```

As you can see, our local repository is pretty small, it contains only
the packages we need for this lab. If you have changed the publisher to
http://pkg.oracle.com/solaris/release as it's described in the
introduction, you would see many more packages.

Let's now do a local search for iperf (among the installed packages). It
will find nothing:

``` console
root@solaris:~# pkg search -l iperf
```

You may get the following message after this command:

```
    pkg: Search performance is degraded.
    Run 'pkg rebuild-index' to improve search speed. 
```

So, if you really want to improve search speed, then go ahead and
rebuild the index as instructed. However, it is not critical for the
rest of the lab.

Now do the same search, without the `-l` flag so it goes to the
repository:

``` console
root@solaris:~# pkg search iperf
```

Next we get some information about the package like date of creation,
version number, etc.

``` console
root@solaris:~# pkg info -r iperf
```

And we can see exactly what files make up the package:

``` console
root@solaris:~# pkg contents -r iperf
root@solaris:~# pkg search benchmark/iperf:depend::
```

Now, after we have learned everything about the package we are about to
install, we can try a "dry run" before actually installing it:

``` console
root@solaris:~# pkg install -n iperf
root@solaris:~# pkg install -nv iperf
```

If we are satisfied with the results, we can install the `iperf`
package:

``` console
root@solaris:~# pkg install iperf
```

Here we demonstrate what kind of metadata is kept in IPS:

``` console
root@solaris:~# pkg contents -t file -o owner,group,mode,pkg.size,path iperf
```

Now we run iperf to show it's now found:

``` console
root@solaris:~# iperf
```


