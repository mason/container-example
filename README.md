



PRESENTATION
- docker made possible by cgroups and namespaces
- what are they
- demo


mkdir /tmp/container
docker export $(docker create alpine) | tar -xvf -

cd /bin show that there is a /bin/sh
explain that file system is just a bunch of files and binaries
run ./bin/sh
show that this is the beginning of our container


pwd - we are aware of where we are in the filesystem
sudo chroot /tmp/container /bin/sh
pwd
docker makes volumes that store the filesystem

not a complete OS yet. no proc directory. can't do things like ps
need a proc directory. proc is a special directory that stores a filesystem of type proc which linux uses to show info on the system
- sudo mount -t proc proc proc/
- go into container and run ps ... not our ps ids!


namespaces to the rescue
- sudo unshare -p -f --mount-proc=/tmp/container/proc chroot /tmp/container /bin/sh
- ls proc
- ps
- show linux man pages

cgroups
sudo mkdir /sys/fs/cgroup/cpu/container
echo "2000000" > /sys/fs/cgroup/memory/container/memory.limit_in_bytes
echo 15215 > /sys/fs/cgroup/memory/tasks



sudo unshare -p -f --mount-proc=/tmp/container/proc cgexec -g memory:container chroot /tmp/container /bin/sh
mount -nvt tmpfs none /dev
mknod -m 444 /dev/urandom c 1 9
script =======
#!/bin/sh
random="$(dd if=/dev/urandom ibs=20000)"





sudo unshare -p --map-root-user --user -f --mount-proc=/tmp/container/proc chroot /tmp/container /bin/sh





mkdir /tmp/container
docker export $(docker create alpine) | tar -xvf -
mkfifo /tmp/container/tmp/testpipe

sudo vi /etc/systemd/system/container.service
[Unit]
Description=Container example

[Service]
RootDirectory=/tmp/container
ExecStart=/cmd.sh


systemd-run --slice=user-1000.slice --unit=container2 ~/go/src/github.com/hello/main

systemctl set-property --runtime container2.service MemoryLimit=30M
vi /sys/fs/cgroup/memory/user.slice/user-1000.slice/container2.service/memory.limit_in_bytes

systemctl start container2 
systemctl stop container2 

systemd-cgls
