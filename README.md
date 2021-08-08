# How to use this
You can use this easily in 3 steps  
1. Clone this repo
  ```
  git clone https://github.com/nrazavi70/Ansible.git
  ```
2. Edit the host file and enter your target machine addresses. [**You have to enter the domain name for target machines that use windows**]  
  ```
  [Linux-Clients]
  192.168.20.66
  192.168.20.79
  192.168.20.76

  [Windows-Clients]
  ast-rodc.ramand.local
  chia-machine.ramand.local

  [Proxies]

```
3. Run the ansible-playbook command:
  ```
  ansible-playbook Zabbix.yml -i hosts 
  ```
