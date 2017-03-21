**Task:** Your departments want to know how much resources do they use
to pay their fair share for the datacenter infrastructure.

**Lab:** Some familiar Solaris commands now include a `-Z` parameter to
help you to monitor zones behavior. Try `ps -efZ` and `prstat -Z` to take a
look. Try also a new command `zonestat(1)` to show zone statistics.

``` console
root@solaris:~# zonestat -z zone1,zone2 5 

Collecting data for first interval...
Interval: 1, Duration: 0:00:05
SUMMARY                   Cpus/Online: 1/1   PhysMem: 2047M  VirtMem: 3071M
                    ---CPU----  --PhysMem-- --VirtMem-- --PhysNet--
               ZONE  USED %PART  USED %USED  USED %USED PBYTE %PUSE
            [total]  0.05 5.45%  968M 47.3% 1251M 40.7%     0 0.00%
           [system]  0.01 1.51%  287M 14.0%  735M 23.9%     -     -
              zone1  0.00 0.16% 73.8M 3.60% 66.3M 2.16%     0 0.00%
              zone2  0.00 0.13% 73.9M 3.61% 67.2M 2.18%     0 0.00%
```

Note the parameters you can observe with `zonestat`: CPU utilization,
physical and virtual memory usage, network bandwidth utilization.

