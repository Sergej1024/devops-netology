# devops-netology
Stroka
Будут проигнорированы все файлы в каталоге terraform
файлы с разрешением tfstate
файл crash.log
файлы с расширением .tfvars
файлы переопределения
конфигурационные файлы  .terrformrc terraform.rc

#Домашнее задание к занятию «2.4. Инструменты Git»
#Вопрос1
$ git show aefea
commit aefead2207ef7e2aa5dc81a34aedf0cad4c32545
Author: Alisdair McDiarmid <alisdair@users.noreply.github.com>
Date:   Thu Jun 18 10:29:58 2020 -0400

    Update CHANGELOG.md
    
#Вопрос2
$ git show 85024
commit 85024d3100126de36331c6982bfaac02cdab9e76 (tag: v0.12.23)
Author: tf-release-bot <terraform@hashicorp.com>
Date:   Thu Mar 5 20:56:10 2020 +0000

    v0.12.23
    
#Вопрос3
$ git log --pretty=%P -n 1 b8d720
56cd7859e05c36c06b56d013b55a252d0bb7e158
9ea88f22fc6269854151c571162c5bcf958bee2b

#Вопрос4
$ git log  v0.12.23..v0.12.24  --oneline
33ff1c03b (tag: v0.12.24) v0.12.24
b14b74c49 [Website] vmc provider links
3f235065b Update CHANGELOG.md
6ae64e247 registry: Fix panic when server is unreachable
5c619ca1b website: Remove links to the getting started guide's old location
06275647e Update CHANGELOG.md
d5f9411f5 command: Fix bug when using terraform login on Windows
4b6d06cc5 Update CHANGELOG.md
dd01a3507 Update CHANGELOG.md
225466bc3 Cleanup after v0.12.23 release

#Вопрос5
$ git log -S'func providerSource' --oneline
5af1e6234 main: Honor explicit provider_installation CLI config when present
8c928e835 main: Consult local directories as potential mirrors of providers

$ git grep --break --heading -n -e'func providerSource'
provider_source.go
23:func providerSource(configs []*cliconfig.ProviderInstallation, services *disco.Disco) (get
providers.Source, tfdiags.Diagnostics) {
187:func providerSourceForCLIConfigLocation(loc cliconfig.ProviderInstallationLocation, servi
ces *disco.Disco) (getproviders.Source, tfdiags.Diagnostics) {

#Вопрос6
$ git grep --break --heading -n -e'globalPluginDirs'
commands.go
88:             GlobalPluginDirs: globalPluginDirs(),
430:    helperPlugins := pluginDiscovery.FindPlugins("credentials", globalPluginDirs())

internal/command/cliconfig/config_unix.go
34:             // FIXME: homeDir gets called from globalPluginDirs during init, before

plugins.go
12:// globalPluginDirs returns directories that should be searched for
18:func globalPluginDirs() []string {
$ git log -S globalPluginDirs --oneline
35a058fb3 main: configure credentials from the CLI config file
c0b176109 prevent log output during init
8364383c3 Push plugin discovery down into command package
$ git log -L :'func globalPluginDirs':plugins.go --oneline
78b122055 Remove config.go and update things using its aliases

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -16,14 +18,14 @@
 func globalPluginDirs() []string {
        var ret []string
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
-       dir, err := ConfigDir()
+       dir, err := cliconfig.ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
                machineDir := fmt.Sprintf("%s_%s", runtime.GOOS, runtime.GOARCH)
                ret = append(ret, filepath.Join(dir, "plugins"))
                ret = append(ret, filepath.Join(dir, "plugins", machineDir))
        }

        return ret
 }
52dbf9483 keep .terraform.d/plugins for discovery

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -16,13 +16,14 @@
 func globalPluginDirs() []string {
        var ret []string
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
        dir, err := ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
                machineDir := fmt.Sprintf("%s_%s", runtime.GOOS, runtime.GOARCH)
+               ret = append(ret, filepath.Join(dir, "plugins"))
                ret = append(ret, filepath.Join(dir, "plugins", machineDir))
        }

        return ret
 }
41ab0aef7 Add missing OS_ARCH dir to global plugin paths

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -14,12 +16,13 @@
 func globalPluginDirs() []string {
        var ret []string
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
        dir, err := ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
-               ret = append(ret, filepath.Join(dir, "plugins"))
+               machineDir := fmt.Sprintf("%s_%s", runtime.GOOS, runtime.GOARCH)
+               ret = append(ret, filepath.Join(dir, "plugins", machineDir))
        }

        return ret
 }
66ebff90c move some more plugin search path logic to command

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -16,22 +14,12 @@
 func globalPluginDirs() []string {
        var ret []string
-
-       // Look in the same directory as the Terraform executable.
-       // If found, this replaces what we found in the config path.
-       exePath, err := osext.Executable()
-       if err != nil {
-               log.Printf("[ERROR] Error discovering exe directory: %s", err)
-       } else {
-               ret = append(ret, filepath.Dir(exePath))
-       }
-
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
        dir, err := ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
                ret = append(ret, filepath.Join(dir, "plugins"))
        }

        return ret
 }
8364383c3 Push plugin discovery down into command package

diff --git a/plugins.go b/plugins.go
--- /dev/null
+++ b/plugins.go
@@ -0,0 +16,22 @@
+func globalPluginDirs() []string {
+       var ret []string
+
+       // Look in the same directory as the Terraform executable.
+       // If found, this replaces what we found in the config path.
+       exePath, err := osext.Executable()
+       if err != nil {
+               log.Printf("[ERROR] Error discovering exe directory: %s", err)
+       } else {
+               ret = append(ret, filepath.Dir(exePath))
+       }
+
+       // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
+       dir, err := ConfigDir()
+       if err != nil {
+               log.Printf("[ERROR] Error finding global config directory: %s", err)
+       } else {
+               ret = append(ret, filepath.Join(dir, "plugins"))
+       }
+
+       return ret
+}

#Вопрос7
$ git log -S'func synchronizedWriters' --pretty=format:'%h - %an %ae'
bdfea50cc - James Bardin j.bardin@gmail.com
5ac311e2a - Martin Atkins mart@degeneration.co.uk

## Домашнее задание к занятию "3.1. Работа в терминале, лекция 1"

1.	Установите средство виртуализации Oracle VirtualBox. 
Задание выполнялось на Windows путем нажатия кнопок Далие Готово
Так же выполнилось на Fedora34
# dnf install virtualbox
2.	Установите средство автоматизации Hashicorp Vagrant. 
Задание выполнялось на Windows скачал с сайта программу.
Так же выполнилось на Fedora34
# dnf install -y dnf-plugins-core
# dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
# dnf -y install vagrant

3.	В вашем основном окружении подготовьте удобный для дальнейшей работы терминал. 

Для работы выбран Windows Terminal.
4.	С помощью базового файла конфигурации запустите Ubuntu 20.04 в VirtualBox посредством Vagrant: 
В Windows:
Запускаем Windows Terminal 
cd .vagrant.d
vagrant init generic/centos8
vagrant up
vagrant ssh

5.	Ознакомьтесь с графическим интерфейсом VirtualBox, посмотрите как выглядит виртуальная машина, которую создал для вас Vagrant, какие аппаратные ресурсы ей выделены. Какие ресурсы выделены по-умолчанию? 

RAM:2048mb
CPU:2 cpu
HDD:128gb
video:256mb
6.	Ознакомьтесь с возможностями конфигурации VirtualBox через Vagrantfile: документация. Как добавить оперативной памяти или ресурсов процессора виртуальной машине? 
добавлением комманд в VagrantFile:
   config.vm.provider "virtualbox" do |v|
     vb.memory = 1024
     vb.cpu = 2
   end
  
7.	Команда vagrant ssh из директории, в которой содержится Vagrantfile, позволит вам оказаться внутри виртуальной машины без каких-либо дополнительных настроек. Попрактикуйтесь в выполнении обсуждаемых команд в терминале Ubuntu. 

выполнено    
8.	Ознакомиться с разделами man bash, почитать о настройках самого bash: 
какой переменной можно задать длину журнала history, и на какой строчке manual это описывается?
1. HISTFILESIZE - максимальное число строк в файле истории для сохранения, строка 1155
2. HISTSIZE - число команд для сохранения, строка 1178
что делает директива ignoreboth в bash?
ignoreboth это сокращение для 2х директив ignorespace and ignoredups 
    ignorespace - не сохранять команды, начинающиеся с пробела 
    ignoredups - не сохранять команду, если такая уже имеется в истории
9.	В каких сценариях использования применимы скобки {} и на какой строчке man bash это описано? 

{} - зарезервированные слова, список, в т.ч. список команд команд в отличии от "(...)" исполнятся в текущем инстансе, используется в различных условных циклах, условных операторах, или ограничивает тело функции, 
В командах выполняет подстановку элементов из списка, если упрощенно то  цикличное выполнение команд с подстановкой 
например mkdir ./DIR_{A..Z} - создаст каталоги сименами DIR_A, DIR_B и т.д. до DIR_Z
строка 343
10.	Основываясь на предыдущем вопросе, как создать однократным вызовом touch 100000 файлов? А получилось ли создать 300000? 

touch {000001..100000}.txt - создаст в текущей директории соответсвющее число фалов

300000 - создать не удасться, это слишком дилинный список аргументов, максимальное число получил экспериментально - 110188
11.	В man bash поищите по /\[\[. Что делает конструкция [[ -d /tmp ]] 

проверяет условие у -d /tmp и возвращает ее статус (0 или 1), наличие катаолга /tmp

Например в скрипте можно так:

if [[ -d /tmp ]]
then
    echo "каталог есть"
else
    echo "каталога нет"
fi
12.	Основываясь на знаниях о просмотре текущих (например, PATH) и установке новых переменных; командах, которые мы рассматривали, добейтесь в выводе type -a bash в виртуальной машине наличия первым пунктом в списке: 

[vagrant@centos8 ~]$ mkdir /tmp/new_path_dir/
[vagrant@centos8 ~]$ cp /bin/bash /tmp/new_path_dir/
[vagrant@centos8 ~]$ type -a bash
bash is /usr/bin/bash
[vagrant@centos8 ~]$ PATH=/tmp/new_path_dir/:$PATH
[vagrant@centos8 ~]$ type -a bash
bash is /tmp/new_path_dir/bash
bash is /usr/bin/bash
  
    
13.	Чем отличается планирование команд с помощью batch и at? 

at - команда запускается в указанное время (в параметре)
batch - запускается когда уровень загрузки системы снизится ниже 1.5.
14.	Завершите работу виртуальной машины чтобы не расходовать ресурсы компьютера и/или батарею ноутбука. 

Выполнено: vagrant suspend


