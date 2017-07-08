# Linux Containers
Linux container technologies like docker and rkt are popular containers engines that have gained a lot of mainstream use. This document is a short guide on how they work and how to make your own basic container.


## Linux containers are made possible by cgroups and namespaces
Namespaces dictate what resources a process can see and have access to. Things like Cgroups, IPC, Networks, Mounts, PIDs, Users, and UTS are the namespaces you can share or make private for new processes. More information on namespaces can be found [here](http://man7.org/linux/man-pages/man7/namespaces.7.html). Namespaces make it possible for processes in a container to feel isolated from the host environment and even vs other containers.

Cgroups dictate how much of a resource that a process has access to. Some of the resources that can be limited include cpu, memory, blkio, etc. More information can be found [here](http://man7.org/linux/man-pages/man7/cgroups.7.html)



make container directory
`mkdir /tmp/container`
`docker export $(docker create alpine) | tar -xvf -`

beginnings of a container
- explain that file system is just a bunch of files and binaries
`run ./bin/sh`

change root
pwd - we are aware of where we are in the filesystem
`sudo chroot /tmp/container /bin/sh`
`pwd`
- docker makes volumes that store the filesystem

not a complete OS yet. no proc directory. can't do things like ps
need a proc directory. proc is a special directory that stores a filesystem of type proc which linux uses to show info on the system
`sudo mount -t proc proc proc/`
- go into container and run ps ... not our ps ids!


namespaces to the rescue
`sudo unshare -p -f --mount-proc=/tmp/container/proc chroot /tmp/container /bin/sh`
- ls proc
- ps
- show linux man pages

cgroups
`sudo mkdir /sys/fs/cgroup/cpu/container`
`echo "2000000" > /sys/fs/cgroup/memory/container/memory.limit_in_bytes`



`sudo unshare -p -f --mount-proc=/tmp/container/proc cgexec -g memory:container chroot /tmp/container /bin/sh`
`mknod -m 444 /dev/urandom c 1 9`
shell script 
```
#!/bin/sh
random="$(dd if=/dev/urandom ibs=20000)"
```





