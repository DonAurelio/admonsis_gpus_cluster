# JupyterHub + SLURM

**Requirements:** Ubuntu 22.04 LTS

## Getting Started

1. Log in to the **head node** and create an SSH key.

   ```sh
   ssh-keygen -t rsa
   ```

2. Copy the SSH key to all **compute nodes**.

   ```sh
   ssh-copy-id <username>@<ip>
   ```

3. Install Ansible on the head node (which serves as the Control Machine).

   ```sh
   sudo apt-get update
   sudo apt-add-repository ppa:ansible/ansible -y
   sudo apt-get install ansible -y
   sudo apt-get install python3-passlib -y
   ```

4. Configure Ansible to communicate with each compute node (the Machines Under Control).
   The Ansible Inventory File specifies how Ansible accesses remote machines.
   This file is typically located at `/etc/ansible/hosts`.

   ```sh
   sudo vim /etc/ansible/hosts

   [cluster]
   ss-00 ansible_host=<ip> ansible_port=22 ansible_user=<username> ansible_ssh_private_key_file=/home/<username>/.ssh/id_rsa ansible_python_interpreter=/usr/bin/python3.10 ansible_ssh_common_args='-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null'
   ```

5. Test if Ansible can connect to each Machine Under Control.

   ```sh
   ansible -m ping cluster
   ```

6. Test sending a command to all machines.

   ```sh
   ansible -K -a "/bin/ls" cluster
   ```

## Cluster Provision

1. Provision the head node.

   ```sh
   ansible-playbook -K main.yml --tags "head_node"
   ```

2. Reboot the head node.

   ```sh
   sudo reboot now
   ```

3. Provision the compute nodes.

   ```sh
   ansible-playbook -K main.yml --tags "compute_nodes"
   ```
