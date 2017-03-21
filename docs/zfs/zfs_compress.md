**Task:** You are getting low on your disk space. Now you know how to
add more disks to your pool and expand your file system. What other ZFS
features can help you to solve this problem?

**Lab:** In our lab we will compress our Solaris manuals directory and
see if we are able to use it after that. Create a separate filesystem
for this on our 'labpool' ZFS pool:

``` console
root@solaris:~# zfs create labpool/zman 
root@solaris:~# zfs list | grep zman 
labpool/zman                     44.9K   1.06G  44.9K  /labpool/zman
```

Set compression to "gzip" (there are options to gzip and other
algorithms too--check the manual). You can do that also while creating
the filesystem.

``` console
root@solaris:~# zfs set compression=gzip labpool/zman 
```

Copy the first part of Solaris manuals there (it will take some time, be
patient):

``` console
root@solaris:~# cp -rp /usr/share/man/man1 /labpool/zman/ 
```

Compare the sizes:

``` console
root@solaris:~# du -sh /usr/share/man/man1 /labpool/zman/man1
13M   /usr/share/man/man1
5.6M   /labpool/zman/man1
``` 


We just have saved about 57% of disk space. Not bad! Check if you are
able to use the manuals after compression:

``` console
root@solaris:~# export MANPATH=/labpool/zman ; man ls 
```

Interesting to note: it may sound counterintuitive, but using
compression actually *increases* file system's performance. You may
think: "Compression uses extra CPU time, so it should slow down file
system operations, right?". But try to think further. Imagine a file
that takes two blocks on your disk. To write this file you have to write
two blocks, right? If you compress this file by 50% you have to write
only one block. Now the question is: "What is faster, your disk or your
CPU?". Of course, it takes much less time to compress a block of data
than to write it on the disk. OK, it's easy to explain, but is it
confirmed by practice? Yes, it is! Take a look at the blog of Don
MacAsksill and see how he had confirmed that ZFS compression increases
performance:
<http://don.blogs.smugmug.com/2008/10/13/zfs-mysqlinnodb-compression-update/>.
Note that it works best when you use the default LZJB algorithm by using
plain "compression=on" parameter. You might consider it a good default
practice when creating ZFS file systems. There are exceptions, of
course: image, video, encrypted and already compressed data will not
give you this advantage as they will not be compressed.

