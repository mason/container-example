# Linux Containers
Linux container technologies like docker and rkt are popular containers engines that have gained a lot of mainstream use. This document is a short guide on how they work and how to make your own basic container.


## Linux containers are made possible by cgroups and namespaces
Namespaces dictate what resources a process can see and have access to. Things like Cgroups, IPC, Networks, Mounts, PIDs, Users, and UTS are the namespaces you can share or make private for new processes. More information on namespaces can be found [here](http://man7.org/linux/man-pages/man7/namespaces.7.html). Namespaces make it possible for processes in a container to feel isolated from the host environment and even vs other containers.

Cgroups dictate how much of a resource that a process has access to. Some of the resources that can be limited include cpu, memory, blkio, etc. More information can be found [here](http://man7.org/linux/man-pages/man7/cgroups.7.html)


## Create your own container
The best way to understand how containers work is to make one. Below is a short guide on how to create simple containers.

### Make a directory to house our container in
`$ mkdir /tmp/container && cd $_`

### Export an existing file system to the container directory
We are gonna cheat a bit here and just export the file system of an existing docker image(alpine). We do this for convenience. It provides us with a basic file directory structure and some basic binaries for our use.
`$ docker pull alpine`
`$ docker export $(docker create alpine) | tar -xvf -`

### Beginnings of a container
Inside the `/bin` directory are some pre existing binaries. Try running the shell app.
`$ ./bin/sh`
You are now running a shell with all the capabilities that it provides. Running `ls` will show the contents of our `/tmp/container` directory, `pwd` shows the directory `/tmp/container` etc etc. Play around in this shell. The problem you will notice is that this environment is not very isolated. `exit` from this shell.

# Change the root of our container
When we run `pwd`, we are aware of where we are in the host filesystem. Lets make our container have it's own root
`$ sudo chroot /tmp/container /bin/sh`
`chroot` changes the root directory of the resulting process that is run(in this case /bin/sh). Now when running `pwd` it shows `/`. Our container now has it's own root directory. Our container is coming along!

# Mount the proc directory
Since our container just contains folders and binaries, we need to mount a special directory so that data from the kernel can be bubbled up to the OS. Run `sudo chroot /tmp/container /bin/sh` and then run `ps`. Notice process info doesn't return. This is because we don't have a special ps run `exit` to exit shell. Lets mount the special `/proc` directory that gives the system meta data held in the kernel.


`$ sudo mount -t proc proc proc/`
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





