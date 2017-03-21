One of the main new features in Oracle Solaris 11 is a new packaging
system, called IPS (Image Packaging System). In this lab we will explore
its capabilities and learn how to work with packages from System
Administrator's perspective.

IPS uses a network-based repository model which is also used by modern Linux distributions (`yum/rpm` in Red Hat Enterprise Linux and Oracle Linux, `apt` in Debian and Ubuntu, etc.). All the information about package and its dependencies is stored in the package's metadata. IPS is responsible for packages' and overall system's integrity, so it's not a sysadmin's headache anymore. 

Another good news for sysadmins is that in IPS there are no "patches". No more long README files, no more guessing which patch to install first. Everything is done by a single command (`pkg update`) and IPS takes care of everything.

You can read more about IPS concepts in the official [Oracle Solaris 11 documentation](https://docs.oracle.com/cd/E53394_01/html/E54739/ghyer.html)

In this lab we will address the following typical situations.


|Situation | Exercise|
|----------|---------|
You have an ISO file with Solaris 11 repository and you have to install it and share for your local network|[IPS Repository](ips_repo.md)
You need a package and you want to find it in the repository and install it|[IPS Packages](ips_install.md)
You want to make sure that your packages weren't changed (maliciously or by mistake)|[IPS Verify](ips_verify.md)
You want to install a new package, but in a separate Boot Environment|[IPS and BE](ips_be.md)
You have in-house development and you want to use Solaris mechanism to install and update your local applications|[IPS for Developers](ips_dev.md)  


After you are done with the exercises you can check you IPS history:

``` console
root@solaris:~# pkg history
```

And the verbose history:

``` console
root@solaris:~# pkg history -l|more
```

This might be helpful if you have a team of sysadmins and you want to know what was done before you.

