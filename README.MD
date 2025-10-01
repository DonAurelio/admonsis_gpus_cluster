# HPCSS Cluster Provision

**Requirements:** Ubuntu 22.04 LTS

## Getting started

Install Ansible on the master node (which serves as the Control Machine).

```sh
sudo apt-get update
sudo apt-add-repository ppa:ansible/ansible -y
sudo apt-get install ansible -y
sudo apt-get install python3-passlib
```

Tell Ansible how to communicate with each slave machine (which will be the Machines Under Control). The Ansible Inventory File specifies how Ansible can access remote machines. This file is typically located at `/etc/ansible/hosts`.

```sh
sudo vim /etc/ansible/hosts

[cluster]
ss-00 ansible_host=<ip> ansible_port=22 ansible_user=<username> ansible_ssh_private_key_file=/home/<username>/.ssh/id_rsa ansible_python_interpreter=/usr/bin/python3.10 ansible_ssh_common_args='-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null'
```

Test if Ansible can connect to each Machine Under Control.

```sh
ansible -m ping cluster
```

Test sending a command to all machines.

```sh
ansible -K -a "/bin/ls" cluster
```

## Cluster Provision

1. Povision de master node.
    ```sh
    ansible-playbook -K main.yml --tags "master_node"
    ```
2. Reboot the master node.
    ```sh
    sudo reboot now
    ```
3. Provision de slave nodes
    ```sh
    ansible-playbook -K main.yml --tags "slave_nodes"
    ```
