



PRESENTATION
- docker made possible by cgroups and namespaces
- demo


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





