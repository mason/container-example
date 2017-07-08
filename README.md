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
Since our container just contains folders and binaries, we need to mount a special directory so that data from the kernel can be bubbled up to the OS. Run `sudo chroot /tmp/container /bin/sh` and then run `ps`. Notice process info doesn't return. This is because we don't have a special directory called proc which is a filesystem of type proc. Run `exit` to exit shell and return to host environment. Lets mount the special `/proc` directory that gives the system the meta data held in the kernel.

`$ sudo mount -t proc proc proc/`
Jump into the container `$ sudo chroot /tmp/container /bin/sh` and run `ps`. Notice there is now process info but it has process info of the host system. Lets fix that so that we only see processes of the container system. Run `exit` to return to host system.


# Namespaces to the rescue
We can run a process under a new namespace from the unshare command or the clone command. We will use the unshare command.
`sudo unshare -p -f --mount-proc=/tmp/container/proc chroot /tmp/container /bin/sh`
This is saying to run the `chroot` command in a new pid namespace(denoted by -p). We also pass in the location of the proc directory so that the new process will know to store the new proc info in that location. Run `ps` and notice that it now only has 2 process ids and they are all small process id numbers. Run `exit` to return to host system.

# Lets limit how much memory our container can use using cgroups
To create a new cgroup all we have to do is create a new folder in the subsystem of the resource we want to limit. In this case we create a dirctory in the memory subsystem.
`$ sudo mkdir /sys/fs/cgroup/memory/container`

Give this container only 2mb memory limit. We can do this by echoing the size in bytes we want to limit in the file `memory.limit_in_bytes`
`$ sudo su`
`$ echo "2000000" > /sys/fs/cgroup/memory/container/memory.limit_in_bytes`
`$ exit`

Create the shell script `mem.sh` that takes up memory
```
#!/bin/sh
random="$(dd if=/dev/urandom ibs=20000)"
```

Make script executable `$ chmod +x mem.sh`
Create special linux device that produces noise for our script to get random data from. `sudo mknod -m 444 dev/urandom c 1 9`

Execute the `/bin/sh` script with it's own PID namespace and it's own memory cgroup.
`sudo unshare -p -f --mount-proc=/tmp/container/proc cgexec -g memory:container chroot /tmp/container /bin/sh`

Run the `./mem.sh` shell script and wait a bit. It will run out of memory.

### Congratulations, you just created your own container!







