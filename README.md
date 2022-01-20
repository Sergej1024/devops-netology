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

# Домашнее задание к занятию "3.2. Работа в терминале, лекция 2"

1. Какого типа команда cd? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей, если считаете что она могла бы быть другого типа.

 Строго говоря, это вообще никакая не утилита. Ее нет в файловой системе. Это встроенная команда Bash и меняет текущую папку только для оболочки, в которой выполняется.
2. Какая альтернатива без pipe команде grep <some_string> <some_file> | wc -l? man grep поможет в ответе на этот вопрос. Ознакомьтесь с документом о других подобных некорректных вариантах использования pipe.

[vagrant@centos8 ~]$ cat tst_bash
fgjksbfkg;bgkdb
kjdgbjlgbled
12378245
[vagrant@centos8 ~]$ grep 12378245 tst_bash | wc -l
1
[vagrant@centos8 ~]$ grep 12378245 tst_bash -c
1
 Useless Use of Wc -l
This is my personal favorite. There is actually a whole class of "Useless Use of (something) | grep (something) | (something)" problems but this one usually manifests itself in scripts riddled by useless backticks and pretzel logic.

Anything that looks like

    	something | grep '..*' | wc -l

can usually be rewritten like something along the lines of

    	something | grep -c .   # Notice that . is better than '..*'


3. Какой процесс с PID 1 является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?
systemd(1)─┬─NetworkManager(905)─┬─{NetworkManager}(908)
           │                     └─{NetworkManager}(910)
           ├─VBoxService(1117)─┬─{VBoxService}(1118)
           │                   ├─{VBoxService}(1119)
           │                   ├─{VBoxService}(1121)
           │                   ├─{VBoxService}(1122)
           │                   ├─{VBoxService}(1123)
           │                   ├─{VBoxService}(1125)
           │                   ├─{VBoxService}(1126)
           │                   └─{VBoxService}(1127)
           ├─agetty(929)
           ├─auditd(803)───{auditd}(804)
           ├─chronyd(849)
           ├─crond(923)
           ├─dbus-daemon(833)───{dbus-daemon}(852)
           ├─firewalld(875)───{firewalld}(1021)
           ├─haveged(708)
           ├─irqbalance(827)───{irqbalance}(835)
           ├─polkitd(830)─┬─{polkitd}(863)
           │              ├─{polkitd}(864)
           │              ├─{polkitd}(868)
           │              ├─{polkitd}(869)
           │              └─{polkitd}(873)
           ├─rsyslogd(1027)─┬─{rsyslogd}(1033)
           │                └─{rsyslogd}(1034)
           ├─sshd(979)───sshd(37694)───sshd(37697)───bash(37698)───pstree(38096)
           ├─sssd(828)─┬─sssd_be(859)
           │           └─sssd_nss(872)
           ├─systemd(37616)───(sd-pam)(37619)
           ├─systemd-journal(710)
           ├─systemd-logind(891)
           ├─systemd-udevd(707)
           └─tuned(914)─┬─{tuned}(955)
                        ├─{tuned}(959)
                        ├─{tuned}(963)
                        └─{tuned}(965)
4. Как будет выглядеть команда, которая перенаправит вывод stderr ls на другую сессию терминала?
Вызов из pts/0:
ls -l 2>/dev/pts/1

5. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.
[vagrant@centos8 ~]$ cat tst_bash
fgjksbfkg;bgkdb
kjdgbjlgbled
12378245
[vagrant@centos8 ~]$ cat tst_bash_out
cat: tst_bash_out: No such file or directory
[root@centos8 vagrant]#  cat <tst_bash >tst_bash_out
[root@centos8 vagrant]# cat tst_bash_out
fgjksbfkg;bgkdb
kjdgbjlgbled
12378245
6. Получится ли находясь в графическом режиме, вывести данные из PTY в какой-либо из эмуляторов TTY? Сможете ли вы наблюдать выводимые данные?

Да, сможем, надо перенаправить поток на tty echo Hello, world! > /dev/tty2

7. Выполните команду bash 5>&1. К чему она приведет? Что будет, если вы выполните echo netology > /proc/$$/fd/5? Почему так происходит?

bash 5>&1 - Создаст дескриптор с 5 и перенатправит его в stdout
echo netology > /proc/$$/fd/5 - выведет в дескриптор "5", который был пернеаправлен в stdout
[root@centos8 vagrant]# bash 5>&1
[root@centos8 vagrant]# echo netology > /proc/$$/fd/5
netology
[root@centos8 vagrant]#

8. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от | на stdin команды справа. Это можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.

[root@centos8 vagrant]# ls -l /root 7>&2 2>&1 1>&7 |grep denied -c
total 0
0


7>&2 - новый дескриптор перенаправили в stderr
2>&1 - stderr перенаправили в stdout 
1>&7 - stdout - перенаправили в в новый дескриптор

9. Что выведет команда cat /proc/$$/environ? Как еще можно получить аналогичный по содержанию вывод?

Переменные окружения оболочки, их также можно получить с помощью команд env, printenv

10. Используя man, опишите что доступно по адресам /proc/<PID>/cmdline, /proc/<PID>/exe.

/proc/<PID>/cmdline - полный путь до исполняемого файла процесса [PID]  (строка 231)
/proc/<PID>/exe - содержит ссылку до файла запущенного для процесса [PID], cat выведет содержимое запущенного файла, запуск этого файла,  запустит еще одну копию самого файла  (строка 285)
    
11. Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью /proc/cpuinfo.
sse 4.2
    
12. При открытии нового окна терминала и vagrant ssh создается новая сессия и выделяется pty. Это можно подтвердить командой tty, которая упоминалась в лекции 3.2. Однако:

vagrant@netology1:~$ ssh localhost 'tty'
not a tty

Почитайте, почему так происходит, и как изменить поведение.
    
[vagrant@centos8 ~]$ ssh localhost 'tty'
The authenticity of host 'localhost (127.0.0.1)' can't be established.
RSA key fingerprint is SHA256:gMgAdx4zBi0Vw2RAc7p8MFi2v9ywt6GKMJikau5Af/E.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'localhost' (RSA) to the list of known hosts.
vagrant@localhost: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).

но с локальной машины выдеает так
 [root@homesrv ~]#  ssh localhost 'tty'
The authenticity of host 'localhost (::1)' can't be established.
ECDSA key fingerprint is SHA256:QLP2BaHZ78GbBnbkDI2AQSKuWlyQmQWvq+9K8bzdlPU.
ECDSA key fingerprint is MD5:45:77:ad:39:29:15:60:dd:ce:04:32:04:be:3a:99:4b.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
root@localhost's password:
не телетайп
    
   если добавить праметр -t то получится так, команда принудительно создаст псевдопользователя
[root@homesrv ~]#  ssh -t localhost 'tty'
root@localhost's password:
/dev/pts/1
Connection to localhost closed.   

13. Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись reptyr. Например, так можно перенести в screen процесс, который вы запустили по ошибке в обычной SSH-сессии.

    
   Пререквизиты: есть reptyrи tmux/ screen установлены; Вы сможете найти их с apt-getили yum, в зависимости от вашей платформы.

    Используйте Ctrl+, Z чтобы приостановить процесс.

    Возобновите процесс в фоновом режиме с bg

    Найдите идентификатор процесса фонового процесса с помощью jobs -l

    Вы увидите нечто похожее на это:

    [1]+ 11475 Stopped (signal) yourprocessname

    Отключить работу от текущего родителя (оболочки) с disown yourprocessname

    Начало tmux(предпочтительно) или screen.

    Снова подключите процесс к tmux/ screenсеансу с reptyr:

    reptyr 11475

    Теперь вы можете отсоединить мультиплексор (по умолчанию Ctrl+ B, D для tmuxили Ctrl+ A, D для screen) и отключить SSH, пока ваш процесс продолжается в tmux/ screen.

    Позже, когда вы снова подключитесь к SSH, вы сможете подключиться к мультиплексору (например tmux attach).

 
    
14. sudo echo string > /root/new_file не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без sudo под вашим пользователем. Для решения данной проблемы можно использовать конструкцию echo string | sudo tee /root/new_file. Узнайте что делает команда tee и почему в отличие от sudo echo команда с sudo tee будет работать.
    
команда tee делает вывод одновременно и в файл, указаный в качестве параметра, и в stdout, в данном примере команда получает вывод из stdin, перенаправленный через pipe от stdout команды echo и так как команда запущена от sudo , соотвественно имеет права на запись в файл

    
# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

 
## 1.   Какой системный вызов делает команда cd? В прошлом ДЗ мы выяснили, что cd не является самостоятельной программой, это shell builtin, поэтому запустить strace непосредственно на cd не получится. Тем не менее, вы можете запустить strace на /bin/bash -c 'cd /tmp'. В этом случае вы увидите полный список системных вызовов, которые делает сам bash при старте. Вам нужно найти тот единственный, который относится именно к cd.
    newfstatat(AT_FDCWD, "/tmp", {st_mode=S_IFDIR|S_ISVTX|0777, st_size=520, ...}, 0) = 0
    chdir("/tmp") 

## 2.   Попробуйте использовать команду file на объекты разных типов на файловой системе. Например:

    vagrant@netology1:~$ file /dev/tty
    /dev/tty: character special (5/0)
    vagrant@netology1:~$ file /dev/sda
    /dev/sda: block special (8/0)
    vagrant@netology1:~$ file /bin/bash
    /bin/bash: ELF 64-bit LSB shared object, x86-64

    Используя strace выясните, где находится база данных file на основании которой она делает свои догадки.
    
    Файл базы типов - /usr/share/misc/magic.mgc
    в тексте это:
    
    newfstatat(AT_FDCWD, "/home/sergej/.magic.mgc", 0x7ffc464fe630, 0) = -1 ENOENT (Нет такого файла или каталога)
    newfstatat(AT_FDCWD, "/home/sergej/.magic", 0x7ffc464fe630, 0) = -1 ENOENT (Нет такого файла или каталога)
    openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (Нет такого файла или каталога)
    newfstatat(AT_FDCWD, "/etc/magic", {st_mode=S_IFREG|0644, st_size=111, ...}, 0) = 0
    openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3
    newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=111, ...}, AT_EMPTY_PATH) = 0
    read(3, "# Magic local data for file(1) c"..., 4096) = 111
    read(3, "", 4096)                       = 0
    close(3)                                = 0
    openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
    newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=6652944, ...}, AT_EMPTY_PATH) = 0
    mmap(NULL, 6652944, PROT_READ|PROT_WRITE, MAP_PRIVATE, 3, 0) = 0x7fd1eda3d000

    
## 3.   Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).
    
    [sergej@surg-adm ~]$ lsof | grep /tmp/test1
    ping      301543                           sergej    1w      REG               0,35     18447       1656 /tmp/test1
    [sergej@surg-adm ~]$ lsof | grep /tmp/test1
    ping      301543                           sergej    1w      REG               0,35     24423       1656 /tmp/test1 (deleted)
    [sergej@surg-adm ~]$ cat /proc/301543/fd/1 > /tmp/test1
    [sergej@surg-adm ~]$ cat /tmp/test1
    PING ya.ru (87.250.250.242) 56(84) bytes of data.
    64 bytes from ya.ru (87.250.250.242): icmp_seq=1 ttl=249 time=49.3 ms
    64 bytes from ya.ru (87.250.250.242): icmp_seq=2 ttl=249 time=49.1 ms
    64 bytes from ya.ru (87.250.250.242): icmp_seq=3 ttl=249 time=49.0 ms
    64 bytes from ya.ru (87.250.250.242): icmp_seq=4 ttl=249 time=48.9 ms
    64 bytes from ya.ru (87.250.250.242): icmp_seq=5 ttl=249 time=49.0 ms
    64 bytes from ya.ru (87.250.250.242): icmp_seq=6 ttl=249 time=48.9 ms
    64 bytes from ya.ru (87.250.250.242): icmp_seq=7 ttl=249 time=48.9 ms
    64 bytes from ya.ru (87.250.250.242): icmp_seq=8 ttl=249 time=49.2 ms
    64 bytes from ya.ru (87.250.250.242): icmp_seq=9 ttl=249 time=49.1 ms
    ...
    [sergej@surg-adm ~]$ ls -l /tmp
    итого 100
    -rw-r--r-- 1 sergej sergej    94 янв 18 12:34 847bf70473fa93942054bb1d51dd24d6-{87A94AB0-E370-4cde-98D3-ACC110C5967D
    -rw-rw-r-- 1 sergej sergej 35511 янв 19 10:47 test1

    _Пояснения_
    в первом выводе видем запущеный процес пинга в файл что он работает
    во втором выводе видем что процес работает, а сам файл удален
    и далие из процесса делаем запись в файл и выводим файл.
    
    
## 4.   Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?
    "Зомби" процессы, в отличии от "сирот" освобождают свои ресурсы, но не освобождают запись в таблице процессов. Запись освободиться при вызове wait() родительским процессом.
    
## 5.   В iovisor BCC есть утилита opensnoop:

    root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
    /usr/sbin/opensnoop-bpfcc

    На какие файлы вы увидели вызовы группы open за первую секунду работы утилиты? Воспользуйтесь пакетом bpfcc-tools для Ubuntu 20.04. Дополнительные сведения по установке.
    [sergej@surg-adm tools]$ sudo dnf install bcc
    [sergej@surg-adm ~]$ cd /usr/share/bcc/tools
    [sergej@surg-adm tools]$ sudo ./opensnoop
    [sudo] пароль для sergej: 
    PID    COMM               FD ERR PATH
    889    systemd-oomd        6   0 /proc/meminfo
    2392   Viber              86   0 /usr/share/zoneinfo/Asia/Yekaterinburg
    2392   Viber              86   0 /usr/share/zoneinfo/Asia/Yekaterinburg
    2392   Viber              86   0 /usr/share/zoneinfo/Asia/Yekaterinburg
    2392   Viber              86   0 /usr/share/zoneinfo/Asia/Yekaterinburg
    889    systemd-oomd        6   0 /proc/meminfo
    934    in:imjournal       29   0 /var/log/journal/8c20bed4bb5c404a94103f87f30bccc5/system.journal
    1221   abrt-dump-journ     6   0 /var/log/journal/8c20bed4bb5c404a94103f87f30bccc5/system.journal
    1220   abrt-dump-journ     6   0 /var/log/journal/8c20bed4bb5c404a94103f87f30bccc5/system.journal
    1236   abrt-dump-journ     6   0 /var/log/journal/8c20bed4bb5c404a94103f87f30bccc5/system.journal
    7091   gnome-terminal-    14   0 /tmp

    
## 6.   Какой системный вызов использует uname -a? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в /proc, где можно узнать версию ядра и релиз ОС.
    
    системный вызов uname()

    Цитата :

         Part of the utsname information is also accessible  via  /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.

    cat /etc/redhat-release выводит версию и релиз
    
## 7.   Чем отличается последовательность команд через ; и через && в bash? Например:

    root@netology1:~# test -d /tmp/some_dir; echo Hi
    Hi
    root@netology1:~# test -d /tmp/some_dir && echo Hi
    root@netology1:~#

    Есть ли смысл использовать в bash &&, если применить set -e?
    
    && -  условный оператор, 
    ;  - разделитель последовательных команд

    test -d /tmp/some_dir && echo Hi - в данном случае echo  отработает только при успешном заверщении команды test

    set -e - прерывает сессию при любом ненулевом значении исполняемых команд в конвеере кроме последней. В случае &&  вместе с set -e- вероятно не имеет смысла, так как при ошибке, выполнение команд прекратиться.
    
## 8.   Из каких опций состоит режим bash set -euxo pipefail и почему его хорошо было бы использовать в сценариях?
    -e прерывает выполнение исполнения при ошибке любой команды кроме последней в последовательности 
    -x вывод трейса простых команд 
    -u неустановленные/не заданные параметры и переменные считаются как ошибки, с выводом в stderr текста ошибки и выполнит завершение неинтерактивного вызова
    -o pipefail возвращает код возврата набора/последовательности команд, ненулевой при последней команды или 0 для успешного выполнения команд.

    Повышает деталезацию вывода ошибок(логирования), и завершит сценарий при наличии ошибок, на любом этапе выполнения сценария, кроме последней завершающей команды.
    
## 9.   Используя -o stat для ps, определите, какой наиболее часто встречающийся статус у процессов в системе. В man ps ознакомьтесь (/PROCESS STATE CODES) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).
    Самый часто встречающийся статус это - S

   PROCESS STATE CODES
       Here are the different values that the s, stat and state output specifiers (header "STAT" or "S") will display to describe the state of a process:
       D    uninterruptible sleep (usually IO)
       R    running or runnable (on run queue)
       S    interruptible sleep (waiting for an event to complete)
       T    stopped, either by a job control signal or because it is being traced.
       W    paging (not valid since the 2.6.xx kernel)
       X    dead (should never be seen)
       Z    defunct ("zombie") process, terminated but not reaped by its parent.

       For BSD formats and when the stat keyword is used, additional characters may be displayed:
       <    high-priority (not nice to other users)
       N    low-priority (nice to other users)
       L    has pages locked into memory (for real-time and custom IO)
       s    is a session leader
       l    is multi-threaded (using CLONE_THREAD, like NPTL pthreads do)
       +    is in the foreground process group.
