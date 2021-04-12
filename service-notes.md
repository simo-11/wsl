# Service notes

## Introduction

Notes on required services and logging for running wsl.

Initial focus was on debugging failure to start wsl. Issue was resolved partially by resetting windows which lead to losing admin rights and update of laptop.

Tables using https://www.tablesgenerator.com/markdown_tables

### Reproduce
```
C:\Windows\System32\lxss\tools>bash -c"echo yes"
The system cannot find the file specified.

$ date; time /cygdrive/c/Windows/System32/bash -c"echo yes"; date;stat /cygdrive/c/Windows/{SysWOW64,System32}/NetCfgNotifyObjectHost.exe
Fri Apr  2 13:55:39 EEST 2021
The system cannot find the file specified.

real    0m53.733s
user    0m0.796s
sys     0m1.734s
Fri Apr  2 13:56:34 EEST 2021
  File: /cygdrive/c/Windows/SysWOW64/NetCfgNotifyObjectHost.exe
  Size: 59904           Blocks: 60         IO Block: 65536  regular file
Device: 60a907f4h/1621690356d   Inode: 1970324837706267  Links: 2
Access: (0750/-rwxr-x---)  Uid: (328384/NT SERVICE+TrustedInstaller)   Gid: (328384/NT SERVICE+TrustedInstaller)
Access: 2021-04-02 13:56:34.493813400 +0300
Modify: 2019-12-07 11:09:30.521077800 +0200
Change: 2021-03-26 21:47:52.351471300 +0200
 Birth: 2019-12-07 11:09:30.521077800 +0200
  File: /cygdrive/c/Windows/System32/NetCfgNotifyObjectHost.exe
  Size: 75776           Blocks: 76         IO Block: 65536  regular file
Device: 60a907f4h/1621690356d   Inode: 2533274791079852  Links: 2
Access: (0750/-rwxr-x---)  Uid: (328384/NT SERVICE+TrustedInstaller)   Gid: (328384/NT SERVICE+TrustedInstaller)
Access: 2021-04-02 13:56:34.595820800 +0300
Modify: 2019-12-07 11:09:05.755385600 +0200
Change: 2021-04-01 21:31:40.213649800 +0300
 Birth: 2019-12-07 11:09:05.755385600 +0200
```
##### More details using Process Monitor
 * CommandLine e.g. c:\windows\system32\NetCfgNotifyObjectHost.exe {EAD21270-4FC7-4D17-B829-43227B05C605} 688
 * PID 27092, PPID 17488
 * Start 13.55.48 End 13.55.49

![image](https://user-images.githubusercontent.com/1210784/113411648-06f9c180-93bf-11eb-984c-e853e36540fa.png)

13.55.49.3567623	NetCfgNotifyObjectHost.exe	0x69d4	Process Exit		SUCCESS	Exit Status: 0, User Time: 0.0312500 seconds, Kernel Time: 0.0468750 seconds, Private Bytes: 1 843 200, Peak Private Bytes: 1 929 216, Working Set: 9 502 720, Peak Working Set: 9 506 816

##### For run 14:39:14-14:41:01 
![image](https://user-images.githubusercontent.com/1210784/113446339-4b598180-9400-11eb-96b4-e0ec6b6a5c6e.png)
 * cygwin64/bin/bash 8012
 * system32/bash 26904
 * wsl 17136 14:39:16.1305-14:40:59.328 exit status -1

 * wmp.exe, parent is vmcompute.exe  runtime 14:39:16-14:41:01
   * "C:\WINDOWS\System32\vmwp.exe" AF4725A6-475C-4541-B336-37C4CFE481AB 0x3d8 2.4.2021 14.39.17.8889061
   * 14.41.01.3108619	vmwp.exe	25480	Process Exit		SUCCESS	Exit Status: 0, User Time: 0.0937500 seconds, Kernel Time: 1.8281250 seconds, Private Bytes: 5 033 984, Peak Private Bytes: 8 331 264, Working Set: 13 864 960, Peak Working Set: 43 360 256
 * svchost.exe  PID 24224 Parent PID:	692
   * Command line:	C:\WINDOWS\System32\svchost.exe -k netsvcs -p -s NetSetupSvc 
 * NetCfgNotifyObjectHost.exe multiple processes

### Notes
LxssManager and vmcompute start but WSL network driver does not start.
NetCfgNotifyObjectHost.exe starts many times (>300) during process

Modifying dns configuration for Ethernet driver
![image](https://user-images.githubusercontent.com/1210784/113473314-9cee2480-9471-11eb-9326-f18291ed84d0.png)
causes error.
![image](https://user-images.githubusercontent.com/1210784/113473288-762fee00-9471-11eb-8afb-666ab7e90c7a.png)



# Normally running system

## Related services

| Service  |Before starting bash   | After starting bash | After stoppig bash |
|---|---|---|---|
|LxssManager   | stopped  | running  |   |
|vmcompute |  stopped | running   |   |
|LxssManagerUser   | stopped  | stopped   |   |
|LxssManagerUser_[a-z0-9]{5}   | stopped  | stopped   |   |

## Related Network Drivers
Hyper-V Extensible Virtual Switch is not used in WSL nor Ethernet driver
### vEthernet (WSL)
IP e.g. 172.24.112.1/255.255.240.0
