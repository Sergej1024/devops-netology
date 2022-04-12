## Домашнее задание к занятию "5.2. Применение принципов IaaC в работе с виртуальными машинами"

### Задача 1

- Опишите своими словами основные преимущества применения на практике IaaC паттернов.
- Какой из принципов IaaC является основополагающим?

### Задача 2

- Чем Ansible выгодно отличается от других систем управление конфигурациями?
- Какой, на ваш взгляд, метод работы систем конфигурации более надёжный push или pull?

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

