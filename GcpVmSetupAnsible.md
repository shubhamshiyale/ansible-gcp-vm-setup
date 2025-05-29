# ansible-gcp-vm-setup

## Create vm
gcloud compute instances create ansible-master ansible-node --zone=us-central1-a --machine-type=e2-medium --image-family=debian-11 --image-project=debian-cloud

## login to vm
gcloud compute ssh ansible-master --zone=us-central1-a
gcloud compute ssh ansible-node --zone=us-central1-a 

## install below plugins in both vm 
sudo apt update
sudo apt install ansible -y


## create a generic user 
sudo adduser ansible
sudo usermod -aG sudo ansible

## Switch to user 
sudo su - ansible

## generate an SSH key
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa -N ""

## Get the external ips
gcloud compute instances list #copy the external ip


## share the fingerprints to the node
ssh-copy-id ansible@<EXTERNAL_NODE_IP>

## if ssh-copy-id didnt worked do manually
gcloud compute ssh ansible-master --zone=us-central1-a
cat ~/.ssh/id_rsa.pub ## copy the key 
gcloud compute ssh ansible-node --zone=us-central1-a
sudo su - ansible
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys  # paste the public key content here
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys


## from master validate the connection
ssh ansible@<EXTERNAL_NODE_IP>

## Create Ansible inventory file
gcloud compute ssh ansible-master --zone=us-central1-a # login back to master
mkdir ~/ansible && cd ~/ansible
------
vi hosts

[nodes]
ansible-node ansible_host=<EXTERNAL_NODE_IP> ansible_user=ansible

## Ping Test
ansible -i hosts nodes -m ping


## validate the connection 
ansible-node | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

## Test the Installation from Master

### Case 1: If `ansible` user has passwordless sudo enabled

1. Enable passwordless sudo for the `ansible` user on **both** master and node:

```bash
sudo visudo

ansible ALL=(ALL) NOPASSWD:ALL

## From the master VM, run the following command to install nginx on the node:
ansible -i hosts nodes -m apt -a "name=nginx state=present update_cache=yes" --become

## If ansible user requires a sudo password
ansible -i hosts nodes -m apt -a "name=nginx state=present update_cache=yes" --become --ask-become-pass

## Expected Output

ansible-node | SUCCESS => {
    "changed": true,
    "stdout": "...",
}

## SSH into the node and check if nginx is running:
systemctl status nginx





