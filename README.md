# SKOPE Server Config

SKOPE uses the Docker ecosystem to minimize changes necessary on any 
host server. This document describes the change necessary to successfully
deploy SKOPE on other systems.

The basic system changes required to deploy SKOPE locally are minimal
and shown below. Installation and deployment using existing SKOPE data
comes down to the following 3 steps:

1. prep the local system
1. copy data from the existing SKOPE system
1. start the SKOPE services

### Basic System Config

1. add group skope gid=1258
1. add ubuntu user to skope group
1. add vm.max_map_count = 262144 for elasticsearch
1. install docker-ce
1. install docker-compose
1. add public keys for collaborators

```
$ sudo addgroup -gid 1258 skope
Adding group `skope' (GID 1258) ...
Done.
$ sudo usermod -a -G skope ubuntu
$ sudo sysctl -w -q vm.max_map_count=262144
$ echo 'vm.max_map_count=262144' | sudo tee /etc/sysctl.d/99-elasticsearch.conf
vm.max_map_count
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
...
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
...
$ sudo apt-get update
...
$ sudo apt-get install docker-ce
...
$ sudo curl -L https://github.com/docker/compose/releases/download/1.20.1/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
...
$ sudo chmod +x /usr/local/bin/docker-compose
```

### Reboot

Now would be a good time for `reboot` and sanity check. `groups` should show
the local user in group skope, `sysctl vm.max_map_count` should return 
262144, and `sudo docker ps` should show docker running.

### Get Existing Data

If NFS mounting data from NCSA's storage condo include the following steps:
1. apt install nfs-common
1. mount skope directory from storage condo

```
$ sudo apt-get install nfs-common
...
$ sudo mkdir /projects/skope
$ echo 'cnfs.ncsa.illinois.edu:/condo/skope /projects/skope nfs auto,noatime,nolock,bg,nfsvers=3,intr,tcp,actimeo=1800 0 0' | sudo tee --append /etc/fstab
$ sudo mount -a
```

If copying data from one of the existing servers include the following steps:
1. ensure public key is available on SKOPE server
1. create local /projects/skope directory
1. set permissions on /projects/skope
1. rsync from /projects/skope/datasets and /projects/skope/staging

```
$ sudo mkdir -p /projects/skope
$ sudo chown ubuntu:skope /projects/skope
$ rsync -avz ubuntu@staging.openskope.org:/projects/skope/datasets /projects/skope
...
$ rsync -avz ubuntu@staging.openskope.org:/projects/skope/staging /projects/skope
...
```

### Starting SKOPE

This section is from the skope-deployment repository but repeated here
for convenience. See the skope-deployment instructions for the most recent
changes.

1. clone the GitHub repository
1. change to staging branch
1. use docker-compose to run start services

```
$ git clone https://github.com/openskope/skope-deployment.git
...
$ cd skope-deployment
$ git checkout staging
...
$ sudo docker-compose up -d
...
```
