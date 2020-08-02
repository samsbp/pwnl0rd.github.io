---
title: "Understanding-Linux: Process"
date: 2020-07-28T23:31:48+05:30
Description: "Understanding Linux: Linux Processes"
Tags: [linux,kernel,unix,process]
Categories: [linux,kernel,unix,process]
<!--draft: True-->
---

![Main Image](/img/main/linux-process/main.png)

  Hi Y'ALL, glad to meet you in this blog post. This post will cover the basic concepts on process mainly in linux system and we will have an hands-on with linux process and how it works. Understanding the concepts of  Operating System will help in developing robust & perfomance-oriented applications. It also helps in penetesting (cybersecurity).

## What is a process?
  I hate definition but no other go buddy :) . Process is a program in execution. Then you may have a question, what is a program?. In this context, a program is an executable file. In every operating system, there is a format for a file to be  executable.
  - Windows - [PE format](https://en.wikipedia.org/wiki/Portable_Executable)
  - Linux - [ELF format](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)
  - Mac - [MACH-O](https://en.wikipedia.org/wiki/Mach-O)

The executable files should be in one of the format, respective to the operating system.

## Parent & Child

Process ID aka PID is used to identify the running process. For example , in a linux machine, `ps` is the command used to find all the running processes.
```
pwn@ubuntu:~$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.4 169020  8496 ?        Ss   Aug01   0:14 /sbin/init auto noprompt
root           2  0.0  0.0      0     0 ?        S    Aug01   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        I<   Aug01   0:00 [rcu_gp]
root           4  0.0  0.0      0     0 ?        I<   Aug01   0:00 [rcu_par_gp]
root           6  0.0  0.0      0     0 ?        I<   Aug01   0:00 [kworker/0:0H-kblockd]
root           9  0.0  0.0      0     0 ?        I<   Aug01   0:00 [mm_percpu_wq]
root          10  0.0  0.0      0     0 ?        S    Aug01   0:02 [ksoftirqd/0]
root          11  0.0  0.0      0     0 ?        I    Aug01   0:04 [rcu_sched]
root          12  0.0  0.0      0     0 ?        S    Aug01   0:00 [migration/0]
root          13  0.0  0.0      0     0 ?        S    Aug01   0:00 [idle_inject/0]
root          14  0.0  0.0      0     0 ?        S    Aug01   0:00 [cpuhp/0]
root          15  0.0  0.0      0     0 ?        S    Aug01   0:00 [kdevtmpfs]
root          16  0.0  0.0      0     0 ?        I<   Aug01   0:00 [netns]
root          17  0.0  0.0      0     0 ?        S    Aug01   0:00 [rcu_tasks_kthre]
root          18  0.0  0.0      0     0 ?        S    Aug01   0:00 [kauditd]
root          19  0.0  0.0      0     0 ?        S    Aug01   0:00 [khungtaskd]
root          20  0.0  0.0      0     0 ?        S    Aug01   0:00 [oom_reaper]
root          21  0.0  0.0      0     0 ?        I<   Aug01   0:00 [writeback]
root          22  0.0  0.0      0     0 ?        S    Aug01   0:01 [kcompactd0]
root          23  0.0  0.0      0     0 ?        SN   Aug01   0:00 [ksmd]
root          24  0.0  0.0      0     0 ?        SN   Aug01   0:00 [khugepaged]
root         116  0.0  0.0      0     0 ?        I<   Aug01   0:00 [kintegrityd]
root         117  0.0  0.0      0     0 ?        I<   Aug01   0:00 [kblockd]
root         118  0.0  0.0      0     0 ?        I<   Aug01   0:00 [blkcg_punt_bio]
root         119  0.0  0.0      0     0 ?        I<   Aug01   0:00 [tpm_dev_wq]
root         120  0.0  0.0      0     0 ?        I<   Aug01   0:00 [ata_sff]
root         121  0.0  0.0      0     0 ?        I<   Aug01   0:00 [md]
root         122  0.0  0.0      0     0 ?        I<   Aug01   0:00 [edac-poller]
root         123  0.0  0.0      0     0 ?        I<   Aug01   0:00 [devfreq_wq]
root         124  0.0  0.0      0     0 ?        S    Aug01   0:00 [watchdogd]
root         127  0.0  0.0      0     0 ?        S    Aug01   0:07 [kswapd0]
root         128  0.0  0.0      0     0 ?        S    Aug01   0:00 [ecryptfs-kthrea]
root         131  0.0  0.0      0     0 ?        I<   Aug01   0:00 [kthrotld]
..............................................................................................
```

I have just mentioned a few process in the ps command output above but we will get a lot. Whoa wait!, How does these many process are there in the first place?. For that, we will go to its root. When you boot into your linux, the kernel is the one first to be loaded, after finishing all the loadings of drivers and kernel modules it will spawn the systemd(or init) process. The systemd process is the first process that will be executed. The systemd process will start to run all the configured daemons(usually process names ends with d, example : ssh -> sshd ) by creating a child process from systemd process for individual daemons. Well, we can visualize this process forking as a tree.

`pstree` - process visualisation tool

```
pwn@ubuntu:~$ pstree -p
systemd(1)─┬─ModemManager(747)─┬─{ModemManager}(761)
           │                   └─{ModemManager}(765)
           ├─NetworkManager(670)─┬─{NetworkManager}(718)
           │                     └─{NetworkManager}(736)
           ├─VGAuthService(1959)
           ├─accounts-daemon(657)─┬─{accounts-daemon}(663)
           │                      └─{accounts-daemon}(732)
           ├─acpid(658)
           ├─avahi-daemon(661)───avahi-daemon(719)
           ├─bluetoothd(662)
           ├─colord(4313)─┬─{colord}(4314)
           │              └─{colord}(4316)
           ├─containerd(71023)─┬─containerd-shim(74834)─┬─sh(74860)
           │                   │                        ├─{containerd-shim}(74835)
           │                   │                       ├─{containerd-shim}(74836)
           │                   │                        ├─{containerd-shim}(74837)
           │                   │                        ├─{containerd-shim}(74838)
           │                   │                        ├─{containerd-shim}(74839)
           │                   │                        ├─{containerd-shim}(74840)
           │                   │                        ├─{containerd-shim}(74841)
           │                   │                        ├─{containerd-shim}(74843)
           │                   │                        └─{containerd-shim}(74883)
           │                   ├─{containerd}(71031)
           │                   ├─{containerd}(71032)
           │                   ├─{containerd}(71033)
           │                   ├─{containerd}(71034)
           │                   ├─{containerd}(71035)
           │                   ├─{containerd}(71037)
           │                   ├─{containerd}(71040)
           │                   ├─{containerd}(71041)
           │                   └─{containerd}(74696)
           ├─cron(667)
           ├─cups-browsed(69655)─┬─{cups-browsed}(69722)
           │                     └─{cups-browsed}(69723)
           ├─cupsd(69648)
           ├─dbus-daemon(669)
           ├─dockerd(73293)─┬─{dockerd}(73305)
           │                ├─{dockerd}(73306)
           │                ├─{dockerd}(73307)
           │                ├─{dockerd}(73308)
           │                ├─{dockerd}(73313)
           │                ├─{dockerd}(73314)
           │                ├─{dockerd}(73318)
           │                ├─{dockerd}(74542)
           │                └─{dockerd}(74543)
           ├─gdm3(3982)─┬─gdm-session-wor(4451)─┬─gdm-x-session(4516)─┬─Xorg(4525)───{Xorg}(4565)
           │            │                       │                     ├─gnome-session-b(4568)─┬─ssh-agent(4662)
           │            │                       │                     │                       ├─{gnome-session-b}(4682)
           │            │                       │                     │                       └─{gnome-session-b}(4683)
           │            │                       │                     ├─{gdm-x-session}(4524)
           │            │                       │                     └─{gdm-x-session}(4564)
           │            │                       ├─{gdm-session-wor}(4452)
           │            │                       └─{gdm-session-wor}(4453)
           │            ├─{gdm3}(3983)
           │            └─{gdm3}(3984)
           ├─gnome-keyring-d(4475)─┬─{gnome-keyring-d}(4476)
           │                       ├─{gnome-keyring-d}(4477)
           │                       └─{gnome-keyring-d}(4736)
           ├─ibus-daemon(4680)─┬─ibus-engine-sim(4769)─┬─{ibus-engine-sim}(4770)
           │                   │                       └─{ibus-engine-sim}(4772)
           │                   ├─ibus-extension-(4702)─┬─{ibus-extension-}(4717)
           │                   │                       ├─{ibus-extension-}(4718)
           │                   │                       └─{ibus-extension-}(4783)
           │                   ├─ibus-memconf(4695)─┬─{ibus-memconf}(4701)
           │                   │                    └─{ibus-memconf}(4705)
           │                   ├─ibus-ui-gtk3(4696)─┬─{ibus-ui-gtk3}(4721)
           │                   │                    ├─{ibus-ui-gtk3}(4722)
           │                   │                    └─{ibus-ui-gtk3}(4726)
           │                   ├─{ibus-daemon}(4689)
           │                   └─{ibus-daemon}(4690)
           ├─ibus-x11(4706)─┬─{ibus-x11}(4713)
           │                └─{ibus-x11}(4714)
           ├─kerneloops(829)
           ├─kerneloops(836)
           ├─networkd-dispat(689)
           ├─polkitd(690)─┬─{polkitd}(694)
           │              └─{polkitd}(733)
           ├─rsyslogd(696)─┬─{rsyslogd}(720)
           │               ├─{rsyslogd}(721)
           │               └─{rsyslogd}(722)
           ├─rtkit-daemon(4072)─┬─{rtkit-daemon}(4073)
           │                    └─{rtkit-daemon}(4074)
           ├─snapd(59489)─┬─{snapd}(59498)
           │              ├─{snapd}(59499)
           │              ├─{snapd}(59500)
           │              ├─{snapd}(59501)
           │              ├─{snapd}(59505)
           │              ├─{snapd}(59506)
           │              ├─{snapd}(59507)
           │              ├─{snapd}(59560)
           │              └─{snapd}(60102)
           ├─switcheroo-cont(701)─┬─{switcheroo-cont}(712)
           │                      └─{switcheroo-cont}(734)

           .............................................................................
```

As you can see from the ouput of pstree, the systemd is the first process that has pid 1 (but i told lies ,haha! there will be a process with pid 0 that takes care of process scheduling so for now we dont consider the pid 0 process) and then we have ModemManager(747),NetworkManager(670),accounts-daemon(657), etc are child processes of systemd and these child processes will create other child processes also. If we see bash i.e linux terminal, it will have the following visualisation

```
pwn@ubuntu:~$ pstree | grep bash
        |         |-gnome-terminal--+-bash---bash-+-grep

```

We can see the flow for bash process as systemd->gnome-terminal->bash->grep(the command that is executed , the grep part of pstree).
And there comes the concept of forking.


### fork vs exec
There are two significant syscalls(syscalls are the one that connects userspace program and the kernel), [fork](https://linux.die.net/man/2/fork) and [exec](https://man7.org/linux/man-pages/man3/exec.3.html) to create child process. The fork syscall will create a clone of the parent process and both, the parent and child, will have the same virtual address space whereas in exec the parent process will be replaced by the child process and parent process will get control only after child process finishes its execution. 

example for fork call
```
pwn@ubuntu:~$ cat forkpython.py 
import os

def child():
   print('\nA new child ',  os.getpid())
   input("in child")
   os._exit(0)  

def parent():
   while True:
      newpid = os.fork()
      if newpid == 0:
         child()
      else:
         pids = (os.getpid(), newpid)
         print("parent: %d, child: %d\n" % pids)
      reply = input("q for quit / c for new fork")
      if reply == 'c': 
          continue
      else:
          break

parent()
pwn@ubuntu:~$ python3 forkpython.py 
parent: 82224, child: 82225

q for quit / c for new fork
A new child  82225
in child

```
To see the virtual address space, we can use procfs information. Find the process id of both parent and child using `ps` command. Use diff to see any changes in virtual address space
```
pwn@ubuntu:~$ ps aux | grep forkpython.py
pwn        82224  0.0  0.4  26092  8628 pts/2    S+   09:36   0:00 python3 forkpython.py
pwn        82225  0.0  0.3  26092  6300 pts/2    S+   09:36   0:00 python3 forkpython.py
pwn        82263  0.0  0.0  17668   732 pts/5    S+   09:40   0:00 grep --color=auto forkpython.py
pwn@ubuntu:~$ diff /proc/82224/maps /proc/82225/maps
pwn@ubuntu:~$ 
```

As you can see, there are two process that runs `python3 forkpython.py` with PID's 82224 and 82225. In this, one of them will be parent and other will be child. The /proc/PID/maps pseudo file will contain the memory map for the process that is what are all the memory regions are used by the process. And there is no output for the `diff` command, because both the parent and the child will have the same memory map.

Let's see in the case of exec,

```
pwn@ubuntu:~$ cat execpython.py 
#!/usr/bin/python
import os
args=['']
input("press enter")
os.execvp("/bin/sh",args)
pwn@ubuntu:~$ python execpython.py 
press enter

```
In new tab, execute ps 
```
pwn@ubuntu:~$ ps aux | grep execpython.py
pwn        82461  0.0  0.3  24308  6912 pts/2    S+   09:50   0:00 python execpython.py
pwn        82486  0.0  0.0  17668   732 pts/1    S+   09:51   0:00 grep --color=auto execpython.py
```

The pid for the process is 82461 . Now press enter in the previous tab where the program was executed and run the tail command and see the ps command output
```
pwn@ubuntu:~$ python execpython.py 
press enter
$ tail -f /etc/hosts
127.0.0.1	localhost
127.0.1.1	ubuntu

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
_
```

```
pwn@ubuntu:~$ ps aux | grep 82461
pwn        82461 0.0  0.0  16744   520 pts/2    S+   09:55   0:00 tail -f /etc/hosts
pwn        82572  0.0  0.0  17664   732 pts/1    S+   09:56   0:00 grep --color=auto 82461
```

Now you can see that the previous `python execpython.py` has been replace by `tail` command but having the same PID. So , exec does not create new process but it overwrites the old process with the new one. 

## PCB
Process Control Blocks aka PCB is a kernel data structure that maintains information on the processes that is of any state (state can be running,queued etc). Some of the data structures of the PCB are accessible through procfs. procfs is a pesudo file system. Below explains the procfs.

procfs can be found at /proc directory. To find the information about the process, we should find its PID from `ps` command.

For more read the man pages [procfs](https://man7.org/linux/man-pages/man5/procfs.5.html)
