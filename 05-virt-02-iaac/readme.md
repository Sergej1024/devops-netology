## Домашнее задание к занятию "5.2. Применение принципов IaaC в работе с виртуальными машинами"

### Задача 1

1. Опишите своими словами основные преимущества применения на практике IaaC паттернов.

    Унификация настройки серверной части, стабильность среды разработки, позволяет ускорить процесс производства и вывода продукта.

2. Какой из принципов IaaC является основополагающим?
    
   Индемпотентность - принцип при котором неважно количество повторов операции, результат всегда будет один.

### Задача 2

- Чем Ansible выгодно отличается от других систем управление конфигурациями?

   не требует установки клиентской части, работает на штатном SSH 
- Какой, на ваш взгляд, метод работы систем конфигурации более надёжный push или pull?

   По моему push надежнее, по причине ты сам запускаешь процесс когда, считаешь это нужным.

### Задача 3

Установить на личный компьютер:

- VirtualBox
- Vagrant
- Ansible

Приложить вывод команд установленных версий каждой из программ, оформленный в markdown.

```shell
[sergej@surg-adm ~]$ vagrant version
Installed Version: 2.2.16

Vagrant was unable to check for the latest version of Vagrant.
Please check manually at https://www.vagrantup.com
[sergej@surg-adm ~]$ vboxmanage --version
6.1.32r149290
[sergej@surg-adm ~]$ virtualbox --help | head -n 1 | awk '{print $NF}'
v6.1.32
[sergej@surg-adm ~]$ ansible --version
ansible 2.9.27
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/sergej/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.10/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.10.4 (main, Mar 25 2022, 00:00:00) [GCC 11.2.1 20220127 (Red Hat 11.2.1-9)]
[sergej@surg-adm ~]$ 
```

###Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

- Создать виртуальную машину.
- Зайти внутрь ВМ, убедиться, что Docker установлен с помощью команды

```shell
docker ps
```

```shell
[sergej@surg-adm vagrant]$ vagrant up
Bringing machine 'server1.netology' up with 'virtualbox' provider...
==> server1.netology: Importing base box 'bento/ubuntu-20.04'...
==> server1.netology: Matching MAC address for NAT networking...
==> server1.netology: Setting the name of the VM: server1.netology
==> server1.netology: Clearing any previously set network interfaces...
==> server1.netology: Preparing network interfaces based on configuration...
    server1.netology: Adapter 1: nat
    server1.netology: Adapter 2: hostonly
==> server1.netology: Forwarding ports...
    server1.netology: 22 (guest) => 20011 (host) (adapter 1)
    server1.netology: 22 (guest) => 2222 (host) (adapter 1)
==> server1.netology: Running 'pre-boot' VM customizations...
==> server1.netology: Booting VM...
==> server1.netology: Waiting for machine to boot. This may take a few minutes...
    server1.netology: SSH address: 127.0.0.1:2222
    server1.netology: SSH username: vagrant
    server1.netology: SSH auth method: private key
    server1.netology: 
    server1.netology: Vagrant insecure key detected. Vagrant will automatically replace
    server1.netology: this with a newly generated keypair for better security.
    server1.netology: 
    server1.netology: Inserting generated public key within guest...
    server1.netology: Removing insecure key from the guest if it's present...
    server1.netology: Key inserted! Disconnecting and reconnecting using new SSH key...
==> server1.netology: Machine booted and ready!
==> server1.netology: Checking for guest additions in VM...
==> server1.netology: Setting hostname...
==> server1.netology: Configuring and enabling network interfaces...
==> server1.netology: Mounting shared folders...
    server1.netology: /vagrant => /home/sergej/devops-netology/05-virt-02-iaac/src/vagrant
==> server1.netology: Running provisioner: ansible...
    server1.netology: Running ansible-playbook...

PLAY [nodes] *******************************************************************

TASK [Gathering Facts] *********************************************************
ok: [server1.netology]

TASK [Create directory for ssh-keys] *******************************************
ok: [server1.netology]

TASK [Adding rsa-key in /root/.ssh/authorized_keys] ****************************
changed: [server1.netology]

TASK [Checking DNS] ************************************************************
changed: [server1.netology]

TASK [Installing tools] ********************************************************
ok: [server1.netology] => (item=['git', 'curl'])

TASK [Installing docker] *******************************************************
changed: [server1.netology]

TASK [Add the current user to docker group] ************************************
changed: [server1.netology]

PLAY RECAP *********************************************************************
server1.netology           : ok=7    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[sergej@surg-adm vagrant]$ vagrant ssh
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 System information disabled due to load higher than 1.0


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento
Last login: Tue Apr 12 07:55:28 2022 from 10.0.2.2
vagrant@server1:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
vagrant@server1:~$  exit
logout
Connection to 127.0.0.1 closed.
[sergej@surg-adm vagrant]$ vagrant halt
==> server1.netology: Attempting graceful shutdown of VM...
[sergej@surg-adm vagrant]$ 
```
