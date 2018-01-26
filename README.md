# server-config
Configures servers for SKOPE services using Ansible.

* add group skope gid=1258
* add ubuntu user to skope group 
* apt install nfs-common
* mount skope directory from storage condo
* symlink from /projects/skope to mount skope mount point
* add vm.max_map_count = 262144 for elasticsearch
* install docker-ce 
* set max TCP packet size in docker config (still necessary?)
