# vagrant_elk_one_master_two_clients
ELK v5.4.0 stack. Single master (CentOS7) and two clients (CentOS7 and Ubuntu 16.04).

## Credit to HowToForge
   Follows the recipe here: https://www.howtoforge.com/tutorial/how-to-install-elastic-stack-on-centos-7/  
   But uses ELK v5.4.0 instead of v5.1.1

## Setup instructions:
To use this project:
1. Install VirtualBox (tested with v5.1.36)
1. Install Vagrant
1. Install Vagrant vagrant-sshfs plugin (used to share cert from master to clients)

   sudo vagrant plugin install vagrant-sshfs  

   You will need an SFTP server to use the vagrant-sshfs plugin
   * ubuntu: sudo apt-get install openssh-sftp-server
   * rhel:   sudo yum install openssh-sftp-server

1. Install vagrant boxes

   vagrant box add centos/7  
   vagrant box add ubuntu/xenial64

1. Launch VMs  

   vagrant up
