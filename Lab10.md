# Tools docker. Security. Практика.
## Hadolint
Hadolint представляет собой утилиту (линтер), предназначенную для оценки Dockerfile с точки зрения корректности синтаксиса и безопасности инструкций. Также Hadolint проводит проверку на соблюдение лучших практик по написанию инструкций в Dockerfile.

Скачаем image
```sh
/home/noneraspad# docker pull hadolint/hadolint
Using default tag: latest
latest: Pulling from hadolint/hadolint
Digest: sha256:fff226bdf9ebcc08db47fb90ee144dd770120b35c2b1cbbb46e932a650cfe232
Status: Image is up to date for hadolint/hadolint:latest
docker.io/hadolint/hadolint:latest
```
В качестве примера возьмем Dockerfile со следующим содержанием:
```sh
FROM debian
RUN export node_version="0.10" \
&& apt-get update && apt-get -y install nodejs="$node_verion"
COPY package.json usr/src/app
RUN cd /usr/src/app \
&& npm install node-static
EXPOSE 80000
CMD ["npm", "start"]
```
Hadolint можно запустить прямо в контейнере Docker для этого достаточно выполнить команду ниже, передав имя Dockerfile в качестве параметра:
```sh
docker run --rm -i hadolint/hadolint < Dockerfile
```
После того, как анализ будет завершен, в терминале будут выведены все недочеты, которые были найдены в Dockerfile.
```sh
-:1 DL3006 warning: Always tag the version of an image explicitly
-:2 DL3015 info: Avoid additional packages by specifying `--no-install-recommends`
-:2 DL3009 info: Delete the apt-get lists after installing something
-:2 SC2154 warning: node_version is referenced but not assigned (did you mean 'node_version'?).
-:4 DL3045 warning: `COPY` to a relative destination without `WORKDIR` set.
-:5 DL3003 warning: Use WORKDIR to switch to a directory
-:5 DL3016 warning: Pin versions in npm. Instead of `npm install <package>` use `npm install <package>@<version>`
-:8 DL3011 error: Valid UNIX ports range from 0 to 65535
```
В качестве уровней предупреждения используются следующие:

info — рекомендации не являются обязательными к исправлению, но помогут оптимизировать образ;

warning — необходимо обратить внимание на недочеты, как правило, предоставляются советы из официальной документации по оптимизации Dockerfile;

error — найдены критические ошибки, которые влияют на сборку образа и могут привести к проблем после запуска контейнера.
## Dive
Dive — утилита, которая визуально отображает подробную информацию о Docker образах и их слоях. 

Установим Dive
```sh
/home/noneraspad# sudo apt install ./dive_0.9.2_linux_amd64.deb
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Note, selecting 'dive' instead of './dive_0.9.2_linux_amd64.deb'
The following packages were automatically installed and are no longer required:
  linux-image-6.2.0-26-generic linux-modules-6.2.0-26-generic
  linux-modules-extra-6.2.0-26-generic
Use 'sudo apt autoremove' to remove them.
The following NEW packages will be installed:
  dive
0 upgraded, 1 newly installed, 0 to remove and 6 not upgraded.
Need to get 0 B/4 765 kB of archives.
After this operation, 12,4 MB of additional disk space will be used.
Get:1 /home/noneraspad/dive_0.9.2_linux_amd64.deb dive amd64 0.9.2 [4 765 kB]
Selecting previously unselected package dive.
(Reading database ... 243073 files and directories currently installed.)
Preparing to unpack .../dive_0.9.2_linux_amd64.deb ...
Unpacking dive (0.9.2) ...
Setting up dive (0.9.2) ...
N: Download is performed unsandboxed as root as file '/home/noneraspad/dive_0.9.2_linux_amd64.deb' couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)
root@Ubunty-22:/home/noneraspad#
```
Dive можно запустить в контейнере, где последним аргументом указывается название образа или контейнера, который необходимо проанализировать:
```sh
docker run --rm -it \
    -v /var/run/docker.sock:/var/run/docker.sock \
    wagoodman/dive:latest portainer/portainer-ce
```
В результате полечим:
```sh
Cmp   Size  Command                               ├── bin
     69 MB  FROM a0441b36624a46d                  │   ├── bash
    7.0 MB  set -eux;  apt-get update;  apt-get i │   ├── cat           
     29 MB  set -ex   && savedAptMark="$(apt-mark │   ├── chgrp   
       0 B  cd /usr/local/bin  && ln -s idle3 idl │   ├── chmod        
    9.5 MB  set -ex;   savedAptMark="$(apt-mark s │   ├── chown       
     21 MB  apt update -y && apt upgrade -y       │   ├── cp        
       0 B  #(nop) WORKDIR /app                   │   ├── dash           
     780 B  #(nop) COPY file:dee2049857c0d2462dfd │   ├── date     
     15 MB  /usr/local/bin/python -m pip install  │   ├── dd                           
    129 MB  pip3 install -r requirements.txt      │   ├── df                       
     63 MB  #(nop) COPY dir:20622e83d6174117e1202 │   ├── dir                              
                                                  │   ├── dmesg                     
│ Layer Details ├──────────────────────────────── │   ├── dnsdomainname → hostname     
                                                  │   ├── domainname → hostname    
Tags:   (unavailable)                             │   ├── echo                           
Id:     4cd949ff443ee86d59b852cd897a3706fee36c0f7 │   ├── egrep                  
dcf81afb6f867b33d506679                           │   ├── false                    
Digest: sha256:ddc260ab913e1beb894c1ab211bec2b938 │   ├── fgrep                     
9dbb2d3536b96d43a46cf059c13b8e                    │   ├── findmnt   
```
На панели справа отображаются слои образа, а слева — подробная информация о каждом слое. Dive позволяет проанализировать размер образа и выявить те слои, которые занимают больше всех места на диске. 
## Dockle
Dockle — инструмент для проверки безопасности образов контейнеров, который можно использовать для поиска уязвимостей. Кроме того, с его помощью можно выполнять проверку на соответствие Best Practice, чтобы убедиться, что образ действительно создаётся на основе сохраненной истории команд.

Установим Dockle
```sh
/home/noneraspad/Lab10# ./dockle incedos_fl:latest
```
Запускаем утилиту, указав имя образа. Если проблем нет, в консоль будет выведено сообщение PASS, а при обнаружении проблем или уязвимостей — подробная информация о них:
```sh
FATAL   - CIS-DI-0010: Do not store credential in environment variables/files        * Suspicious filename found : usr/local/lib/python3.10/site-packages/pymongo/settings.py (You can suppress it with "-af settings.py")
FATAL   - DKL-DI-0005: Clear apt-get caches
        * Use 'rm -rf /var/lib/apt/lists' after 'apt-get install|update' : /bin/sh -c apt update -y && apt upgrade -y
WARN    - CIS-DI-0001: Create a user for the container
        * Last user should not be root
WARN    - DKL-DI-0006: Avoid latest tag
        * Avoid 'latest' tag
INFO    - CIS-DI-0005: Enable Content trust for Docker
        * export DOCKER_CONTENT_TRUST=1 before docker pull/build
INFO    - CIS-DI-0006: Add HEALTHCHECK instruction to the container image
        * not found HEALTHCHECK statement
INFO    - CIS-DI-0008: Confirm safety of setuid/setgid files
        * setgid file: grwxr-xr-x usr/bin/wall
        * setuid file: urwxr-xr-x usr/bin/passwd
        * setgid file: grwxr-xr-x usr/bin/expiry
        * setgid file: grwxr-xr-x usr/bin/chage
        * setuid file: urwxr-xr-x bin/su
        * setgid file: grwxr-xr-x sbin/unix_chkpwd
        * setuid file: urwxr-xr-x usr/bin/chsh
        * setuid file: urwxr-xr-x usr/bin/chfn
        * setuid file: urwxr-xr-x usr/bin/newgrp
        * setuid file: urwxr-xr-x bin/umount
        * setuid file: urwxr-xr-x bin/mount
        * setuid file: urwxr-xr-x usr/bin/gpasswd
INFO    - DKL-LI-0003: Only put necessary files
        * Suspicious directory : tmp 
        * unnecessary file : app/mongotest/docker-compose.yml 
        * unnecessary file : app/Dockerfile 
        * unnecessary file : app/docker-compose.yml
```
## Anchore
Anchore Engine — централизованная служба проверки, анализа и сертификации образов Docker. 

Она сканирует образы, используя данные об уязвимостях (feeds) от вендоров ОС, таких как Red Hat или Debian.

Пулим Anchore
```sh
/home/noneraspad# docker pull anchore/grype
Using default tag: latest
latest: Pulling from anchore/grype
8969c7df4bbe: Pull complete 
ec0dbbd1a377: Pull complete 
7f7a59c7e6ee: Pull complete 
Digest: sha256:12b3b56d62116200795d43e568162257e9518c479e5348cc5ac4bdd4ca0bf4e8
Status: Downloaded newer image for anchore/grype:latest
docker.io/anchore/grype:latest
root@Ubunty-22:/home/noneraspad# 
```
Запустим Anchore
```sh
docker run -it --rm anchore/grype mongo
```
В результате получим: 
```sh
✔ Vulnerability DB                [updated]  
 ✔ Parsed image                    sha256:5acb2131d51f8ae39cbf4360b2f3f3e6d78d  
 ✔ Cataloged packages              [308 packages]  
 ✔ Scanned for vulnerabilities     [95 vulnerability matches]  
   ├── by severity: 6 critical, 34 high, 25 medium, 27 low, 3 negligible
   └── by status:   12 fixed, 83 not-fixed, 0 ignored 
[0069]  WARN some package(s) are missing CPEs. This may result in missing vulner
[0084]  WARN could not match by package language (package=Pkg(type=go-module, na
[0084]  WARN could not match by package language (package=Pkg(type=go-module, na
[0084]  WARN could not match by package language (package=Pkg(type=go-module, na
[0084]  WARN could not match by package language (package=Pkg(type=go-module, na
[0084]  WARN could not match by package language (package=Pkg(type=go-module, na
[0084]  WARN could not match by package language (package=Pkg(type=go-module, na
[0084]  WARN could not match by package language (package=Pkg(type=go-module, na
[0084]  WARN could not match by package language (package=Pkg(type=go-module, na
NAME                            INSTALLED                     FIXED-IN                      TYPE       VULNERABILITY        SEVERITY   
bash                            5.1-6ubuntu1                                                deb        CVE-2022-3715        Low         
coreutils                       8.32-4.1ubuntu1                                             deb        CVE-2016-2781        Low         
dirmngr                         2.2.27-3ubuntu2.1                                           deb        CVE-2022-3219        Low         
gcc-12-base                     12.3.0-1ubuntu1~22.04                                       deb        CVE-2022-27943       Low         
github.com/opencontainers/runc  v1.1.0                        1.1.5                         go-module  GHSA-vpvm-3wq2-2wvm  High        
github.com/opencontainers/runc  v1.1.0                        1.1.5                         go-module  GHSA-g2j6-57v7-gm8c  Medium      
github.com/opencontainers/runc  v1.1.0                        1.1.2                         go-module  GHSA-f3fp-gc8g-vw66  Medium      
github.com/opencontainers/runc  v1.1.0                        1.1.5                         go-module  GHSA-m8cg-xc2p-r3fc  Low         
gnupg                           2.2.27-3ubuntu2.1                                           deb        CVE-2022-3219        Low         
gnupg-l10n                      2.2.27-3ubuntu2.1                                           deb        CVE-2022-3219        Low         
gnupg-utils                     2.2.27-3ubuntu2.1                                           deb        CVE-2022-3219        Low         
gpg                             2.2.27-3ubuntu2.1                                           deb        CVE-2022-3219        Low         
gpg-agent                       2.2.27-3ubuntu2.1                                           deb        CVE-2022-3219        Low         
gpg-wks-client                  2.2.27-3ubuntu2.1                                           deb        CVE-2022-3219        Low         
gpg-wks-server                  2.2.27-3ubuntu2.1                                           deb        CVE-2022-3219        Low         
gpgconf                         2.2.27-3ubuntu2.1                                           deb        CVE-2022-3219        Low         
gpgsm                           2.2.27-3ubuntu2.1                                           deb        CVE-2022-3219        Low         
gpgv                            2.2.27-3ubuntu2.1                                           deb        CVE-2022-3219        Low         
libc-bin                        2.35-0ubuntu3.4               2.35-0ubuntu3.5               deb        CVE-2023-5156        Medium      
libc-bin                        2.35-0ubuntu3.4               2.35-0ubuntu3.5               deb        CVE-2023-4813        Low         
libc-bin                        2.35-0ubuntu3.4               2.35-0ubuntu3.5               deb        CVE-2023-4806        Low         
libc-bin                        2.35-0ubuntu3.4                                             deb        CVE-2016-20013       Negligible  
libc6                           2.35-0ubuntu3.4               2.35-0ubuntu3.5               deb        CVE-2023-5156        Medium      
libc6                           2.35-0ubuntu3.4               2.35-0ubuntu3.5               deb        CVE-2023-4813        Low         
libc6                           2.35-0ubuntu3.4               2.35-0ubuntu3.5               deb        CVE-2023-4806        Low         
libc6                           2.35-0ubuntu3.4                                             deb        CVE-2016-20013       Negligible  
libcurl4                        7.81.0-1ubuntu1.14            7.81.0-1ubuntu1.15            deb        CVE-2023-46218       Medium      
libgcc-s1                       12.3.0-1ubuntu1~22.04                                       deb        CVE-2022-27943       Low         
libldap-2.5-0                   2.5.16+dfsg-0ubuntu0.22.04.1                                deb        CVE-2023-2953        Low         
liblzma5                        5.2.5-2ubuntu1                                              deb        CVE-2020-22916       Medium      
libpcre3                        2:8.39-13ubuntu0.22.04.1                                    deb        CVE-2017-11164       Negligible  
libsqlite3-0                    3.37.2-2ubuntu0.1                                           deb        CVE-2022-46908       Low         
libstdc++6                      12.3.0-1ubuntu1~22.04                                       deb        CVE-2022-27943       Low         
libzstd1                        1.4.8+dfsg-3build1                                          deb        CVE-2022-4899        Low         
login                           1:4.8.1-2ubuntu2.1                                          deb        CVE-2023-29383       Low         
passwd                          1:4.8.1-2ubuntu2.1                                          deb        CVE-2023-29383       Low         
stdlib                          go1.18.2                                                    go-module  CVE-2023-39323       Critical    
stdlib                          go1.18.2                                                    go-module  CVE-2023-29405       Critical    
stdlib                          go1.18.2                                                    go-module  CVE-2023-29404       Critical    
stdlib                          go1.18.2                                                    go-module  CVE-2023-29402       Critical    
stdlib                          go1.18.2                                                    go-module  CVE-2023-24540       Critical    
stdlib                          go1.18.2                                                    go-module  CVE-2023-24538       Critical    
stdlib                          go1.18.2                                                    go-module  CVE-2023-45287       High        
stdlib                          go1.18.2                                                    go-module  CVE-2023-45285       High        
stdlib                          go1.18.2                                                    go-module  CVE-2023-44487       High        
stdlib                          go1.18.2                                                    go-module  CVE-2023-29403       High        
stdlib                          go1.18.2                                                    go-module  CVE-2023-29400       High        
stdlib                          go1.18.2                                                    go-module  CVE-2023-24539       High        
stdlib                          go1.18.2                                                    go-module  CVE-2023-24537       High        
stdlib                          go1.18.2                                                    go-module  CVE-2023-24536       High        
stdlib                          go1.18.2                                                    go-module  CVE-2023-24534       High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-41725       High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-41724       High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-41723       High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-41722       High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-41715       High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-32189       High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-30635       High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-30633       High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-30632       High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-30631       High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-30630       High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-30580       High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-2880        High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-2879        High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-28131       High        
stdlib                          go1.18.2                                                    go-module  CVE-2022-27664       High        
stdlib                          go1.18.2                                                    go-module  CVE-2023-39326       Medium      
stdlib                          go1.18.2                                                    go-module  CVE-2023-39319       Medium      
stdlib                          go1.18.2                                                    go-module  CVE-2023-39318       Medium      
stdlib                          go1.18.2                                                    go-module  CVE-2023-29409       Medium      
stdlib                          go1.18.2                                                    go-module  CVE-2023-29406       Medium      
stdlib                          go1.18.2                                                    go-module  CVE-2023-24532       Medium      
stdlib                          go1.18.2                                                    go-module  CVE-2022-41717       Medium      
stdlib                          go1.18.2                                                    go-module  CVE-2022-32148       Medium      
stdlib                          go1.18.2                                                    go-module  CVE-2022-1962        Medium      
stdlib                          go1.18.2                                                    go-module  CVE-2022-1705        Medium      
stdlib                          go1.18.2                                                    go-module  CVE-2022-30629       Low         
stdlib                          go1.20.11                                                   go-module  CVE-2023-45285       High        
stdlib                          go1.20.11                                                   go-module  CVE-2023-39326       Medium      
tar                             1.34+dfsg-1ubuntu0.1.22.04.1  1.34+dfsg-1ubuntu0.1.22.04.2  deb        CVE-2023-39804       Medium
```
## Docker Bench for Security
Docker Bench for Security — это скрипт, который автоматически проверяет настройки Docker по рекомендациям CIS и генерирует отчёт о проблемах, требующих внимания

Запуск утилиты
```sh
docker run --rm --net host --pid host --userns host --cap-add audit_control \
    -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
    -v /etc:/etc:ro \
    -v /lib/systemd/system:/lib/systemd/system:ro \
    -v /usr/bin/containerd:/usr/bin/containerd:ro \
    -v /usr/bin/runc:/usr/bin/runc:ro \
    -v /usr/lib/systemd:/usr/lib/systemd:ro \
    -v /var/lib:/var/lib:ro \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    --label docker_bench_security \
    docker/docker-bench-security
```
В результате получим:
```sh
# ------------------------------------------------------------------------------
# Docker Bench for Security v1.3.4
#
# Docker, Inc. (c) 2015-
#
# Checks for dozens of common best-practices around deploying Docker containers in production.
# Inspired by the CIS Docker Community Edition Benchmark v1.1.0.
# ------------------------------------------------------------------------------

Initializing Thu Dec  9 07:07:03 UTC 2021


[INFO] 1 - Host Configuration
[WARN] 1.1  - Ensure a separate partition for containers has been created
[NOTE] 1.2  - Ensure the container host has been Hardened
[INFO] 1.3  - Ensure Docker is up to date
```

### Практические задания для текущего занятия

1. Воспольщуйтесь линтером HADOLINT для анализа Dockerfile из предыдущих работ.
2. Воспользуйтесь приложением Dive для анлиза слоев вашего образа.
3. Представьте отчет о безопасности вашего приложения из Лабораторной 6.
4. Воспользуйтесь образом anchore/grype для анализа уязвимостей из приложения, используемого в задании 3.
   






