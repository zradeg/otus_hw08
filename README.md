# Управление пакетами. Дистрибьюция софта

## Задачи
1. создать свой RPM
2. создать свой репо и разместить там свой RPM
* реализовать это все либо в вагранте, либо развернуть у себя через nginx и дать ссылку на репо

#### Приложены файлы:
* nginx.spec - готовый spec-файл для nginx
* hw08-otus.repo - готовый конфиг для развернутого репозитория - на время проверки ДЗ будет поднят и доступен.

## 1. Создание своего RPM

Буду собирать nginx сконфигурированный сразу с openssl

Устанавливаем пакеты, необходимые для сборки RPM, а также docker, чтобы развернуть репозиторий из-под него
```
[root@ztest ~]# yum install -y redhat-lsb-core wget rpmdevtools rpm-build createrepo yum-utils gcc docker
```

Скачиваем исходники nginx
```
[root@ztest ~]# wget https://nginx.org/packages/centos/7/SRPMS/nginx-1.14.1-1.el7_4.ngx.src.rpm
--2020-03-17 02:06:29--  https://nginx.org/packages/centos/7/SRPMS/nginx-1.14.1-1.el7_4.ngx.src.rpm
Resolving nginx.org (nginx.org)... 95.211.80.227, 62.210.92.35, 2001:1af8:4060:a004:21::e3 Connecting to nginx.org (nginx.org)|95.211.80.227|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1033399 (1009K) [application/x-redhat-package-manager]
Saving to: ‘nginx-1.14.1-1.el7_4.ngx.src.rpm’

100%[=================================================>] 1,033,399   2.06MB/s   in 0.5s

2020-03-17 02:06:30 (2.06 MB/s) - ‘nginx-1.14.1-1.el7_4.ngx.src.rpm’ saved [1033399/1033399]
```

Устанавливаем скачанный пакет
```
[root@ztest ~]# rpm -i nginx-1.14.1-1.el7_4.ngx.src.rpm
warning: nginx-1.14.1-1.el7_4.ngx.src.rpm: Header V4 RSA/SHA1 Signature, key ID 7bd9bf62: NOKEY
```

Скачиваем и распаковываем исходники openssl
```
[root@ztest ~]# wget https://www.openssl.org/source/latest.tar.gz
--2020-03-17 02:07:00--  https://www.openssl.org/source/latest.tar.gz
Resolving www.openssl.org (www.openssl.org)... 2.19.116.8, 2a02:26f0:18:486::c1e, 2a02:26f0:18:482::c1e
Connecting to www.openssl.org (www.openssl.org)|2.19.116.8|:443... connected.
HTTP request sent, awaiting response... 302 Moved Temporarily
Location: https://www.openssl.org/source/openssl-1.1.1d.tar.gz [following]
--2020-03-17 02:07:03--  https://www.openssl.org/source/openssl-1.1.1d.tar.gz
Reusing existing connection to www.openssl.org:443.
HTTP request sent, awaiting response... 200 OK
Length: 8845861 (8.4M) [application/x-gzip]
Saving to: ‘latest.tar.gz’

100%[=================================================>] 8,845,861   39.2MB/s   in 0.2s

2020-03-17 02:07:03 (39.2 MB/s) - ‘latest.tar.gz’ saved [8845861/8845861]

[root@ztest ~]# tar -xvf latest.tar.gz
```

Устанавливаем зависимости, чтобы избежать ошибок в дальнейшем
```
[root@ztest ~]# yum-builddep rpmbuild/SPECS/nginx.spec
Loaded plugins: fastestmirror
Enabling base-source repository
Enabling extras-source repository
Enabling updates-source repository
Loading mirror speeds from cached hostfile
 * base: centos-mirror.rbc.ru
 * extras: mirror.logol.ru
 * updates: centos-mirror.rbc.ru
base-source                                                         | 2.9 kB  00:00:00
extras-source                                                       | 2.9 kB  00:00:00
updates-source                                                      | 2.9 kB  00:00:00
(1/3): extras-source/7/primary_db                                   |  14 kB  00:00:00
(2/3): updates-source/7/primary_db                                  | 105 kB  00:00:01
(3/3): base-source/7/primary_db                                     | 1.1 MB  00:00:02
Checking for new repos for mirrors
Getting requirements for rpmbuild/SPECS/nginx.spec
 --> Already installed : redhat-lsb-core-4.1-27.el7.centos.1.x86_64
 --> Already installed : systemd-219-67.el7.x86_64
 --> 1:openssl-devel-1.0.2k-19.el7.x86_64
 --> zlib-devel-1.2.7-18.el7.x86_64
 --> pcre-devel-8.32-17.el7.x86_64
--> Running transaction check
---> Package openssl-devel.x86_64 1:1.0.2k-19.el7 will be installed
--> Processing Dependency: krb5-devel(x86-64) for package: 1:openssl-devel-1.0.2k-19.el7.x86_64
---> Package pcre-devel.x86_64 0:8.32-17.el7 will be installed
---> Package zlib-devel.x86_64 0:1.2.7-18.el7 will be installed
--> Running transaction check
---> Package krb5-devel.x86_64 0:1.15.1-37.el7_7.2 will be installed
--> Processing Dependency: libkadm5(x86-64) = 1.15.1-37.el7_7.2 for package: krb5-devel-1.15.1-37.el7_7.2.x86_64
--> Processing Dependency: krb5-libs(x86-64) = 1.15.1-37.el7_7.2 for package: krb5-devel-1.15.1-37.el7_7.2.x86_64
--> Processing Dependency: libverto-devel for package: krb5-devel-1.15.1-37.el7_7.2.x86_64 --> Processing Dependency: libselinux-devel for package: krb5-devel-1.15.1-37.el7_7.2.x86_64
--> Processing Dependency: libcom_err-devel for package: krb5-devel-1.15.1-37.el7_7.2.x86_64
--> Processing Dependency: keyutils-libs-devel for package: krb5-devel-1.15.1-37.el7_7.2.x86_64
--> Running transaction check
---> Package keyutils-libs-devel.x86_64 0:1.5.8-3.el7 will be installed
---> Package krb5-libs.x86_64 0:1.15.1-37.el7_6 will be updated
---> Package krb5-libs.x86_64 0:1.15.1-37.el7_7.2 will be an update
---> Package libcom_err-devel.x86_64 0:1.42.9-16.el7 will be installed
---> Package libkadm5.x86_64 0:1.15.1-37.el7_7.2 will be installed
---> Package libselinux-devel.x86_64 0:2.5-14.1.el7 will be installed
--> Processing Dependency: libsepol-devel(x86-64) >= 2.5-10 for package: libselinux-devel-2.5-14.1.el7.x86_64
--> Processing Dependency: pkgconfig(libsepol) for package: libselinux-devel-2.5-14.1.el7.x86_64
---> Package libverto-devel.x86_64 0:0.2.5-4.el7 will be installed
--> Running transaction check
---> Package libsepol-devel.x86_64 0:2.5-10.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================== Package                    Arch          Version                     Repository      Size ===========================================================================================Installing:
 openssl-devel              x86_64        1:1.0.2k-19.el7             base           1.5 M  pcre-devel                 x86_64        8.32-17.el7                 base           480 k  zlib-devel                 x86_64        1.2.7-18.el7                base            50 k Installing for dependencies:
 keyutils-libs-devel        x86_64        1.5.8-3.el7                 base            37 k  krb5-devel                 x86_64        1.15.1-37.el7_7.2           updates        272 k  libcom_err-devel           x86_64        1.42.9-16.el7               base            32 k  libkadm5                   x86_64        1.15.1-37.el7_7.2           updates        178 k  libselinux-devel           x86_64        2.5-14.1.el7                base           187 k  libsepol-devel             x86_64        2.5-10.el7                  base            77 k  libverto-devel             x86_64        0.2.5-4.el7                 base            12 k Updating for dependencies:
 krb5-libs                  x86_64        1.15.1-37.el7_7.2           updates        805 k

Transaction Summary
===========================================================================================Install  3 Packages (+7 Dependent packages)
Upgrade             ( 1 Dependent package)

Total download size: 3.6 M
Is this ok [y/d/N]: y
Downloading packages:
No Presto metadata available for updates
(1/11): keyutils-libs-devel-1.5.8-3.el7.x86_64.rpm                  |  37 kB  00:00:00
(2/11): krb5-devel-1.15.1-37.el7_7.2.x86_64.rpm                     | 272 kB  00:00:00
(3/11): libcom_err-devel-1.42.9-16.el7.x86_64.rpm                   |  32 kB  00:00:00
(4/11): libselinux-devel-2.5-14.1.el7.x86_64.rpm                    | 187 kB  00:00:00
(5/11): libkadm5-1.15.1-37.el7_7.2.x86_64.rpm                       | 178 kB  00:00:00
(6/11): libverto-devel-0.2.5-4.el7.x86_64.rpm                       |  12 kB  00:00:00
(7/11): krb5-libs-1.15.1-37.el7_7.2.x86_64.rpm                      | 805 kB  00:00:00
(8/11): openssl-devel-1.0.2k-19.el7.x86_64.rpm                      | 1.5 MB  00:00:00
(9/11): pcre-devel-8.32-17.el7.x86_64.rpm                           | 480 kB  00:00:00
(10/11): libsepol-devel-2.5-10.el7.x86_64.rpm                       |  77 kB  00:00:00
(11/11): zlib-devel-1.2.7-18.el7.x86_64.rpm                         |  50 kB  00:00:00
-------------------------------------------------------------------------------------------Total                                                      7.1 MB/s | 3.6 MB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : krb5-libs-1.15.1-37.el7_7.2.x86_64                                     1/12
  Installing : libkadm5-1.15.1-37.el7_7.2.x86_64                                      2/12
  Installing : keyutils-libs-devel-1.5.8-3.el7.x86_64                                 3/12
  Installing : libsepol-devel-2.5-10.el7.x86_64                                       4/12
  Installing : pcre-devel-8.32-17.el7.x86_64                                          5/12
  Installing : libselinux-devel-2.5-14.1.el7.x86_64                                   6/12
  Installing : libcom_err-devel-1.42.9-16.el7.x86_64                                  7/12
  Installing : zlib-devel-1.2.7-18.el7.x86_64                                         8/12
  Installing : libverto-devel-0.2.5-4.el7.x86_64                                      9/12
  Installing : krb5-devel-1.15.1-37.el7_7.2.x86_64                                   10/12
  Installing : 1:openssl-devel-1.0.2k-19.el7.x86_64                                  11/12
  Cleanup    : krb5-libs-1.15.1-37.el7_6.x86_64                                      12/12
  Verifying  : krb5-libs-1.15.1-37.el7_7.2.x86_64                                     1/12
  Verifying  : libverto-devel-0.2.5-4.el7.x86_64                                      2/12
  Verifying  : krb5-devel-1.15.1-37.el7_7.2.x86_64                                    3/12
  Verifying  : zlib-devel-1.2.7-18.el7.x86_64                                         4/12
  Verifying  : libcom_err-devel-1.42.9-16.el7.x86_64                                  5/12
  Verifying  : libkadm5-1.15.1-37.el7_7.2.x86_64                                      6/12
  Verifying  : pcre-devel-8.32-17.el7.x86_64                                          7/12
  Verifying  : 1:openssl-devel-1.0.2k-19.el7.x86_64                                   8/12
  Verifying  : libselinux-devel-2.5-14.1.el7.x86_64                                   9/12
  Verifying  : libsepol-devel-2.5-10.el7.x86_64                                      10/12
  Verifying  : keyutils-libs-devel-1.5.8-3.el7.x86_64                                11/12
  Verifying  : krb5-libs-1.15.1-37.el7_6.x86_64                                      12/12

Installed:
  openssl-devel.x86_64 1:1.0.2k-19.el7           pcre-devel.x86_64 0:8.32-17.el7
  zlib-devel.x86_64 0:1.2.7-18.el7

Dependency Installed:
  keyutils-libs-devel.x86_64 0:1.5.8-3.el7      krb5-devel.x86_64 0:1.15.1-37.el7_7.2
  libcom_err-devel.x86_64 0:1.42.9-16.el7       libkadm5.x86_64 0:1.15.1-37.el7_7.2
  libselinux-devel.x86_64 0:2.5-14.1.el7        libsepol-devel.x86_64 0:2.5-10.el7
  libverto-devel.x86_64 0:0.2.5-4.el7

Dependency Updated:
  krb5-libs.x86_64 0:1.15.1-37.el7_7.2

Complete!
```

Формируем spec-файл для nginx (готовый файл приложен с заданием)
```
[root@ztest ~]# cat >rpmbuild/SPECS/nginx.spec
```

Выполняем сборку пакета
```
[root@ztest ~]# rpmbuild -bb rpmbuild/SPECS/nginx.spec
```

Перемещаем пакет в директорию, где будет располагаться репозиторий
```
[root@ztest ~]# mv rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm /srv/nginx/
```

## 2. Развертывание репозитория
Поднимаю контейнер.
```
[root@ztest ~]# docker run --name nginx -v /srv/nginx/:/usr/share/nginx/html -p 80:80 -d nginx
```
Здесь:
* --name - имя, под которым будет запущен контейнер
* /srv/nginx/:/usr/share/nginx/html - проброс хостовой директории внутрь контейнера
* -p 80:80 - проброс порта (соответственно, 80 хостовый на 80 контейнера)
* -d nginx - образ, который неоходимо зайдествовать. При осутствии такового в локальном репозитории будет скачан из облака.

Создаю мета-окружение для репозитория
```
[root@ztest ~]# createrepo /srv/nginx
```

Проверяю
```
[root@ztest ~]# yum repolist enabled
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos-mirror.rbc.ru
 * extras: mirror.logol.ru
 * updates: centos-mirror.rbc.ru
repo id                                  repo name                                   status
base/7/x86_64                            CentOS-7 - Base                             10,097
extras/7/x86_64                          CentOS-7 - Extras                              335
local                                    HW08-Otus-Base                                   0
updates/7/x86_64                         CentOS-7 - Updates                           1,487
repolist: 11,919
[root@ztest ~]#
```
Вижу, что пакеты отсутствуют.

Проверяю, задействована ли в настройках nginx опция autoindex
```
[root@ztest ~]# docker exec -it nginx bash

root@fae0f61535df:/# cat -n /etc/nginx/conf.d/default.conf
     1  server {
     2      listen       80;
     3      server_name  localhost;
     4
     5      #charset koi8-r;
     6      #access_log  /var/log/nginx/host.access.log  main;
     7
     8      location / {
     9          root   /usr/share/nginx/html;
    10          index  index.html index.htm;
    11      }
    12
```
Опция действительно отсутствует.

Поэтому добавляю ее и перегружаю конфигурацию nginx
```
root@fae0f61535df:/# sed -i "11s/\}/    autoindex on;\n    \}/" /etc/nginx/conf.d/default.conf

root@fae0f61535df:/# cat -n /etc/nginx/conf.d/default.conf | head -n12
     1  server {
     2      listen       80;
     3      server_name  localhost;
     4
     5      #charset koi8-r;
     6      #access_log  /var/log/nginx/host.access.log  main;
     7
     8      location / {
     9          root   /usr/share/nginx/html;
    10          index  index.html index.htm;
    11          autoindex on;
    12      }
	
root@fae0f61535df:/# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
root@fae0f61535df:/# nginx -s reload
2020/03/17 14:47:08 [notice] 52#52: signal process started
```

Снова проверяю
```
[root@ztest ~]# yum repolist enabled
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos-mirror.rbc.ru
 * extras: mirror.logol.ru
 * updates: centos-mirror.rbc.ru
repo id                                  repo name                                   status
base/7/x86_64                            CentOS-7 - Base                             10,097
extras/7/x86_64                          CentOS-7 - Extras                              335
local                                    HW08-Otus-Base                                   0
updates/7/x86_64                         CentOS-7 - Updates                           1,487
repolist: 11,919
[root@ztest ~]#
```
Пакетов все еще нет. Но при этом браузер показывает, что они есть.

Пересоздаю кеш пакетного менеджера
```
[root@ztest ~]# yum makecache
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos-mirror.rbc.ru
 * extras: mirror.logol.ru
 * updates: centos-mirror.rbc.ru
base                                                                | 3.6 kB  00:00:00
extras                                                              | 2.9 kB  00:00:00
local                                                               | 2.9 kB  00:00:00
updates                                                             | 2.9 kB  00:00:00
(1/9): extras/7/x86_64/filelists_db                                 | 216 kB  00:00:00
(2/9): extras/7/x86_64/other_db                                     | 106 kB  00:00:00
(3/9): local/filelists_db                                           | 1.5 kB  00:00:00
(4/9): local/primary_db                                             | 2.7 kB  00:00:00
(5/9): base/7/x86_64/other_db                                       | 2.6 MB  00:00:00
(6/9): local/other_db                                               |  958 B  00:00:00
(7/9): base/7/x86_64/filelists_db                                   | 7.3 MB  00:00:00
(8/9): updates/7/x86_64/other_db                                    | 493 kB  00:00:00
(9/9): updates/7/x86_64/filelists_db                                | 4.0 MB  00:00:00
Metadata Cache Created
```

Снова проверяю:
```
[root@ztest ~]# yum repolist enabled
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos-mirror.rbc.ru
 * extras: mirror.logol.ru
 * updates: centos-mirror.rbc.ru
repo id                                  repo name                                   status
base/7/x86_64                            CentOS-7 - Base                             10,097
extras/7/x86_64                          CentOS-7 - Extras                              335
local                                    HW08-Otus-Base                                   1
updates/7/x86_64                         CentOS-7 - Updates                           1,487
repolist: 11,920
```
Бинго!