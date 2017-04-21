# Usage

`diyc [hv][-m NUMBER] [-ip IPV4 ADDRESS] <NAME> <IMAGE> <CMD>`

    -h, --help           print the help

    -i, --ip             ip address of the container, if not set then host
                         network is used. It must be in the 172.16.0/16 network
                         as the bridge diyc0 is 172.16.0.1

    -m, --mem            maximum size of the memory in MB allowed for the container
                         by default there no explicit limit defined.

    -v, --verbose        more verbose output

    <NAME>               name of the container, needs to be unique

    <IMAGE>              image to be used for the container, must be a directory name
                         under the images directory

    <CMD>                command to be executed inside the container



## Example to get a container running

```sh

$ git clone git@github.com:w-vi/diyc.git
Cloning into 'diyc'...
remote: Counting objects: 9, done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 9 (delta 1), reused 9 (delta 1), pack-reused 0
Receiving objects: 100% (9/9), 12.95 KiB | 0 bytes/s, done.
Resolving deltas: 100% (1/1), done.

$ cd diyc

$ make setup
sudo iptables -A FORWARD -i enp5s0 -o veth -j ACCEPT || true
sudo iptables -A FORWARD -o enp5s0 -i veth -j ACCEPT || true
sudo iptables -t nat -A POSTROUTING -s 172.16.0.0/16 -j MASQUERADE || true
sudo ip link add name diyc0 type bridge || true
sudo ip addr add dev diyc0 172.16.0.1/24 || true
sudo ip link set diyc0 up || true
sudo iptables -A FORWARD -i enp5s0 -o diyc0 -j ACCEPT || true
sudo iptables -A FORWARD -o enp5s0 -i diyc0 -j ACCEPT || true
sudo iptables -A FORWARD -o diyc0 -i diyc0 -j ACCEPT || true
mkdir -p containers
mkdir -p images

$ make
gcc -std=c99 -Wall -Werror -O2 src/diyc.c -o diyc
gcc -std=c99 -Wall -Werror -O2 src/nsexec.c -o nsexec

$ docker pull debian
Using default tag: latest
latest: Pulling from library/debian
Digest: sha256:72f784399fd2719b4cb4e16ef8e369a39dc67f53d978cd3e2e7bf4e502c7b793
Status: Image is up to date for debian:latest

$ docker run -ti debian /bin/bash

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
2c924241399c        debian              "/bin/bash"         21 seconds ago      Up 20 seconds                           epic_leavitt

$ docker export 2c924241399c >! debian.tar

$ mkdir images/debian

$ tar -xf debian.tar -C images/debian/

$ sudo ./diyc my1 debian bash

> root@my1:/# exit
  exit

```