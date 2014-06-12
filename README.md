Devstack-Environment
====================

A single- or multi-node DevStack deployment driven by Vagrant and Ansible.

This allows you to bring up a complete single- or multi-node DevStack
deployment with just a single command.

You can deploy either a virtual environment via Vagrant, or use the
Ansible playbooks on their own to deploy on real hardware.


Prerequisites (for Vagrant deployments)
=======================================
You need to have installed:

    - Virtualbox
    - Vagrant
    - Ansible

On Ubuntu based systems, you should be able to do this:

    $ apt-get install virtualbox vagrant ansible

If you want to make sure you have the latest version of Ansible (highly
recommended), and you are still running an older version of Ubuntu (such
as 12.04, for example), you could instead do:

    $ apt-get install virtualbox vagrant python-pip
    $ pip install ansible


Prerequisites (for hardware deployments)
========================================
You need to have installed:

    - Ansible

On Ubuntu based systems, you should be able to do this:

    $ apt-get ansible

If you want to make sure you have the latest version of Ansible (highly
recommended), and you are still running an older version of Ubuntu (such
as 12.04, for example), you could instead do:

    $ apt-get install python-pip
    $ pip install ansible


Instructions (for Vagrant deployments)
======================================

    1. Clone this repository:

        $ git clone git@github.com:CiscoSystems/devstack-environment.git

    2. Change into the directory:

        $ cd devstack-environment

    3. Start the virtual machines:

        $ vagrant up

       Depending on your network connection, this step can take more than
       an hour, so please be patient.

    4. If everything has completed without problem, you can log into the
       controller and compute host, like so:

        $ vargant ssh controller
        $ vagrant ssh compute-1

You will then be logged in as user 'vagrant'. In your home directory you
have a 'workspace' directory, inside of which you will find 'log' and
'devstack'. On the controller, after changeing into the devstack directory,
you can run 'source openrc admin demo', after which you can issue various
OpenStack commands.


Instructions (for hardware deployments)
=======================================

    0. Have some hardware machines available with fresh installs of
       Ubuntu 14.04.

    1. Clone this repository:

        $ git clone git@github.com:CiscoSystems/devstack-environment.git

    2. Change into the directory:

        $ cd devstack-environment

    3. Copy the file 'vagrant_hosts_multi' or rename it to 'hosts'.

    4. Edit the 'hosts' file to change the IP addresses of your actual
       machines. You don't have to use a cache machine, so the apt-pip-cache
       machine may be removed.

    5. Open the 'vars/extra_vars.yml' file. Edit the 'pub_eth_interface'
       value as well as the 'CACHE.pkg_cache' and 'CACHE.pkg_cache_existing_ip_addr'
       settings as needed.

    6. Start the Ansible playbooks like so:

        $ ansible-playbook -i hosts site.yml


What does it do?
================
If you use Vagrant:

Have a look at the file 'Vagrantfile'. It describes to Vagrant which virtual
machines it should create. You can see three machines: One just serves as
an apt and pip cache (very useful), the other two are a controller and a
compute host (please note that the controller also acts as a compute host,
so OpenStack guests can also run on the controller machine).

After the VMs are created, they are going to be provisioned via Ansible
playbooks.

If you use hardware:

The Ansible playbooks will ensure that a suitable user is created and
available and will download, configure and install DevStack as specified.


FAQ
===
What if I want a fresh DevStack install?
----------------------------------------
There are two ways to go about it:

You can re-run the Ansible provisioning step with:

    $ vagrant provision                   <- if you use Vagrant

    or

    $ ansible-playbook -i hosts site.yml  <- if you use hardware

This will visit the the existing compute and controller hosts, run
"unstack.sh" and "clean.sh" before installing DevStack from scratch.

If you use Vagrant and you wish to destroy the compute and controller
host to make sure you have a fresh start, you can do these steps:

    $ vagrant destroy controller compute-1
    $ vagrant up
    $ vagrant provision

Note that we are avoiding destruction of the cache host. No need to do so.


What if I want different IP addresses for my hosts with Vagrant?
----------------------------------------------------------------
In the Vargantfile you find this section:

    servers = {
                :"controller"      => '192.168.99.11',
                :"compute-1"       => '192.168.99.22',
                :"apt-pip-cache"   => '192.168.99.99',
              }
    last_host_name = "apt-pip-cache"
    last_host_ip   = "192.168.99.99"

This is where the IP addresses of the machines are defined. You can change
those to some other private addresses that you may prefer.

Please note that you will also have to edit the "vagrant_hosts_multi" file,
specfically this section:

    [all-hosts]
    controller        ansible_ssh_host=192.168.99.11
    compute-1         ansible_ssh_host=192.168.99.22
    apt-pip-cache     ansible_ssh_host=192.168.99.99

Adjust the IP addresses to whatever values you have chosen.


What if I just want a single-node DevStack? Or a 5-node DevStack?
-----------------------------------------------------------------
In that case, look at that same section that we have described above in the
Vagrantfile. Comment our (or remove) the 'compute-1' host. Likewise, do the
same in the "vagrant_hosts_multi" file. In that case, find the start of the
'[compute-hosts'] section in that file and remove the compute-1 entry.

If you want more compte hosts, add them in the Vargantfile as well as the
"vagrant_hosts_multi" file (or your 'hosts' file if you use hardware).


Where can I specify what version of DevStack I'm deploying?
-----------------------------------------------------------
Find the "vars/extra_vars.yml" file and look for the DEVSTACK section.


What if I want to use different modules/tunnels/settings for DevStack?
----------------------------------------------------------------------
The DevStack settings are controller by their 'localrc' files. Please have a
look at the 'playbooks/templates' directory. In that directory, you can see
sub-directories, such as "gre". Now look inside there and you can find template
files, one for the controller and one for the compute hosts. You can adjust the
settings there as you wish.

Now look back in the "vars/extra_vars.yml" file and find the
DEVSTACK.tunnel_type setting. That setting may be "gre", for example. The
value of this setting determines the sub-directory out of which the localrc
files are taken. So, if you want to create your own configs, create a new
sub-directory in the templates directory, put your localrc templates for
compute and controller hosts in there and adjust. Then change the 'tunnel_type'
setting to the name of your new sub-directory.


How do I access the Horizon web UI for the DevStack deployment?
---------------------------------------------------------------
Point your browser to the IP address of the controller host. If you use
Vagrant, by default that address will be 192.168.99.11, so:

    http://192.168.99.11

If you changed the addresses (edited the Vagrantfile and host file), or use
haardware servers, change the URL accordingly.

