# ZFS
## Описание/Пошаговая инструкция выполнения домашнего задания:
1. Определить алгоритм с наилучшим сжатием. Затем: отрабатываем навыки работы с созданием томов и установкой параметров. Находим наилучшее сжатие. Шаги:
* определить какие алгоритмы сжатия поддерживает zfs (gzip gzip-N, zle lzjb, lz4);
* создать 4 файловых системы на каждой применить свой алгоритм сжатия; Для сжатия использовать либо текстовый файл либо группу файлов:
* скачать файл “Война и мир” и расположить на файловой системе wget -O War_and_Peace.txt http://www.gutenberg.org/ebooks/2600.txt.utf-8, либо скачать файл ядра распаковать и расположить на файловой системе. Результат:
* список команд которыми получен результат с их выводами;
* вывод команды из которой видно какой из алгоритмов лучше.
2. Определить настройки pool’a. Зачем: для переноса дисков между системами используется функция export/import. Отрабатываем навыки работы с файловой системой ZFS. Шаги:
* загрузить архив с файлами локально. https://drive.google.com/open id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg Распаковать.
* с помощью команды zfs import собрать pool ZFS;
* командами zfs определить настройки:
* размер хранилища;
* тип pool;
* значение recordsize;
* какое сжатие используется;
* какая контрольная сумма используется. Результат:
* список команд которыми восстановили pool . Желательно с Output команд;
* файл с описанием настроек settings.
3. Найти сообщение от преподавателей. Зачем: для бэкапа используются технологии snapshot. Snapshot можно передавать между хостами и восстанавливать с помощью send/receive. Отрабатываем навыки восстановления snapshot и переноса файла. Шаги:
* скопировать файл из удаленной директории. https://drive.google.com/file/d/1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG/view?usp=sharing Файл был получен командой zfs send otus/storage@task2 > otus_task2.file
* восстановить файл локально. zfs receive
* найти зашифрованное сообщение в файле secret_message Результат:
* список шагов которыми восстанавливали;
* зашифрованное сообщение.

# Выполнение домашнего задания:

 ПК на Unix c 8ГБ ОЗУ или виртуальная машина с включенной Nested
Virtualization.

 Предварительно установленное и настроенное следующее ПО:

 Hashicorp Vagrant (https://www.vagrantup.com/downloads)

 Oracle VirtualBox (https://www.virtualbox.org/wiki/Linux_Downloads).

 Установка виртуальной машины - ```Vagrant up server```

 После установки заходим в нее - ```Vagrant ssh server``` 

 Переходим в root пользователя - ```sudo su```

 Проверим версию установленной ZFS 

  ```[root@server vagrant]# zfs --version```

 >zfs-2.0.7-1

 >zfs-kmod-2.0.7-1

 ## 1. Определение алгоритма с наилучшим сжатием

 ```[root@server vagrant]# lsblk```

 >     NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
 >     sda      8:0    0   64G  0 disk
 >     ├─sda1   8:1    0  2.1G  0 part [SWAP]
 >     └─sda2   8:2    0 61.9G  0 part /
 >     sdb      8:16   0    1G  0 disk
 >     sdc      8:32   0    1G  0 disk
 >     sdd      8:48   0    1G  0 disk
 >     sde      8:64   0    1G  0 disk
 >     sdf      8:80   0    1G  0 disk
 >     sdg      8:96   0    1G  0 disk
 >     sdh      8:112  0    1G  0 disk
 >     sdi      8:128  0    1G  0 disk

 Создаём 4 пула из 8 дисков в режиме RAID 1:

 ```[root@server vagrant]# zpool create otus1 mirror /dev/sdb /dev/sdc```

```[root@server vagrant]# zpool create otus2 mirror /dev/sdd /dev/sde```

```[root@server vagrant]# zpool create otus3 mirror /dev/sdf /dev/sdg```

```[root@server vagrant]# zpool create otus4 mirror /dev/sdh /dev/sdi```

Смотрим информацию о пулах: 

```[root@server vagrant]# zpool list```

>     NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
>     otus1   960M   105K   960M        -         -     0%     0%  1.00x    ONLINE  -
>     otus2   960M   105K   960M        -         -     0%     0%  1.00x    ONLINE  -
>     otus3   960M   105K   960M        -         -     0%     0%  1.00x    ONLINE  -
>     otus4   960M   105K   960M        -         -     0%     0%  1.00x    ONLINE  -
>[root@server vagrant]#

Команда ```zpool status``` показывает информацию о каждом диске, состоянии
сканирования и об ошибках чтения, записи и совпадения хэш-сумм. Команда
```zpool list``` показывает информацию о размере пула, количеству занятого и
свободного места, дедупликации и т.д.

Добавим разные алгоритмы сжатия в каждую файловую систему:
* Алгоритм lzjb: ```zfs set compression=lzjb otus1```
* Алгоритм lz4: ```zfs set compression=lz4 otus2```
* Алгоритм gzip: ```zfs set compression=gzip-9 otus3```
* Алгоритм zle: ```zfs set compression=zle otus4```

```[root@server vagrant]# zfs set compression=lzjb otus1```

```[root@server vagrant]# zfs set compression=lz4 otus2```

```[root@server vagrant]#  zfs set compression=gzip-9 otus3```

```[root@server vagrant]# zfs set compression=zle otus4```

Проверим, что все файловые системы имеют разные методы сжатия:

```[root@server vagrant]# zfs get all | grep compression```

>     otus1 compression lzjb local
>     otus2 compression lz4 local
>     otus3 compression gzip-9 local
>     otus4 compression zle local

Сжатие файлов будет работать только с файлами, которые были добавлены
после включение настройки сжатия.

Скачаем один и тотже файл во все пулы от файла война и мир:

```[root@server vagrant]# for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done```

>    --2022-05-23 05:12:19--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40809473 (39M) [text/plain]
Saving to: ‘/otus1/pg2600.converter.log’

>  pg2600.converter.log            100%[=======================================================>]  38.92M  4.73MB/s    in 8.5s

> 2022-05-23 05:12:29 (4.57 MB/s) - ‘/otus1/pg2600.converter.log’ saved [40809473/40809473]

> --2022-05-23 05:12:29--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40809473 (39M) [text/plain]
Saving to: ‘/otus2/pg2600.converter.log’

> pg2600.converter.log            100%[=======================================================>]  38.92M  9.25MB/s    in 5.3s

> 2022-05-23 05:12:34 (7.37 MB/s) - ‘/otus2/pg2600.converter.log’ saved [40809473/40809473]

> --2022-05-23 05:12:34--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40809473 (39M) [text/plain]
Saving to: ‘/otus3/pg2600.converter.log’

> pg2600.converter.log            100%[=======================================================>]  38.92M  4.42MB/s    in 9.5s

> 2022-05-23 05:12:45 (4.11 MB/s) - ‘/otus3/pg2600.converter.log’ saved [40809473/40809473]

> --2022-05-23 05:12:45--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40809473 (39M) [text/plain]
Saving to: ‘/otus4/pg2600.converter.log’

> pg2600.converter.log            100%[=======================================================>]  38.92M  7.48MB/s    in 6.2s

> 2022-05-23 05:12:51 (6.27 MB/s) - ‘/otus4/pg2600.converter.log’ saved [40809473/40809473]

> [root@server vagrant]#


Проверим, что файл был скачан во все пулы:


```[root@server vagrant]# ll /otus*```

>     /otus1:
>     total 22020
>     -rw-r--r--. 1 root root 40809473 May  2 08:01 pg2600.converter.log

>     /otus2:
>     total 17972
>     -rw-r--r--. 1 root root 40809473 May  2 08:01 pg2600.converter.log

>     /otus3:
>     total 10949
>     -rw-r--r--. 1 root root 40809473 May  2 08:01 pg2600.converter.log

>     /otus4:
>     total 39881
>     -rw-r--r--. 1 root root 40809473 May  2 08:01 pg2600.converter.log
>     [root@server vagrant]#
 
Уже на этом этапе видно, что самый оптимальный метод сжатия у нас
используется в пуле otus3.

Проверим, сколько места занимает один и тот же файл в разных пулах и
проверим степень сжатия файлов:

```[root@server vagrant]# zfs list```
>     NAME    USED  AVAIL     REFER  MOUNTPOINT
>     otus1  21.8M   810M     21.5M  /otus1
>     otus2  17.7M   814M     17.6M  /otus2
>     otus3  10.8M   821M     10.7M  /otus3
>     otus4  39.1M   793M     39.0M  /otus4
>     [root@server vagrant]#

```[root@server vagrant]# zfs get all | grep compressratio | grep -v ref```
>     otus1  compressratio         1.81x                  -
>     otus2  compressratio         2.22x                  -
>     otus3  compressratio         3.64x                  -
>     otus4  compressratio         1.00x                  -
Таким образом, у нас получается, что алгоритм gzip-9 самый эффективный
по сжатию.

## 2. Определение настроек пула
Скачиваем архив в домашний каталог:
[root@server vagrant]# wget -O archive.tar.gz --no-check-certificate https://drive.google.com/u/0/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg&export=download

>--2022-05-23 05:30:11--  https://drive.google.com/u/0/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg
Resolving drive.google.com (drive.google.com)... 142.250.184.206
Connecting to drive.google.com (drive.google.com)|142.250.184.206|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://drive.google.com/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg [following]
--2022-05-23 05:30:12--  https://drive.google.com/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg
Reusing existing connection to drive.google.com:443.
HTTP request sent, awaiting response... 303 See Other
Location: https://doc-0c-bo-docs.googleusercontent.com/docs/securesc/ha0ro937gcuc7l7deffksulhg5h7mbp1/h5tsace6lsospsj73s7t5jk780fg4us7/1653283800000/16189157874053420687/*/1KRBNW33QWqbvbVHa3hLJivOAt60yukkg [following]
Warning: wildcards not supported in HTTP.
--2022-05-23 05:30:16--  https://doc-0c-bo-docs.googleusercontent.com/docs/securesc/ha0ro937gcuc7l7deffksulhg5h7mbp1/h5tsace6lsospsj73s7t5jk780fg4us7/1653283800000/16189157874053420687/*/1KRBNW33QWqbvbVHa3hLJivOAt60yukkg
Resolving doc-0c-bo-docs.googleusercontent.com (doc-0c-bo-docs.googleusercontent.com)... 142.250.185.161
Connecting to doc-0c-bo-docs.googleusercontent.com (doc-0c-bo-docs.googleusercontent.com)|142.250.185.161|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7275140 (6.9M) [application/x-gzip]
Saving to: ‘archive.tar.gz’

>archive.tar.gz                  100%[=======================================================>]   6.94M  38.6MB/s    in 0.2s

>2022-05-23 05:30:17 (38.6 MB/s) - ‘archive.tar.gz’ saved [7275140/7275140]

Разархивируем его:

```[root@server vagrant]# tar -xzvf archive.tar.gz```
>     zpoolexport/
>     zpoolexport/filea
>     zpoolexport/fileb
>     [root@server vagrant]#

Проверим, возможно ли импортировать данный каталог в пул:

```[root@server vagrant]# zpool import -d zpoolexport/```
>      pool: otus
>           id: 6554193320433390805
>        state: ONLINE
>      status: Some supported features are not enabled on the pool.
>       action: The pool can be imported using its name or numeric identifier, though
>              some features will not be available without an explicit 'zpool >      upgrade'.
>       config:
>              otus                                 ONLINE
>                mirror-0                           ONLINE
>                  /home/vagrant/zpoolexport/filea  ONLINE
>                  /home/vagrant/zpoolexport/fileb  ONLINE

Данный вывод показывает нам имя пула, тип raid и его состав.

Сделаем импорт данного пула к нам в ОС:

```[root@server vagrant]# zpool import -d zpoolexport/ otus```

```[root@server vagrant]# zpool status otus```

 >      pool: otus
 >      state: ONLINE
>      status: Some supported features are not enabled on the pool. The pool can
>              still be used, but some features are unavailable.
>      action: Enable all features using 'zpool upgrade'. Once this is done,
>              the pool may no longer be accessible by software that does not support
>              the features. See zpool-features(5) for details.
>      config:
>              NAME                                 STATE     READ WRITE CKSUM
>              otus                                 ONLINE       0     0     0
>                mirror-0                           ONLINE       0     0     0
>                  /home/vagrant/zpoolexport/filea  ONLINE       0     0     0
>                  /home/vagrant/zpoolexport/fileb  ONLINE       0     0     0
>      
>      errors: No known data errors
>      [root@server vagrant]#

Команда zpool status выдаст нам информацию о составе импортированного
пула.

Если у Вас уже есть пул с именем otus, то можно поменять его имя во
время импорта: ```zpool import -d zpoolexport/ otus newotus```

Далее нам нужно определить настройки

Запрос сразу всех параметров пула: ```zpool get all otus```

Запрос сразу всех параметром файловой системы: ```zfs get all otus```

```[root@server vagrant]#  zfs get all otus```
>     NAME  PROPERTY              VALUE                  SOURCE
>     otus  type                  filesystem             -
>     otus  creation              Fri May 15  4:00 2020  -
>     otus  used                  2.04M                  -
>     otus  available             350M                   -
>     otus  referenced            24K                    -
>     otus  compressratio         1.00x                  -
>     otus  mounted               yes                    -
>     otus  quota                 none                   default
>     otus  reservation           none                   default
>     otus  recordsize            128K                   local
>     otus  mountpoint            /otus                  default
>     otus  sharenfs              off                    default
>     otus  checksum              sha256                 local
>     otus  compression           zle                    local
>     otus  atime                 on                     default
>     otus  devices               on                     default
>     otus  exec                  on                     default
>     otus  setuid                on                     default
>     otus  readonly              off                    default
>     otus  zoned                 off                    default
>     otus  snapdir               hidden                 default
>     otus  aclmode               discard                default
>     otus  aclinherit            restricted             default
>     otus  createtxg             1                      -
>     otus  canmount              on                     default
>     otus  xattr                 on                     default
>     otus  copies                1                      default
>     otus  version               5                      -
>     otus  utf8only              off                    -
>     otus  normalization         none                   -
>     otus  casesensitivity       sensitive              -
>     otus  vscan                 off                    default
>     otus  nbmand                off                    default
>     otus  sharesmb              off                    default
>     otus  refquota              none                   default
>     otus  refreservation        none                   default
>     otus  guid                  14592242904030363272   -
>     otus  primarycache          all                    default
>     otus  secondarycache        all                    default
>     otus  usedbysnapshots       0B                     -
>     otus  usedbydataset         24K                    -
>     otus  usedbychildren        2.01M                  -
>     otus  usedbyrefreservation  0B                     -
>     otus  logbias               latency                default
>     otus  objsetid              54                     -
>     otus  dedup                 off                    default
>     otus  mlslabel              none                   default
>     otus  sync                  standard               default
>     otus  dnodesize             legacy                 default
>     otus  refcompressratio      1.00x                  -
>     otus  written               24K                    -
>     otus  logicalused           1020K                  -
>     otus  logicalreferenced     12K                    -
>     otus  volmode               default                default
>     otus  filesystem_limit      none                   default
>     otus  snapshot_limit        none                   default
>     otus  filesystem_count      none                   default
>     otus  snapshot_count        none                   default
>     otus  snapdev               hidden                 default
>     otus  acltype               off                    default
>     otus  context               none                   default
>     otus  fscontext             none                   default
>     otus  defcontext            none                   default
>     otus  rootcontext           none                   default
>     otus  relatime              off                    default
>     otus  redundant_metadata    all                    default
>     otus  overlay               on                     default
>     otus  encryption            off                    default
>     otus  keylocation           none                   default
>     otus  keyformat             none                   default
>     otus  pbkdf2iters           0                      default
>     otus  special_small_blocks  0                      default
>     [root@server vagrant]#

C помощью команды grep можно уточнить конкретный параметр, например:

Размер: ```zfs get available otus```
```[root@server vagrant]# zfs get available otus```
>     NAME  PROPERTY   VALUE  SOURCE
>     otus  available  350M   -

Тип: ```zfs get readonly otus```

```[root@server vagrant]# zfs get readonly otus```
>     NAME  PROPERTY  VALUE   SOURCE
>     otus  readonly  off     default

По типу FS мы можем понять, что позволяет выполнять чтение и запись

Значение recordsize: ```zfs get recordsize otus```

```[root@server vagrant]# zfs get recordsize otus```

>     NAME  PROPERTY    VALUE    SOURCE
>     otus  recordsize  128K     local

Тип сжатия (или параметр отключения): ```zfs get compression otus```

```[root@server vagrant]# zfs get compression otus```

>     NAME  PROPERTY     VALUE           SOURCE
>     otus  compression  zle             local

Тип контрольной суммы: ```zfs get checksum otus```

```[root@server vagrant]# zfs get checksum otus```

>     NAME  PROPERTY  VALUE      SOURCE
>     otus  checksum  sha256     local

## 3. Работа со снапшотом, поиск сообщения от преподавателя

Скачаем файл, указанный в задании:

```[root@server vagrant]# wget -O otus_task2.file --no-check-certificate https://drive.google.com/u/0/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG&export=download```

>     [1] 23247
>     [root@server vagrant]#
>     Redirecting output to ‘wget-log.1’.
>     [1]+  Done                    wget -O otus_task2.file --no-check-certificate https://drive.google.com/u/0/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG
>     [root@server vagrant]# ]

```[root@server vagrant]# cat wget-log.1```

>     --2022-05-23 06:13:09--  https://drive.google.com/u/0/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG
>     Resolving drive.google.com (drive.google.com)... 142.250.186.174
>     Connecting to drive.google.com (drive.google.com)|142.250.186.174|:443... connected.
>     HTTP request sent, awaiting response... 302 Found
>     Location: https://drive.google.com/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG [following]
>     --2022-05-23 06:13:09--  https://drive.google.com/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG
>     Reusing existing connection to drive.google.com:443.
>     HTTP request sent, awaiting response... 303 See Other
>     Location: https://doc-00-bo-docs.googleusercontent.com/docs/securesc/ha0ro937gcuc7l7deffksulhg5h7mbp1/nq1gihlvn8cu6cfi25p0igldpfeb5bj0/1653286350000/16189157874053420687/*/1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG [following]
>     Warning: wildcards not supported in HTTP.
>     --2022-05-23 06:13:10--  https://doc-00-bo-docs.googleusercontent.com/docs/securesc/ha0ro937gcuc7l7deffksulhg5h7mbp1/nq1gihlvn8cu6cfi25p0igldpfeb5bj0/1653286350000/16189157874053420687/*/1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG
>     Resolving doc-00-bo-docs.googleusercontent.com (doc-00-bo-docs.googleusercontent.com)... 142.250.186.161
>     Connecting to doc-00-bo-docs.googleusercontent.com (doc-00-bo-docs.googleusercontent.com)|142.250.186.161|:443... connected.
>     HTTP request sent, awaiting response... 200 OK
>     Length: 5432736 (5.2M) [application/octet-stream]
>     Saving to: ‘otus_task2.file’

>     otus_task2.file                 100%[=======================================================>]   5.18M  23.9MB/s    in 0.2s
>     2022-05-23 06:13:10 (23.9 MB/s) - ‘otus_task2.file’ saved [5432736/5432736]
>     [root@server vagrant]#

Восстановим файловую систему из снапшота: 

```[root@server vagrant]# zfs receive otus/test@today < otus_task2.file```

Далее, ищем в каталоге /otus/test файл с именем “secret_message”:

```[root@server vagrant]# find /otus/test -name "secret_message"```

>     /otus/test/task1/file_mess/secret_message

Смотрим содержимое найденного файла:

```[root@server vagrant]# cat /otus/test/task1/file_mess/secret_message```
>     https://github.com/sindresorhus/awesome

Тут мы видим ссылку на GitHub, можем скопировать её в адресную строку и
посмотреть репозиторий.











