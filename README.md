# Lesson-6-NFS
#Домашнее задание
#Работа с NFS

Что нужно сделать?

  Запустить 2 виртуальных машины (сервер NFS и клиента);
  На сервере NFS должна быть подготовлена и экспортирована директория;
  В экспортированной директории должна быть поддиректория с именем upload с правами на запись в неё;
  Экспортированная директория должна автоматически монтироваться на клиенте при старте виртуальной машины (systemd, autofs или fstab — любым способом);
  Монтирование и работа NFS на клиенте должна быть организована с использованием NFSv3.

# Цель  Задания 
  Научиться разворачивать NFS сервис и подключать клиента.
  В данном уроке будет выполнено с помощью автоматизации ansible.

# Старт
  Развернуто 3 ВМ на базе Ubuntu 20.04
  ansible-server, nfs-server и  nfs-client

# 1 Подготовка ansible-server 
 # Установка пакетов  
    sudo apt install -y python3-pip sshpass
    python3 -m pip install --user ansible
#      # Обновите переменные окружения 
        source ~/.profile

# 2 inventory и playbooks
---
# inventory.yaml
# nfs-server.yml
# nfs-client.yml
---

# 3 Установка и настройка сервера
  Запускаем с fysible-server
 user@ubuntu:~$ ansible-playbook nfs-server.yml -i  inventory.yaml 

PLAY [Сбор фактов с client] ***************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [client-nfs]

PLAY [Настройка NFS сервера] **************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [server-nfs]

TASK [Установка пакетов] ******************************************************************************************************************************
changed: [server-nfs]

TASK [Запуск и включение в автозагрузку] **************************************************************************************************************
changed: [server-nfs]

TASK [Создание основной директории] *******************************************************************************************************************
changed: [server-nfs]

TASK [Создание директории upload] *********************************************************************************************************************
changed: [server-nfs]

TASK [Настройка экспорта директории] ******************************************************************************************************************
changed: [server-nfs]

TASK [Экспорт директории] *****************************************************************************************************************************
ok: [server-nfs]

TASK [Получение информации об экспортированных директориях] *******************************************************************************************
ok: [server-nfs]

TASK [Вывод информации об директориях] ****************************************************************************************************************
ok: [server-nfs] => {
    "msg": [
        "/srv/share  192.168.122.4(rw,wdelay,root_squash,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)"
    ]
}

RUNNING HANDLER [restart nfs-server] ******************************************************************************************************************
changed: [server-nfs]

PLAY RECAP ********************************************************************************************************************************************
client-nfs                 : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
server-nfs                 : ok=10   changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0    

# Успех

# 4 Установка и настройка клиента
user@ubuntu:~$ ansible-playbook nfs-client.yml -i  inventory.yaml 

PLAY [Сбор фактов с сервера] **************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [server-nfs]

PLAY [Настройка клиента] ******************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [client-nfs]

TASK [Установка пакетов NFS клиента] ******************************************************************************************************************
changed: [client-nfs]

TASK [Создание точки монтирования] ********************************************************************************************************************
ok: [client-nfs]

TASK [Добавление автомонтирования в /etc/fstab] *******************************************************************************************************
changed: [client-nfs]

TASK [Запуск и включение remote-fs.target] ************************************************************************************************************
ok: [client-nfs]

TASK [Перезагрузка systemd] ***************************************************************************************************************************
ok: [client-nfs]

TASK [Проверка монтирования] **************************************************************************************************************************
ok: [client-nfs]

TASK [Вывод результата монтирования] ******************************************************************************************************************
ok: [client-nfs] => {
    "msg": [
        "nsfs on /run/snapd/ns/lxd.mnt type nsfs (rw)"
    ]
}

RUNNING HANDLER [restart remote-fs] *******************************************************************************************************************
changed: [client-nfs]

PLAY RECAP ********************************************************************************************************************************************
client-nfs                 : ok=9    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
server-nfs                 : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
# Успех

# Проверка работоспособности 
  Заходим на сервер. 
  Заходим в каталог /srv/share/upload.
  Создаём тестовый файл touch check_file.
  Заходим на клиент.
  Заходим в каталог /mnt/upload. 
  Проверяем наличие ранее созданного файла.
  Создаём тестовый файл touch client_file. 
  Проверяем, что файл успешно создан.
    Если вышеуказанные проверки прошли успешно, это значит, что проблем с правами нет. 

# Проврка сервера и создание тестового файла: 
  * заходим на сервер в отдельном окне терминала;
      Last login: Wed Jul 22 12:03:59 2026 from 192.168.122.20
      user@serv:/srv/share/upload$ ls
      user@serv:/srv/share/upload$ 
  * перезагружаем сервер;
      user@serv:/srv/share/upload$ sudo reboot
  * заходим на сервер;
      Last login: Wed Jul 22 12:31:50 2026 from 192.168.122.1
      user@serv:~$ 
 
  * проверяем наличие файлов в каталоге /srv/share/upload/;
      user@serv:~$ ls /srv/share/upload/
      user@serv:~$  # пусто
      
  * проверяем экспорты exportfs -s; 
    # При раскатке сервера он проверяет экспорт в консоль
    ok: [server-nfs] => {
    "msg": [
        "/srv/share  192.168.122.4(rw,wdelay,root_squash,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)"
  * Проверкаа по заданию в ручном режиме
      user@serv:~$ sudo exportfs -s
      [sudo] password for user: 
      /srv/share  192.168.122.4(rw,wdelay,root_squash,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
  
  * проверяем работу RPC showmount -a 192.168.122.23
      user@serv:~$ sudo showmount -a 192.168.122.23
      All mount points on 192.168.122.23:
      192.168.122.4:/srv/share
  * Создадим файлы в каталоге /srv/share/upload/ 
      user@serv:~$ cp /var/log/syslog  /srv/share/upload/
      user@serv:~$ ls /srv/share/upload/
      syslog

# Проверяем клиент: 
 * Заходим на клиент;
    artem@mechrevo:~$ ssh user@192.168.122.4
    Last login: Wed Jul 22 12:30:09 2026 from 192.168.122.1
    user@client:~$   
 * перезагружаем клиент;
    user@client:~$ sudo reboot
    [sudo] password for user: 
    Connection to 192.168.122.4 closed by remote host.
 * заходим на клиент;
    Last login: Wed Jul 22 12:50:42 2026 from 192.168.122.1
    user@client:~$
 * проверяем работу RPC showmount -a  ;
    user@client:~$ sudo showmount -a 192.168.122.23
    All mount points on 192.168.122.23:
    192.168.122.4:/srv/share
 * заходим в каталог /mnt/upload;
    user@client:~$ cd /mnt/upload/
    user@client:/mnt/upload$ 
 * проверяем статус монтирования mount | grep mnt;
    user@client:/mnt/upload$ mount | grep mnt
    systemd-1 on /mnt type autofs (rw,relatime,fd=58,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=20821)
    192.168.122.23:/srv/share on /mnt type nfs (rw,relatime,vers=3,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=192.168.122.23,mountvers=3,mountport=52810,mountproto=udp,local_lock=none,addr=192.168.122.23)
    nsfs on /run/snapd/ns/lxd.mnt type nsfs (rw)
 * проверяем наличие ранее созданных файлов;
    user@client:/mnt/upload$ ls -la 
    total 312
    drwxrwxrwx 2 nobody nogroup   4096 Jul 22 13:02 .
    drwxr-xr-x 3 nobody nogroup   4096 Jul 22 12:02 ..
    -rw-r----- 1 user   user    307609 Jul 22 13:02 syslog
 * создаём тестовый файл touch final_check;
    user@client:/mnt/upload$ touch 16
    user@client:/mnt/upload$ ls -la
    total 312
    drwxrwxrwx 2 nobody nogroup   4096 Jul 22 13:07 .
    drwxr-xr-x 3 nobody nogroup   4096 Jul 22 12:02 ..
    -rw-rw-r-- 1 user   user         0 Jul 22 13:07 16
    -rw-r----- 1 user   user    307609 Jul 22 13:02 syslog
 * проверяем, что файл успешно создан.
   user@serv:~$ ls /srv/share/upload/ -la
   total 312
   drwxrwxrwx 2 nobody nogroup   4096 Jul 22 13:07 .
   drwxr-xr-x 3 nobody nogroup   4096 Jul 22 12:02 ..
   -rw-rw-r-- 1 user   user         0 Jul 22 13:07 16
   -rw-r----- 1 user   user    307609 Jul 22 13:02 syslog

# Если вышеуказанные проверки прошли успешно, это значит, что демонстрационный стенд работоспособен и готов к работе.

# 
