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

## Configure SLURM

1. Connect via SSH to the compute node and check its specifications:

   ```sh
   slurmd -C

   NodeName=ss-01 CPUs=4 Boards=1 SocketsPerBoard=2 CoresPerSocket=2 ThreadsPerCore=1 RealMemory=7949
   ```

2. Return to the head node and update the `slurm.conf` file. Add the compute node specifications, making sure to append `State=UNKNOWN` at the end. Each compute node requires its own line:

   ```sh
   sudo vim /etc/slurm/slurm.conf

   # COMPUTE NODES
   NodeName=ss-01 CPUs=4 Boards=1 SocketsPerBoard=2 CoresPerSocket=2 ThreadsPerCore=1 RealMemory=7949 State=UNKNOWN
   ```

3. Restart the SLURM controller (`slurmctld`) and `munge` services on the head node:

   ```sh
   sudo systemctl restart slurmctld munge
   ```

4. Restart the `slurmd` service on the compute nodes using Ansible:

   ```sh
   ansible -K -b -a "sudo systemctl restart slurmd"
   ```
