**Task:** You have to install some application packages in the zone and
create users.

**Lab:** Log in in the zone, create a user and install a web server
application.

``` console
root@solaris:~# zlogin zone1 
root@zone1:~# 
```

Play around with the usual sysadmin commands. How can you tell if you
are in a zone or not? First, try `ps -ef`. Do you see anything unusual?
Yes, you are right, the process IDs don't start with 0, but with some
big number. Other than that, no visible difference between the normal
Solaris installation and the zone. Try `uname -a`, `psrinfo`, `cat
/etc/release`... Check if you can access the Internet by pinging
`oracle.com`.

Now let's do something useful with the zone. Like running a web server,
for example. Let's install and run Apache.

``` console
root@zone1:~# pkg list -a *apache* 
. . .Skipped. . . 
web/server/apache-22 2.2.22-0.175.1.0.0.24.0  ---. 
. .Skipped. . . 
root@zone1:~# pkg install apache-22 
. . .Skipped. . . 
```

We've installed it successfully, but it's not running yet.

``` console
root@zone1:~# svcs -a | grep apache 
disabled 6:31:42 svc:/network/http:apache22 
```

Start the Apache web server:

``` console
root@zone1:~# svcadm enable apache22 
root@zone1:~# svcs -a | grep apache 
online 6:34:03 svc:/network/http:apache22 
```

Check if it's working from your global Solaris zone (your Solaris
desktop): start Firefox and enter your zone's IP address into the URL
field: `10.0.2.21`. **It works!** -- the page usually reads. 

Check if it's your zone who is talking. Go back to the zone's terminal
window and change your web server homepage (I'm using `vi` here, as we
don't have many choices in a freshly installed zone. If you are not
familiar with `vi`, check our Vi Quick Reference below):

``` console
root@zone1:~# vi /var/apache2/2.2/htdocs/index.html 
```

Write here something like "This is Zone1 and it works!" and save the
file. Make sure you use `w!` (with the exclamation sign) to save the
read-only file. Now reload the page in Firefox in your Solaris desktop.
Did it work? Congratulations!


!!! tip "Vi Quick Reference"
    If you're unfamiliar with vi, following are a few common
    keyboard commands to get you through this exercise:  
    i = switch to Insert mode  
    Use Insert mode to type in your text.  
    Esc = switch to Command mode  
    In Command mode use:  
    k = up  
    j = down  
    w = right or forward one word  
    b = left or back one word  
    l = right 1 char  
    h = left 1 char  
    x = delete 1 char  
    u = undo  
    dd = delete entire current line  
    :w = write (save) the current file  
    :wq = write and quit  
    :w! = write to a read-only file  
    :q! = quite ignoring changes (do not write)  

What else do we need? Try to create users in the zone.

``` console
root@zone1:~# useradd -m jack 
root@zone1:~# passwd jack
New Password: oracle1 (will not be displayed) 
Re-enter new Password: oracle1 (will not be displayed) 
passwd: password successfully changed for jack 
root@zone1:~# su - jack 
Oracle Corporation  SunOS 5.11  11.0    November 2011
jack@zone1:~$ ls
local.cshrc    local.login    local.profile
jack@zone1:~$ 
```

Looks good! Try to login from your global zone (open another window on
your Solaris desktop):

``` console
lab@solaris:~$ ssh -l jack 10.0.2.21
```

(It's a small letter L here, not the digit 'one')

Exit from the `ssh` session and return back to the global zone. Let's
see how zones look from the global zone's perspective. From here you can
watch processes in non-global zones by using `-Z` command line argument in
`ps(1)`. Try this:

``` console
root@solaris:~# ps -efZ
.....Skipped long output...
zone1     root  4807     1   0 11:47:33 ?           0:00 /usr/lib/ssh/sshd
zone1     root  4132     1   0 11:47:13 ?           0:00 /usr/lib/rad/rad -sp
zone1     root  4736     1   0 11:47:30 ?           0:00 /usr/lib/autofs/automountd
zone1     root  4737  4736   0 11:47:30 ?           0:00 /usr/lib/autofs/automountd
global     root  4921  1636   0 12:25:04 pts/1       0:00 ps -efZ
zone1     root  4869     1   0 11:47:37 ?           0:00 /sbin/dhcpagent
```

All processes are tagged with a zone name: it's either `zone1` or
`global`. Remember to try this command again when you have more zones
running (in our next exercises).

Now login again into the zone and try '`ps -efZ`' inside it. Check if
you can see global zone processes from inside the zone. Remember to
check this again when you have more zones running.

You may also try the `prstat(1M)` command with `-Z` argument and see
what happens.

For your homework: compare global and non-global zones installations.
How many packages are installed in both?How many services are running?
Check if you can login into the global zone with the zone user's (`jack`)
credentials. Check if you can use your zone's root password in the
global zone (of course, if they are different).

