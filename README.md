# glusterfs
The current project enables provisioning of Gluster FS [https://www.gluster.org/] cluster over a host of machines.
It offers an easily configurable mechanism for initializing and setting up the cluster.
In the following subsections we present the workflow for you to set it up.

## Pre-requisites: 
- Ensure that you have installed ansible version 2.9.x on your local machine. Please see [https://www.ansible.com/] for further details on installing ansible on your local machine.
Once ensible is installed, you can verify its version using the command `ansible --version` on you bash shell. you should receive an output such as this:
```
ansible 2.9.1
config file = /Users/antorweep/Documents/dev/mysome_glusterfs/ansible.cfg
configured module search path = ['/Users/antorweep/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
ansible python module location = /usr/local/lib/python3.7/site-packages/ansible
executable location = /usr/local/bin/ansible
python version = 3.7.4 (default, Jul  9 2019, 18:13:23) [Clang 10.0.1 (clang-1001.0.46.4)]
```
- Furthermore, on your local machine you need to have some ansible plugins installed.
  - Navigate to the folder `mysome_glusterfs`
  - Execute the command `ansible-galaxy install -r requirements.yml`

- The remote machines donot need ansible installed. However, all remote hosts **must** have python version `2.7.x` or `above`

## Configuration
There are very few parameters to be configured currently. All configurations are made inside *group_vars/all.yml*. 
- If you are using the automated process for host setup (*see bellow*), it needs few steps to enable ansible to setup the remote environment
  - **API token**
    - In order to connect with Digital Ocean. 
    - Please see [https://www.digitalocean.com/docs/api/create-personal-access-token/]
    - Once you get the token, please find `do_oauth_token` in `all.yml` and set its value there
  - **SSH Keys**
    - In order for setting up the cluster, ansible needs ssh password less login to the host machines.
    - Your ssh public key from your local machines should be registered with digital ocean, see [https://www.digitalocean.com/docs/droplets/how-to/add-ssh-keys/to-account/] 
    - Once you have the ssh keys registered with Digital Ocean, you need to retrived ths ssh key ids from digital ocean. You can execute the following command on the bash on your local machine to get  this ids: `curl -X GET -silent "https://api.digitalocean.com/v2/account/keys" -H "Authorization: Bearer API token`
    - This will get a list of all ssh keys stored to your Digital Ocean account. You need to find the **numeric id** associated with your ssh key name. 
    - Once you get the ssh key id, please find `ssh_keys` in `all.yml` and set its value there within the `[]` brackets
  - **Volume Name**
    - `gluster_cluster_volume` specifies the name of the created glusterfs volume. 
    - You would need this name and ip address (from `inventory/hosts`) of any one of the machines in order to mount this filesystem into another cluster/system.

## Defining the remote host machines
In order to set up gusterfs cluster we would need a set of host machines. Ansible will comunicate with these machines and setup your cluster.

There are two mechanisms to set up the remote machines. Either manually or automated. However, the automated process currently only works on Digital Ocean. It is recommended to move with the automated process as of now

### Automated
- Please navigate to the file `inventory/hosts_template`
- It looks as follows:
```
[gfscluster]



```
- In order the specify the host machines, you need to populate this file `inventory/hosts_template` with the names of the host that you want to create. Each line/row in the file would represent a host machine. The root/first line `[gfscluster]` gives a name to the cluster for internal reference in the project and **must not be changed**. Please fill each line in the format: `hostname ansible_python_interpreter="/path/to/python"`
  - `hostname`: can be any name. Must be unique for each machine. The project will internally refer to the machines with this name
  - `ansible_python_interpreter`: In order for ansible to work, we need python 2.7.x or above available on each remote machine. Here we specify the **path of python on the remote machine** so that our local ansible project know where to find python on these machines.
- The following *example* defines 3 machines as remote hosts
```
[gfscluster]
gfs1 ansible_python_interpreter="/usr/bin/python3"
gfs2 ansible_python_interpreter="/usr/bin/python3"
gfs2 ansible_python_interpreter="/usr/bin/python3"
```

- Next we would run the following playbooks to create
    - the number of VMs / Droplets
    - a block mount on each machines
- Playbook: `000.init.yml`
    - Execute: `ansible-playbook -v 000.init.yml`
    - Create a host file  inside inventory that would be filled up with the ips of the created host machines
- Playbook: `001.spawn_droplets.yml`
  - Execute: `ansible-playbook -v 001.spawn_droplets.yml`
  - Creates the specified number of host machines on digital ocean. It also creates and mounts a block storage in digital ocean for each droplet / vm

### Manual
- Please navigate to the file `inventory/hosts_template`
- It looks as follows:
```
[gfscluster]



```
- Rename this file as `inventory/hosts`
- In order the specify the host machines, you need to populate this file `inventory/hosts` with the ip address of these machines. Each line/row in the file would represent a host machine. The root/first line `[gfscluster]` gives a name to the cluster for internal reference in the project and **must not be changed**. Please fill each line in the format: `hostname ansible_host=remote.machine1.ip.adress  ansible_python_interpreter="/path/to/python"`
  - `hostname`: can be any name. Must be unique for each machine. The project will internally refer to the machines with this name
  - `ansible_host`: the ip address of the remote host. This machine should be accessable over the network with this ip address
  - `ansible_python_interpreter`: In order for ansible to work, we need python 2.7.x or above available on each remote machine. Here we specify the **path of python on the remote machine** so that our local ansible project know where to find python on these machines.
- The following *example* defines 3 machines as remote hosts
```
[gfscluster]
gfs1 ansible_host=147.182.121.59  ansible_python_interpreter="/usr/bin/python3"
gfs2 ansible_host=117.247.73.159  ansible_python_interpreter="/usr/bin/python3"
gfs2 ansible_host=157.245.79.195  ansible_python_interpreter="/usr/bin/python3"
```
- **!!!Required: Ensure that you have password less SSH for these host for the user root**

## Setting up glusterfs
Setting up of glusterfs requires the following steps. Creating th infrastructure with all dependencies installed and starting the gluster services in all the host machines. Finally, there is also mounting the point, but it is required only at client system that uses this gluster cluster. Therefore the mounting is not conducted for this cluster and the playbook is just shown as an example on how to mount this cluster into another system.

- Playbook: `002.setup_glusterfs_infra.yml`
  - Execute: `ansible-playbook -v 002.setup_glusterfs_infra.yml -u root`
  - Sets up the remote host machines to join a glusterfs cluster
- Playbook: `003.setup_glusterfs_cluster.yml`
  - Execute: `ansible-playbook -v 003.setup_glusterfs_cluster.yml -u root`
  - Create the glusterfs cluster
- Playbook: `004.mount_glusterfs.yml`
    - Execute: `ansible-playbook -v 004.mount_glusterfs.yml -u root`
    - Mounts the glusterfs cluster. **Not to be executed for setting up this cluster**. It is rather shown as an example so that this cluster can be mounted into another system or cluster as a network file system