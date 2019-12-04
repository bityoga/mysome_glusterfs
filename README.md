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
- The remote machines donot need ansible installed. However, all remote hosts **must** have python version `2.7.x` or `above`

## Defining the remote host machines
In order to set up gusterfs cluster we would need a set of host machines. Ansible will comunicate with these machines and setup your cluster.

- Please navigate to the file `inventory/hosts`
- It looks as follows:
```
[gfscluster]



```
- In order the specify the host machines, you need to populate this file with the ip address of these machines. Each line/row in the file would represent a host machine. The root/first line `[gfscluster]` gives a name to the cluster for internal reference in the project and **must not be changed**. Please fill each line in the format: `hostname ansible_host=remote.machine1.ip.adress  ansible_python_interpreter="/path/to/python"`
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

## Configuration

## Setting up glusterfs
### Setting up the infrastructure 
### Setting up the cluster
### Mounting glusterfs 