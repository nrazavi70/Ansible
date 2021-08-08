- 1 Ansible
   - 1.1 Ansible Ad-hoc Commands
   - 1.2 Ansible Common Options
   - 1.3 Ansible Configuration File
   - 1.4 Ansible Connection to Windows Host
   - 1.5 Ansible Handlers
   - 1.6 Ansible Installation
   - 1.7 Ansible Inventory File
   - 1.8 Ansible Modules
   - 1.9 Ansible Playbooks
   - 1.10 Ansible Playbook Structure
   - 1.11 Ansible Roles
   - 1.12 Ansible Source Host Preparation
   - 1.13 Ansible Target Machine Preparation
   - 1.14 Ansible Templates
   - 1.15 Ansible Variables
   - 1.16 Ansible with Structure Examples
   - 1.17 YAML Structure


# Ansible Ad-hoc Commands

To run one time commands with Ansible you gotta follow the pattern below:
```
ansible [machine group/single machine] -m [module] -a [args] [Optional Common Options]
```
The first argument is the address to the target machine. This can be either IP address or the domain name of the target machine.

The second argument is the module which contain the action you want to perform on the target machine. For instance the ping action can be done via the p
ing module or app installation in Linux can be done via package module. To read more on this issue you can click here.

The third argument you have to provide Ansible with the required arguments. Some arguments are needed for the modules. These arguments are provide
within double quotations and without them the module wont be working.

Lastly you can enter some common options that are sometimes necessary and sometimes are just optional. You can learn more about these here.

An example of this script would be as follows:

```
ansible 192.168.1.100 -m package -a "name=nginx state=installed" -i hosts -u root
```

# Ansible Common Options

When writing Ansible scripts sometimes you ought to provide some arguments based on your situation. Here is a list of arguments you can provide in your
scripts.

Here are some examples:

|Argument|What the argument does|
|:-:|:-:|
|-k|Prompts for passwords whenever needed|
|-u|Runs the script on the target machine as the user stated|
|-i|Specify the inventory file|
|-a|When you want to run a raw command|

To find a complete list of all the options in Ansible you can visit <https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html>

# Ansible Configuration File

The Ansible configuration file contains many settings which can be helpful specially when you are working around with playbooks. We have multiple
Configuration files which have precedence over each other. The hierarchy of them are as follows:

```
1. ANSIBLE_CONFIG (Environment variable if set)
2. ansible.cfg (in the current directory)
3. ~/.asnible.cfg (in the home directory)
4. /etc/ansible/ansible.cfg
```

# Ansible Connection to Windows Host

First make sure you have installed Ansible correctly. Some issues may occur if you are using python2.7 as your interpreter. You can check this with this
command:
```
ansible --version | grep "python version"
```
After you made sure you are using python3 you can proceed to the next level which is writing you playbook. Now you want to mention a couple of things in
your playbook such as the credentials you want to use for connection, the type of connection and the port for it. To specify these you have to enter these
variables in your playbook like so:

```
-  name: Linux Agent Setup
   hosts: Linux-Clients
   remote_user: root
   roles:
      - Client-Linux
-  name: Windows Agent Setup
   hosts: Windows-Clients
   gather_facts: false
   vars:
      ansible_user: "administrator@RAMAND.LOCAL"
      ansible_password: "*******"
      ansible_connection: winrm
      ansible_winrm_transport: kerberos
      ansible_port: 5985
   roles:
      - Client-Windows
```
You can enter these vars in your inventory file as well like this:
```
[Linux-Clients]
192.168.20.2
192.168.20.3
192.168.20.4

[Windows-Clients]
RODC.ramand.local

[Windows-Clients:vars]
ansible_user="administrator@RAMAND.LOCAL"
ansible_password="*******"
ansible_connection=winrm
ansible_winrm_transport=kerberos
ansible_port=

[Proxies]

```
You can test if you are connecting to windows servers successfully with the following command:
```
kinit USERNAME.DOMAIN_NAME
```

If you are using HTTP you have to config winrm to allow unencrypted logins as well with the following command:
```
Set-Item -Path WSMan:\localhost\Service\AllowUnencrypted -Value true
```
https://argonsys.com/microsoft-cloud/articles/configuring-ansible-manage-windows-servers-step-step/

https://www.ansible.com/blog/connecting-to-a-windows-host

https://github.com/ansible/ansible/issues/


# Ansible Handlers

Ansible handlers are tasks that you may want to run in different parts of an Ansible playbook. You define them by a name at the end of each playbook and
you mention what their action has to be and then throughout the playbook you can call them under "notify" key in each task. Note that if a handler task is
already queued in one task(this means that the task was ran and not skipped), and the next task has the same handler, it won't perform the handler action
twice. An example would be as follows:

```
-  hosts: all
   vars:
      # used for filepaths
      site_name: "tutorialinux"
      # used in the website's markup
      site_title: "Hope."
      # used in the web server configuration file
      site_url: "www.tutorialinux.com"
   
   tasks:
      - name: Install nginx.
        package: name=nginx state=latest
      
      - name: Create website directory
        file: path="/var/www/{{ site_name }}" state=directory mode=
        
      - name: Create main nginx config file
        template:
          src: "templates/nginx.conf"
          dest: "/etc/nginx/nginx.conf"
        notify:
        - restart nginx
      - name: Create nginx vhost config file
        template:
          src: "templates/website.conf"
          dest: "/etc/nginx/conf.d/{{ site_name }}.conf"
        notify:
        - restart nginx
      - name: Create website
        template:
          src: "templates/index.html"
          dest: "/var/www/{{ site_name }}/index.html"
      - name: Remove default nginx vhost configuration
        file: path=/etc/nginx/sites-enabled/default state=absent
        notify:
        - restart nginx

    handlers:
      - name: restart nginx
        service:
          name: nginx
          state: restarted
```
Notice that the handler "restart nginx" is called in many tasks. Some important notes about the behavior of this example:

For example if the tasks "Create main nginx config file" and "Create nginx vhost config file" both are running, while the handler "restart nginx" is
called in them both, the playbook is gonna restart nginx only ONCE.
If a task is skipped (since it was already done which happens when you run the playbook twice or one of the servers already had the same nginx
config file) the handler at that task won't be queued at all and would be skipped as well.
Instead of running the restart task many times, we saved time and space by using handlers.
# Ansible Installation

To install Ansible you have to make sure that its interpreter is python3 (This was not necessary before.). Enter the following command to download get pip file. We need this to install pip which is the python package manager. To do this you have to do the following:
```
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
```
Use python3 to run the file you just downloaded:
```
python3 get-pip.py --user
```
If you want to use Ansible to manage windows servers you also have to install pywinrm. To have to do the following:
First we install pywinrm since you need WinRM to work with windows:
```
pip install pywinrm
```
The you have to install kerberos module with pip which can be problematic sometimes so first run this command:
```
pip install kerberos
```
But if you were experiencing some issues with kerberos module installation you can fix it by installing couple of applications via apt package
manager:
```
sudo apt-get install python3-dev
sudo apt-get install libkrb5-dev
```
and then install GSSAPI and social-auth-kerberos to fix the issue with kerberos module installation:
```
pip install gssapi
pip install social-auth-kerberos
```
Now you can install kerberos module and you would have no trouble connecting to windows machines via WinRM. Do not forget to set the
required data for WinRM connection in case you want to connect to windows. You can learn about these information here.

# Ansible Inventory File

To state the target machine you can either enter its address directly in the Ansible script or you can enter the category name of the target machines. The
name of these categories and also host addresses of target machines are defined in your inventory in a file called Hosts. We have multiple Hosts files
which have precedence over each other. The hierarchy of them are as follows:

```
1.Specified in the script
2.Specified in the playbook
3.Hosts file in the environment
4.Hosts file at /etc/Ansible/Hosts
```

# Ansible Modules

Modules in Ansible are used to perform predefined actions that were their intention in the first place. For instance to install a package, copy/edit a file or to
check if something is working correctly.
To figure out what a module does you can visit here. Notice that if you are visiting the link from Iran you will be needing a VPN or Shecan because of the
sanctions imposed.


# Ansible Playbooks

The subject of Ansible playbook is hard to grasp at the beginning but easy to work with once you get used to it. Basically you enter all your configurations
and host groups that you want those configurations to be applied to in a YAML file and run it with a simple command.
The structure of an Ansible playbook command looks like this:

```
ansible-playbook [PLAYBOOK_FILE] -i [INVENTORY_FILE] -a [args]
```
Below you can see an example of a playbook with its tasks written inside:

```
# Run with ansible-playbook <filename> -k
# (make sure to add the IPs of machines you want to manage to /etc/ansible/hosts first)

- hosts: all
  gather_facts: False
  remote_user: ubuntu
  become: yes
  become_user: root
  become_method: sudo

  tasks:
    - name: Update Packages
      raw: (apt-get update && apt-get -y upgrade)
      
    - name: Install Python 2
      raw: test -e /usr/bin/python || (apt-get update && apt-get install -y python)
      
    - name: Fancy way of doing authorized_keys
      authorized_key: user=root
                    exclusive=no
                    key="{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
                    
    - name: COMMON | Set environment
      blockinfile:
        dest: /etc/environment
        block: |
          LC_ALL=en_US.UTF-8
          LANG=en_US.UTF-8
      register: newenv
      
    - block:
      - name: COMMON | Generate locales
        raw: locale-gen en_US.UTF-8
        
      - name: COMMON | Reconfigure locales
        raw: dpkg-reconfigure locales
      # only run this task block when we've just changed /etc/environment
      when: newenv.changed


    #- name: Create /root/.ssh
    # file: path=/root/.ssh state=directory mode=
    #- name: Create /root/.ssh/authorized_keys from our local ssh pubkey
    # lineinfile: dest=/root/.ssh/authorized_keys line="{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
```
In more complex playbooks, you can use structures to make things clearer, more readable and scalable.


# Ansible Playbook Structure

For readability, scalability and maintenance a structure is recommended for Ansible complex playbooks. Instead of learning this by heart you can simply
use the python code below to create the structure automatically. Just run the python code and feed it with playbook name and roles you want and it
creates the structure required.
```
python ./create_playbook.py [PLAYBOOK_NAME] [ROLE1] [ROLE2] [ROLE3] ...
```
Here is an example of what it would look like:
```
python ./create_playbook.py Zabbix Agent Proxy Web Database
```
# Ansible Roles

Another one of Ansible features are Ansible roles. We can make use of Ansible roles when we are encountering reusable tasks, templates and handlers.
Basically we tag a number of actions with a role name and then call that role name in the playbook.

```
- name: Webserver Setup
  hosts: webservers
  remote_user: root
  roles:
    - common
    - web
```
What this example does is that it has a task within itself to apply the actions within "common" and "web" role to hosts within "webservers" category in the An
sible Inventory File with the remote user "root".

An example of writing a role file would be like below:
```
- name: COMMON | Install basic packages
  package: name={{ item }} state=present
  with_items:
      - vim
      - nano
      - curl
```
This piece of YAML names the role "COMMON" and installs all the packages stated at the end. You can apply this role to any server you want and it would
install "Vim" , "Nano" and "Curl" on those servers.

Note that role has come instead of tasks.


# Ansible Source Host Preparation

Just to avoid adding every single machine key to your "Known-Hosts" file by hand you can enter the Ansible configuration file and disable
host_key_checking option.


# Ansible Target Machine Preparation

To prepare a host for Ansible scripts you have to perform a certain configuration on it. Instead you can run this playbook against it and it would be
prepared.

You can enter the script below to run the playbook but keep in mind that the ssh Username of a sudoer user must be entered in the script in order for it to
work.

```
ansible-playbook prepare_ansible_target.yml -i hosts -u [SUDOER_USER] -k --ask-sudo-pass
```

# Ansible Templates

This is one the best things about Ansible. Basically with templates and Ansible Variables in Ansible you can run one playbook and it would result in each
server having its own configuration.

For instance you can define a variable "server_name". Then you can make a templates folder and put all template files in it. A template file is a file that
has the format of any standard configuration file (it can be a Nginx config, Ansible config, etc host file or any config file), then within that configuration file
you can put the name of those variables by calling them with curly brackets. This way each server would have its own name set in the configuration files
that you intend to edit.


# Ansible Variables

You can user variables in Ansible. This comes in handy specially in playbooks. All you have to do is to define the variables as "vars" at the same level as
"hosts" and when you want to call them you have to use double curly brackets "{{var1}}".

# Ansible with Structure Examples

Located at root directory :


Main Ansible playbook file:
```
---
- name: Database Setup
  hosts: dbservers
  remote_user: root
  roles:
    - common
    - database
    
- name: Webserver Setup
  hosts: webservers
  remote_user: root
  roles:
    - common
    - web
```
Inventory file:
```
[Category1]
Host
Host
```
```
[Category2]
Host
Host
```
```
[Category3]
Host
Host
```
```
[Category1:vars]
ansible_ssh_user = 'Username@MYDOMAIN.COM'
ansible_password = 'password'
ansible_connection = winrm
ansible_ssh_port = 5986
```
Located at Roles are the directories with role name and in each role there are "files" , "handlers" , "meta" , "tasks" , "templates" and "vars" directories. In
"handlers" and "tasks" there is a Main.yml file that contains handlers and tasks of that role:


a.Tasks main file:
```
---
- name: COMMON | Install basic packages
  package: name={{ item }} state=present
  with_items:
    - wget
    - vim
    - nano
    - curl
```
b.Handlers main file:
```
---
- name: restart postgres
  service: name=postgresql state=restarted
```

# YAML Structure

If you want to be able to work with Ansible in a more complex manner, you have to learn how a YAML files is structured. This can be helpful in more than
one way since this structure is used in many services. One of the best ways to learn about the structure is to follow the instructions at here.

## Summary

It is consisted of lists and dictionaries and lists of dictionaries. Each item in the list is started with a "-" and all items have the same indentation. Also keys
and values are separated with ":".

An example of a list in YAML is:

```
- item1
- item2
- item3
- item4
```
Keys and values are like so:

```
key: value
```
Values can be lists as well. You can see examples of this below:
```
key:
- value1
- value2
- value3
- value4
```
Dictionaries can be values as well. An example of it would be like this:
```
Main_key:
key1: value1
key2: value2
key3: value3
key4: value4
```
You can also represent it like this:
```
Main_key: {key1: value1, key2: value2, key3: value3, key4: value4}
```
A dictionary can have other dictionaries as values and within that dictionary you can have a key that has a list as its value. You can see an example of this
below:
```
- Martin:
        name: Martin D'vloper
        job: Developer
        skill:
            - lisp
            - fortran
            - erlang
```

