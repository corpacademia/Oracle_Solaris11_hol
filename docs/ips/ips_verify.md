Show how we can verify a package has not been compromised:

``` console
root@solaris:~# pkg verify iperf
```

Let's check the permission bits of `iperf`:

``` console
root@solaris:~# ls -l /bin/iperf
```

Let's say someone changed the permissions bits like this:

``` console
root@solaris:~# chmod 777 /bin/iperf
```

Let's verify again, and it will return an error:

``` console
root@solaris:~# pkg verify iperf
```

Now let's fix this package:

``` console
root@solaris:~# pkg fix iperf
```

The next few lines show an interesting way of getting a package name
from a arbitrary file. Let's take `vi` editor for test. We first get a
SHA1 digest of `/usr/bin/vi`

``` console
root@solaris:~# digest -a sha1 /usr/bin/vi
```

Since the digest is saved in the package DB, we can search for that hash
and see what matches it:

``` console
root@solaris:~# pkg search -l f2495fa19fcc4b8a403e0bd4fef809d031296c68
```

How can we use it? Imagine someone has renamed some important file to
hide his tracks. Using this method we can find out the original name of
the file.

