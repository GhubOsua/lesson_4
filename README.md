# Урок 4. Задание 1.
В репозитории находятся файлы [Vagrantfile](Vagrantfile), README.

## Описание Vagrantfile:
1. В файле находится параметры вирт. машини и описание решения домашнего задания;
2. Использовал созданную переменную Virtual, для переноса HDD на другой диск;

## Описание решения Задания 1:

### 1. Определить алгоритм с наилучшим сжатием:
1. Загрузка вирт. машины, обновление ОС. virtual up , yum update -y;
2. Создаем zfs pool с именем myraidz1, тип raidz1 на указанных дисках. create myraidz1 raidz1 /dev/sd[b,c,d,e];

```
[root@lesson4 vagrant]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0    1G  0 disk 
├─sdb1                    8:17   0 1014M  0 part 
└─sdb9                    8:25   0    8M  0 part 
sdc                       8:32   0    1G  0 disk 
├─sdc1                    8:33   0 1014M  0 part 
└─sdc9                    8:41   0    8M  0 part 
sdd                       8:48   0    1G  0 disk 
├─sdd1                    8:49   0 1014M  0 part 
└─sdd9                    8:57   0    8M  0 part 
sde                       8:64   0    1G  0 disk 
├─sde1                    8:65   0 1014M  0 part 
└─sde9                    8:73   0    8M  0 part 
```
```
[root@lesson4 vagrant]# zpool list
NAME       SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
myraidz1  3.75G   415K  3.75G        -         -     0%     0%  1.00x    ONLINE  -
[root@lesson4 vagrant]# zpool status
  pool: myraidz1
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	myraidz1    ONLINE       0     0     0
	  raidz1-0  ONLINE       0     0     0
	    sdb     ONLINE       0     0     0
	    sdc     ONLINE       0     0     0
	    sdd     ONLINE       0     0     0
	    sde     ONLINE       0     0     0

errors: No known data errors

```
3. Создаем 4 фс поверх пула myraidz1, с именем fs_1, fs_2, fs_3, fs_4;

```
[root@lesson4 vagrant]# zfs create myraidz1/fs_1
[root@lesson4 vagrant]# zfs create myraidz1/fs_2
[root@lesson4 vagrant]# zfs create myraidz1/fs_3
[root@lesson4 vagrant]# zfs create myraidz1/fs_4
```
```
[root@lesson4 vagrant]# df -h
Filesystem                       Size  Used Avail Use% Mounted on
devtmpfs                         1.8G     0  1.8G   0% /dev
tmpfs                            1.9G     0  1.9G   0% /dev/shm
tmpfs                            1.9G  8.7M  1.8G   1% /run
tmpfs                            1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/VolGroup00-LogVol00   38G  2.2G   36G   6% /
/dev/sda2                       1014M   86M  929M   9% /boot
vagrant                          112G   69G   44G  62% /vagrant
tmpfs                            371M     0  371M   0% /run/user/1000
tmpfs                            371M     0  371M   0% /run/user/0
myraidz1                         2.7G  128K  2.7G   1% /myraidz1
myraidz1/fs_1                    2.7G  128K  2.7G   1% /myraidz1/fs_1
myraidz1/fs_2                    2.7G  128K  2.7G   1% /myraidz1/fs_2
myraidz1/fs_3                    2.7G  128K  2.7G   1% /myraidz1/fs_3
myraidz1/fs_4                    2.7G  128K  2.7G   1% /myraidz1/fs_4

```
4. Необходимо опреределить какие алгоритмы поддерживает zfs;

* 4.1. Можно через команда zfs get;

* 4.2. Либо man zfs | grep 'compression=on'
compression=on|off|gzip|gzip-N|lz4|lzjb|zle

5. Процесс сжатия фс;
 
* 5.1. Устанавливаем сжатие для фс;
 
```
[root@lesson4 vagrant]# zfs set compression=gzip-9 myraidz1/fs_1
[root@lesson4 vagrant]# zfs get compression
NAME           PROPERTY     VALUE     SOURCE
myraidz1       compression  off       default
myraidz1/fs_1  compression  gzip-9    local
myraidz1/fs_2  compression  off       default
myraidz1/fs_3  compression  off       default
myraidz1/fs_4  compression  off       default
[root@lesson4 vagrant]# zfs set compression=zle myraidz1/fs_2
[root@lesson4 vagrant]# zfs set compression=lz4 myraidz1/fs_3
[root@lesson4 vagrant]# zfs set compression=lzjb myraidz1/fs_4
[root@lesson4 vagrant]# zfs get compression
NAME           PROPERTY     VALUE     SOURCE
myraidz1       compression  off       default
myraidz1/fs_1  compression  gzip-9    local
myraidz1/fs_2  compression  zle       local
myraidz1/fs_3  compression  lz4       local
myraidz1/fs_4  compression  lzjb      local
[root@lesson4 vagrant]# ^C
```

* 5.2. Вывод результата сжатия в фс. Скачено два файла .txt - "Война и мир", 2600.txt.utf-8. 

    * 5.2.1. На каждой ФС wget wget -o log https://raw.githubusercontent.com/NSkelsey/cvf/master/war_and_peace.txt; wget -o log http://www.gutenberg.org/ebooks/2600.txt.utf-8 

    * 5.2.2. Вывод сжатие разных методов;

```
[root@lesson4 fs_1]# zfs get compression,compressratio /myraidz1/fs_*
NAME           PROPERTY       VALUE     SOURCE
myraidz1/fs_1  compression    gzip-9    local
myraidz1/fs_1  compressratio  1.91x     -
myraidz1/fs_2  compression    zle       local
myraidz1/fs_2  compressratio  1.04x     -
myraidz1/fs_3  compression    lz4       local
myraidz1/fs_3  compressratio  1.44x     -
myraidz1/fs_4  compression    lzjb      local
myraidz1/fs_4  compressratio  1.28x     -

```
    * 5.2.3. Вывод сколько места занимают ФС;

```
[root@lesson4 fs_1]# du -sh /myraidz1/* | sort -rh 
4.4M	/myraidz1/fs_2
3.6M	/myraidz1/fs_4
3.2M	/myraidz1/fs_3
2.4M	/myraidz1/fs_1
512	/myraidz1/file1
```

### 2. Определить настройки pool’a:
1. Размер хранлища;

```
[root@lesson4 ~]# zpool list
NAME       SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
myraidz1  3.75G  8.84M  3.74G        -         -     0%     0%  1.00x    ONLINE  -
otus       480M  2.10M   478M        -         -     0%     0%  1.00x    ONLINE  -

```

2. Тип pool mirror-0;

```
[root@lesson4 ~]# zpool status otus
  pool: otus
 state: ONLINE
  scan: none requested
config:

	NAME                    STATE     READ WRITE CKSUM
	otus                    ONLINE       0     0     0
	  mirror-0              ONLINE       0     0     0
	    /zpoolexport/filea  ONLINE       0     0     0
	    /zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors

```

3. Значение recordsize;

```
[root@lesson4 ~]# zfs get recordsize 
NAME            PROPERTY    VALUE    SOURCE
myraidz1        recordsize  128K     default
myraidz1/fs_1   recordsize  128K     default
myraidz1/fs_2   recordsize  128K     default
myraidz1/fs_3   recordsize  128K     default
myraidz1/fs_4   recordsize  128K     default
otus            recordsize  128K     local
otus/hometask2  recordsize  128K     inherited from otus
```

4. Какое сжатие используется - zle ;

```
[root@lesson4 ~]# zfs get  compression,compressratio /otus/*
NAME            PROPERTY       VALUE     SOURCE
otus/hometask2  compression    zle       inherited from otus
otus/hometask2  compressratio  1.00x     -
[root@lesson4 ~]# 

``` 

5. Какая контрольная сумма используется - sha256;

```
[root@lesson4 ~]# zfs get checksum otus otus/hometask2
NAME            PROPERTY  VALUE      SOURCE
otus            checksum  sha256     local
otus/hometask2  checksum  sha256     inherited from otus
```

6. Список команд с помощью, которых восстановили pool, с выводами;

* 6.1. Узнал как называется pool;
``` 
[root@lesson4 /]# zpool import -d  zpoolexport/ 
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

	otus                    ONLINE
	  mirror-0              ONLINE
	    /zpoolexport/filea  ONLINE
	    /zpoolexport/fileb  ONLINE
```

* 6.2. Импорт pool;

```
zpool import -d  zpoolexport/  otus   
     pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

	otus                    ONLINE
	  mirror-0              ONLINE
	    /zpoolexport/filea  ONLINE
	    /zpoolexport/fileb  ONLINE
```

### 3. Найти сообщение от преподавателей:
1. Скачал файл, далее восстановление snapshot;

```
[root@lesson4 ~]# zfs receive otus/storage < otus_task2.file
[root@lesson4 ~]# zfs list
NAME             USED  AVAIL     REFER  MOUNTPOINT
myraidz1        6.50M  2.67G     40.4K  /myraidz1
myraidz1/fs_1   6.03M  2.67G     6.03M  /myraidz1/fs_1
myraidz1/fs_2   32.9K  2.67G     32.9K  /myraidz1/fs_2
myraidz1/fs_3   32.9K  2.67G     32.9K  /myraidz1/fs_3
myraidz1/fs_4   32.9K  2.67G     32.9K  /myraidz1/fs_4
otus            4.93M   347M       25K  /otus
otus/hometask2  1.88M   347M     1.88M  /otus/hometask2
otus/storage    2.83M   347M     2.83M  /otus/storage

```

2. Найденное сообщение;

```
[root@lesson4 task1]# find /otus/storage/task1/ -name secret_message -exec cat {} \;
https://github.com/sindresorhus/awesome
``` 
