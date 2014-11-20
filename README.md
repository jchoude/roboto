roboto
======

Repository to contain the Ansible files that can be used to configure machines in the lab and in the general Neuroimaging community.


Usage
------

To configure the packages on a new machine, this system will use a tool called [Ansible](http://www.ansible.com/home).

This system allows the user to configure multiple machines at the same time, from a Control machine. Each configured machine will be called a Slave. In a case where there is only one machine to configure, the Control and Slave machines can be the same computer.

**Basic OS setup for the Slave Machine**

- Install a default Ubuntu (or Ubuntu derived) machine
- Make sure your user is admin

    > groups your_user_name

  should tell you you are in the sudo group
- Make sure the SSH deamon is running
    
    > service ssh status

  should tell you something like 

    > ssh start/running, process 3521

  If it says that the service is not recognized, install the OpenSSH server with:
    
    > sudo apt-get install openssh-server

  If it is not running, use
    > sudo service ssh start

- Add your user to the SSH config
    
    > sudo nano /etc/ssh/sshd_config

  At the end of the file, add a line like
    
    > AllowUsers your_username

  `your_username` must be the username you use to connect to your machine.

**Configuration to allow Ansible to connect to the Slave machine**

-  Configure your Control machine for key-based SSH authentication. The next steps must be done on the **Control machine**.
  - Make sure you have a public / private key pair. To do so:
      
      > ls ~/.ssh/

    If you don’t have the .ssh directory, or if you don’t have something called id_rsa.pub in that directory, you need to generate it. To do so:
      
      > cd ~

      > ssh-keygen

    then press Enter 3 times. Your key pair is now generated.
- The next steps must be done on the **Slave Machine**.
  - Add your public key on the Slave machine. To do so, you need to copy your public key (id_rsa.pub) on your Slave Machine. Supposing it was copied to your home directory, you can then  go to your home on the slave machine.
      
      > cd ~

  - If the id_rsa.pub file was copied in the home directory, you can do
      
      > cat id_rsa.pub > .ssh/authorized_keys

- **From the Control machine**
  - Make sure you can login without a password by doing:
      
      > ssh your_username@slave_machine_address

    If you get connected, everything is good.

**Ansible setup and configuration**

This is to be done on the **Control machine**

- Make sure pip is installed
    
    > pip

  If this doesn’t work, install it
    
    > sudo apt-get install python-pip python-dev libyaml-dev

- Make sure it is updated
    
    > sudo pip install pip -U

- Install the dependencies:
    
    > sudo pip install cython paramiko PyYAML jinja2 httplib2

- Go to a directory where you want to copy source code (for example ~/src)
- Make sure Git is installed
    
    > git

  If not installed, do
    
    > sudo apt-get install git

- Get Ansible’s code
    
    > git clone git://github.com/ansible/ansible.git --recursive

- When you are ready to use Ansible to configure and install tools
    
    > cd ansible

    > source ./hacking/env-setup


**Tools configuration and installation**

Once all the pre-requisites are configured, you can clone this repository. Use the clone link on the right side of the page.
- After cloning, you will need to create a "host file" that will specify which machine to configure. You can call it my_host. It must contain something similar to
    
    > [machine]

    > machine_ip_address

  where `machine_ip_address` is the IP address of the machine to configure. If the **Control** and **Slave** machines are the same, you can put 127.0.0.1 as the IP.
- To specify what you want to install, you will create a file based on `template_playbook.yml`, which is included in this repository. In this file, you must specify the hosts group to apply this playbook to (if you created the file `my_host` as suggested, it will be called `machine`) and the `roles` to apply. Each role is one `task`, or one element to install. They each correspond to a directory in this repository. You can therefore see everything that can be installed by having a look at the directories. For example, if you need to install FSL, you can add the `fsl` role to your `playbook` file.
- Always make sure the `common` and `dependencies` roles are included in the file.
- To do the real setup, you need to do the Ansible setup step (see last step of previous section), and then call
    
    > ansible-playbook -i my_host -u your_user_name -K name_of_playbook_file.yml
    