## Лабораторная работа VIII by Polyakov Andrey

## Ход выполнения
```console
┌──(kali㉿kali)-[~]
└─$ cd /home/kali/GOSICK070404/workspace/projects/
```

```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects]
└─$ git clone https://github.com/GOSICK070404/lab06 lab08

Клонирование в «lab08»…
remote: Enumerating objects: 162, done.
remote: Counting objects: 100% (162/162), done.
remote: Compressing objects: 100% (94/94), done.
remote: Total 162 (delta 75), reused 128 (delta 58), pack-reused 0
Получение объектов: 100% (162/162), 1.04 МиБ | 3.86 МиБ/с, готово.
Определение изменений: 100% (75/75), готово.
```
Переключился на репозиторий для ЛР №8
```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects]
└─$ cd lab08    
```

```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ git remote remove origin
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ git remote add origin https://github.com/GOSICK070404/lab08
```

```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ mkdir demo
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ cat >demo/main.cpp<<EOF
>EOF
```

main.cpp:

```console
#include <print.hpp>

#include <cstdlib>

int main(int argc, char* argv[])
{
  const char* log_path = std::getenv("LOG_PATH");
  if (log_path == nullptr)
  {
    std::cerr << "undefined environment variable: LOG_PATH" << std::endl;
    return 1;
  }
  std::string text;
  while (std::cin >> text)
  {
    std::ofstream out{log_path, std::ios_base::app};
    print(text, out);
    out << std::endl;
  }
}
```

Изменил файл "CMakeLists.txt":

```console
cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_EXAMPLES "Build examples" OFF)
option(BUILD_TESTS "Build tests" OFF)

project(print)
set(PRINT_VERSION_MAJOR 0)
set(PRINT_VERSION_MINOR 1)
set(PRINT_VERSION_PATCH 0)
set(PRINT_VERSION_TWEAK 0)
set(PRINT_VERSION
  ${PRINT_VERSION_MAJOR}.${PRINT_VERSION_MINOR}.${PRINT_VERSION_PATCH}.${PRINT_VERSION_TWEAK})
set(PRINT_VERSION_STRING "v${PRINT_VERSION}")

add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp)

target_include_directories(print PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

if(BUILD_EXAMPLES)
  file(GLOB EXAMPLE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/examples/*.cpp")
  foreach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
    get_filename_component(EXAMPLE_NAME ${EXAMPLE_SOURCE} NAME_WE)
    add_executable(${EXAMPLE_NAME} ${EXAMPLE_SOURCE})
    target_link_libraries(${EXAMPLE_NAME} print)
    install(TARGETS ${EXAMPLE_NAME}
      RUNTIME DESTINATION bin
    )
  endforeach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
endif()

install(TARGETS print
    EXPORT print-config
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)
install(EXPORT print-config DESTINATION cmake)

if(BUILD_TESTS)
    enable_testing()
    file(GLOB ${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
    add_executable(check ${${PROJECT_NAME}_TEST_SOURCES})
    target_link_libraries(check ${PROJECT_NAME} GTest::main)
    add_test(NAME check COMMAND check)
endif()

add_executable(demo demo/main.cpp)
target_link_libraries(demo print)
install(TARGETS demo RUNTIME DESTINATION bin)

include(CPackConfig.cmake)
```

```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ cmake -B build
-- Configuring done
-- Generating done
-- Build files have been written to: /home/kali/GOSICK070404/workspace/projects/lab08/build
```

```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ cmake --build build
[ 50%] Built target print
[100%] Built target demo
```

```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ cat >Dockerfile<<EOF
> EOF
```

Файл "Dockerfile":

```console
FROM ubuntu:18.04

RUN apt update
RUN apt install -yy gcc g++ cmake

COPY . print/
WORKDIR print

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install

ENV LOG_PATH /home/logs/log.txt

VOLUME /home/logs

#WORKDIR _install/bin
WORKDIR /print/_install/bin

ENTRYPOINT ./demo
```
___

```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ docker build -t logger .        
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/build?buildargs=%7B%7D&cachefrom=%5B%5D&cgroupparent=&cpuperiod=0&cpuquota=0&cpusetcpus=&cpusetmems=&cpushares=0&dockerfile=Dockerfile&labels=%7B%7D&memory=0&memswap=0&networkmode=default&rm=1&shmsize=0&t=logger&target=&ulimits=null&version=1": dial unix /var/run/docker.sock: connect: permission denied

```
```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ sudo service docker status
[sudo] password for kali: 
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Mon 2023-06-05 01:40:41 EDT; 5min ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 707 (dockerd)
      Tasks: 8
     Memory: 111.9M
        CPU: 3.355s
     CGroup: /system.slice/docker.service
             └─707 /usr/sbin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Jun 05 01:40:38 kali dockerd[707]: time="2023-06-05T01:40:38.229290921-04:00" level=info msg="[core] Subchannel Connectivity change to READY" module=grpc
Jun 05 01:40:38 kali dockerd[707]: time="2023-06-05T01:40:38.230023470-04:00" level=info msg="[core] Channel Connectivity change to READY" module=grpc
Jun 05 01:40:38 kali dockerd[707]: time="2023-06-05T01:40:38.424722192-04:00" level=info msg="[graphdriver] using prior storage driver: overlay2"
Jun 05 01:40:38 kali dockerd[707]: time="2023-06-05T01:40:38.477680173-04:00" level=info msg="Loading containers: start."
Jun 05 01:40:40 kali dockerd[707]: time="2023-06-05T01:40:40.309528705-04:00" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a preferred IP address"
Jun 05 01:40:40 kali dockerd[707]: time="2023-06-05T01:40:40.793674955-04:00" level=info msg="Loading containers: done."
Jun 05 01:40:41 kali dockerd[707]: time="2023-06-05T01:40:41.278601924-04:00" level=info msg="Docker daemon" commit=5d6db84 graphdriver(s)=overlay2 version=20.10.24+dfsg1
Jun 05 01:40:41 kali dockerd[707]: time="2023-06-05T01:40:41.280755630-04:00" level=info msg="Daemon has completed initialization"
Jun 05 01:40:41 kali systemd[1]: Started docker.service - Docker Application Container Engine.
Jun 05 01:40:41 kali dockerd[707]: time="2023-06-05T01:40:41.399020464-04:00" level=info msg="API listen on /run/docker.sock"

```
```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ sudo ls -la /var/run/docker.sock
srw-rw---- 1 root docker 0 Jun  5 01:40 /var/run/docker.sock
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ users
kali
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ sudo chown kali:docker /var/run/docker.sock                                                                                                                                                                                                                                        
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ sudo ls -la /var/run/docker.sock                   
srw-rw---- 1 kali docker 0 Jun  5 01:40 /var/run/docker.sock

```
```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ docker build -t logger .
```

<details><summary>ResultOfCommand</summary>
  <p>
  
```sh
Sending build context to Docker daemon   2.25MB
Step 1/12 : FROM ubuntu:18.04
18.04: Pulling from library/ubuntu
7c457f213c76: Pull complete 
Digest: sha256:152dc042452c496007f07ca9127571cb9c29697f42acbfad72324b2bb2e43c98
Status: Downloaded newer image for ubuntu:18.04
 ---> f9a80a55f492
Step 2/12 : RUN apt update
 ---> Running in 6c5f02ae0949

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.                                                                                                                                                            
                                                                                                                                                                                                                                           
Get:1 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]                                                                                                                                                                
Get:2 http://archive.ubuntu.com/ubuntu bionic InRelease [242 kB]
Get:3 http://archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]
Get:4 http://archive.ubuntu.com/ubuntu bionic-backports InRelease [83.3 kB]
Get:5 http://security.ubuntu.com/ubuntu bionic-security/universe amd64 Packages [1631 kB]
Get:6 http://archive.ubuntu.com/ubuntu bionic/multiverse amd64 Packages [186 kB]
Get:7 http://archive.ubuntu.com/ubuntu bionic/main amd64 Packages [1344 kB]
Get:8 http://security.ubuntu.com/ubuntu bionic-security/multiverse amd64 Packages [23.8 kB]
Get:9 http://security.ubuntu.com/ubuntu bionic-security/restricted amd64 Packages [1668 kB]
Get:10 http://security.ubuntu.com/ubuntu bionic-security/main amd64 Packages [3358 kB]
Get:11 http://archive.ubuntu.com/ubuntu bionic/restricted amd64 Packages [13.5 kB]
Get:12 http://archive.ubuntu.com/ubuntu bionic/universe amd64 Packages [11.3 MB]
Get:13 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages [3771 kB]
Get:14 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [2403 kB]
Get:15 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse amd64 Packages [30.8 kB]
Get:16 http://archive.ubuntu.com/ubuntu bionic-updates/restricted amd64 Packages [1709 kB]
Get:17 http://archive.ubuntu.com/ubuntu bionic-backports/main amd64 Packages [64.0 kB]
Get:18 http://archive.ubuntu.com/ubuntu bionic-backports/universe amd64 Packages [20.6 kB]
Fetched 28.1 MB in 24s (1165 kB/s)
Reading package lists...
Building dependency tree...
Reading state information...
All packages are up to date.
Removing intermediate container 6c5f02ae0949
 ---> c40f3688c101
Step 3/12 : RUN apt install -yy gcc g++ cmake
 ---> Running in 669548c95112

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.                                                                                                                                                            
                                                                                                                                                                                                                                           
Reading package lists...                                                                                                                                                                                                                   
Building dependency tree...
Reading state information...
The following additional packages will be installed:
  binutils binutils-common binutils-x86-64-linux-gnu ca-certificates
  cmake-data cpp cpp-7 g++-7 gcc-7 gcc-7-base krb5-locales libarchive13
  libasan4 libasn1-8-heimdal libatomic1 libbinutils libc-dev-bin libc6-dev
  libcc1-0 libcilkrts5 libcurl4 libexpat1 libgcc-7-dev libgomp1
  libgssapi-krb5-2 libgssapi3-heimdal libhcrypto4-heimdal libheimbase1-heimdal
  libheimntlm0-heimdal libhx509-5-heimdal libicu60 libisl19 libitm1
  libjsoncpp1 libk5crypto3 libkeyutils1 libkrb5-26-heimdal libkrb5-3
  libkrb5support0 libldap-2.4-2 libldap-common liblsan0 liblzo2-2 libmpc3
  libmpfr6 libmpx2 libnghttp2-14 libpsl5 libquadmath0 librhash0
  libroken18-heimdal librtmp1 libsasl2-2 libsasl2-modules libsasl2-modules-db
  libsqlite3-0 libssl1.1 libstdc++-7-dev libtsan0 libubsan0 libuv1
  libwind0-heimdal libxml2 linux-libc-dev make manpages manpages-dev
  multiarch-support openssl publicsuffix
Suggested packages:
  binutils-doc cmake-doc ninja-build cpp-doc gcc-7-locales g++-multilib
  g++-7-multilib gcc-7-doc libstdc++6-7-dbg gcc-multilib autoconf automake
  libtool flex bison gdb gcc-doc gcc-7-multilib libgcc1-dbg libgomp1-dbg
  libitm1-dbg libatomic1-dbg libasan4-dbg liblsan0-dbg libtsan0-dbg
  libubsan0-dbg libcilkrts5-dbg libmpx2-dbg libquadmath0-dbg lrzip glibc-doc
  krb5-doc krb5-user libsasl2-modules-gssapi-mit
  | libsasl2-modules-gssapi-heimdal libsasl2-modules-ldap libsasl2-modules-otp
  libsasl2-modules-sql libstdc++-7-doc make-doc man-browser
The following NEW packages will be installed:
  binutils binutils-common binutils-x86-64-linux-gnu ca-certificates cmake
  cmake-data cpp cpp-7 g++ g++-7 gcc gcc-7 gcc-7-base krb5-locales
  libarchive13 libasan4 libasn1-8-heimdal libatomic1 libbinutils libc-dev-bin
  libc6-dev libcc1-0 libcilkrts5 libcurl4 libexpat1 libgcc-7-dev libgomp1
  libgssapi-krb5-2 libgssapi3-heimdal libhcrypto4-heimdal libheimbase1-heimdal
  libheimntlm0-heimdal libhx509-5-heimdal libicu60 libisl19 libitm1
  libjsoncpp1 libk5crypto3 libkeyutils1 libkrb5-26-heimdal libkrb5-3
  libkrb5support0 libldap-2.4-2 libldap-common liblsan0 liblzo2-2 libmpc3
  libmpfr6 libmpx2 libnghttp2-14 libpsl5 libquadmath0 librhash0
  libroken18-heimdal librtmp1 libsasl2-2 libsasl2-modules libsasl2-modules-db
  libsqlite3-0 libssl1.1 libstdc++-7-dev libtsan0 libubsan0 libuv1
  libwind0-heimdal libxml2 linux-libc-dev make manpages manpages-dev
  multiarch-support openssl publicsuffix
0 upgraded, 73 newly installed, 0 to remove and 0 not upgraded.
Need to get 62.0 MB of archives.
After this operation, 238 MB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 multiarch-support amd64 2.27-3ubuntu1.6 [6960 B]
Get:2 http://archive.ubuntu.com/ubuntu bionic/main amd64 liblzo2-2 amd64 2.08-1.2 [48.7 kB]
Get:3 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libssl1.1 amd64 1.1.1-1ubuntu2.1~18.04.23 [1303 kB]
Get:4 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 openssl amd64 1.1.1-1ubuntu2.1~18.04.23 [614 kB]
Get:5 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 ca-certificates all 20230311ubuntu0.18.04.1 [151 kB]
Get:6 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libexpat1 amd64 2.2.5-3ubuntu0.9 [82.8 kB]
Get:7 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libicu60 amd64 60.2-3ubuntu3.2 [8050 kB]
Get:8 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libsqlite3-0 amd64 3.22.0-1ubuntu0.7 [499 kB]
Get:9 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libxml2 amd64 2.9.4+dfsg1-6.1ubuntu1.9 [663 kB]
Get:10 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 krb5-locales all 1.16-2ubuntu0.4 [13.4 kB]
Get:11 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libkrb5support0 amd64 1.16-2ubuntu0.4 [30.9 kB]
Get:12 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libk5crypto3 amd64 1.16-2ubuntu0.4 [85.3 kB]
Get:13 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libkeyutils1 amd64 1.5.9-9.2ubuntu2.1 [8764 B]
Get:14 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libkrb5-3 amd64 1.16-2ubuntu0.4 [278 kB]
Get:15 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libgssapi-krb5-2 amd64 1.16-2ubuntu0.4 [122 kB]
Get:16 http://archive.ubuntu.com/ubuntu bionic/main amd64 libpsl5 amd64 0.19.1-5build1 [41.8 kB]
Get:17 http://archive.ubuntu.com/ubuntu bionic/main amd64 manpages all 4.15-1 [1234 kB]
Get:18 http://archive.ubuntu.com/ubuntu bionic/main amd64 publicsuffix all 20180223.1310-1 [97.6 kB]
Get:19 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 binutils-common amd64 2.30-21ubuntu1~18.04.9 [197 kB]
Get:20 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libbinutils amd64 2.30-21ubuntu1~18.04.9 [489 kB]
Get:21 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 binutils-x86-64-linux-gnu amd64 2.30-21ubuntu1~18.04.9 [1839 kB]
Get:22 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 binutils amd64 2.30-21ubuntu1~18.04.9 [3392 B]
Get:23 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 cmake-data all 3.10.2-1ubuntu2.18.04.2 [1332 kB]
Get:24 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libarchive13 amd64 3.2.2-3.1ubuntu0.7 [288 kB]
Get:25 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libroken18-heimdal amd64 7.5.0+dfsg-1ubuntu0.4 [42.3 kB]
Get:26 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libasn1-8-heimdal amd64 7.5.0+dfsg-1ubuntu0.4 [175 kB]
Get:27 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libheimbase1-heimdal amd64 7.5.0+dfsg-1ubuntu0.4 [30.3 kB]
Get:28 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libhcrypto4-heimdal amd64 7.5.0+dfsg-1ubuntu0.4 [85.9 kB]
Get:29 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libwind0-heimdal amd64 7.5.0+dfsg-1ubuntu0.4 [48.0 kB]
Get:30 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libhx509-5-heimdal amd64 7.5.0+dfsg-1ubuntu0.4 [107 kB]
Get:31 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libkrb5-26-heimdal amd64 7.5.0+dfsg-1ubuntu0.4 [207 kB]
Get:32 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libheimntlm0-heimdal amd64 7.5.0+dfsg-1ubuntu0.4 [14.8 kB]
Get:33 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libgssapi3-heimdal amd64 7.5.0+dfsg-1ubuntu0.4 [96.7 kB]
Get:34 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libsasl2-modules-db amd64 2.1.27~101-g0780600+dfsg-3ubuntu2.4 [15.0 kB]
Get:35 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libsasl2-2 amd64 2.1.27~101-g0780600+dfsg-3ubuntu2.4 [49.2 kB]
Get:36 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libldap-common all 2.4.45+dfsg-1ubuntu1.11 [15.8 kB]
Get:37 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libldap-2.4-2 amd64 2.4.45+dfsg-1ubuntu1.11 [154 kB]
Get:38 http://archive.ubuntu.com/ubuntu bionic/main amd64 libnghttp2-14 amd64 1.30.0-1ubuntu1 [77.8 kB]
Get:39 http://archive.ubuntu.com/ubuntu bionic/main amd64 librtmp1 amd64 2.4+20151223.gitfa8646d.1-1 [54.2 kB]
Get:40 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libcurl4 amd64 7.58.0-2ubuntu3.24 [221 kB]
Get:41 http://archive.ubuntu.com/ubuntu bionic/main amd64 libjsoncpp1 amd64 1.7.4-3 [73.6 kB]
Get:42 http://archive.ubuntu.com/ubuntu bionic/main amd64 librhash0 amd64 1.3.6-2 [78.1 kB]
Get:43 http://archive.ubuntu.com/ubuntu bionic/main amd64 libuv1 amd64 1.18.0-3 [64.4 kB]
Get:44 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 cmake amd64 3.10.2-1ubuntu2.18.04.2 [3152 kB]
Get:45 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 gcc-7-base amd64 7.5.0-3ubuntu1~18.04 [18.3 kB]
Get:46 http://archive.ubuntu.com/ubuntu bionic/main amd64 libisl19 amd64 0.19-1 [551 kB]
Get:47 http://archive.ubuntu.com/ubuntu bionic/main amd64 libmpfr6 amd64 4.0.1-1 [243 kB]
Get:48 http://archive.ubuntu.com/ubuntu bionic/main amd64 libmpc3 amd64 1.1.0-1 [40.8 kB]
Get:49 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 cpp-7 amd64 7.5.0-3ubuntu1~18.04 [8591 kB]
Get:50 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 cpp amd64 4:7.4.0-1ubuntu2.3 [27.7 kB]
Get:51 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libcc1-0 amd64 8.4.0-1ubuntu1~18.04 [39.4 kB]
Get:52 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libgomp1 amd64 8.4.0-1ubuntu1~18.04 [76.5 kB]
Get:53 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libitm1 amd64 8.4.0-1ubuntu1~18.04 [27.9 kB]
Get:54 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libatomic1 amd64 8.4.0-1ubuntu1~18.04 [9192 B]
Get:55 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libasan4 amd64 7.5.0-3ubuntu1~18.04 [358 kB]
Get:56 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 liblsan0 amd64 8.4.0-1ubuntu1~18.04 [133 kB]
Get:57 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libtsan0 amd64 8.4.0-1ubuntu1~18.04 [288 kB]
Get:58 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libubsan0 amd64 7.5.0-3ubuntu1~18.04 [126 kB]
Get:59 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libcilkrts5 amd64 7.5.0-3ubuntu1~18.04 [42.5 kB]
Get:60 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libmpx2 amd64 8.4.0-1ubuntu1~18.04 [11.6 kB]
Get:61 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libquadmath0 amd64 8.4.0-1ubuntu1~18.04 [134 kB]
Get:62 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libgcc-7-dev amd64 7.5.0-3ubuntu1~18.04 [2378 kB]
Get:63 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 gcc-7 amd64 7.5.0-3ubuntu1~18.04 [9381 kB]
Get:64 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 gcc amd64 4:7.4.0-1ubuntu2.3 [5184 B]
Get:65 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libc-dev-bin amd64 2.27-3ubuntu1.6 [71.9 kB]
Get:66 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 linux-libc-dev amd64 4.15.0-212.223 [991 kB]
Get:67 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libc6-dev amd64 2.27-3ubuntu1.6 [2587 kB]
Get:68 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libstdc++-7-dev amd64 7.5.0-3ubuntu1~18.04 [1471 kB]
Get:69 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 g++-7 amd64 7.5.0-3ubuntu1~18.04 [9697 kB]
Get:70 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 g++ amd64 4:7.4.0-1ubuntu2.3 [1568 B]
Get:71 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libsasl2-modules amd64 2.1.27~101-g0780600+dfsg-3ubuntu2.4 [48.9 kB]
Get:72 http://archive.ubuntu.com/ubuntu bionic/main amd64 make amd64 4.1-9.1ubuntu1 [154 kB]
Get:73 http://archive.ubuntu.com/ubuntu bionic/main amd64 manpages-dev all 4.15-1 [2217 kB]
debconf: delaying package configuration, since apt-utils is not installed
Fetched 62.0 MB in 1min 15s (828 kB/s)                                                                                                                                                                                                     
Selecting previously unselected package multiarch-support.
(Reading database ... 4050 files and directories currently installed.)
Preparing to unpack .../multiarch-support_2.27-3ubuntu1.6_amd64.deb ...
Unpacking multiarch-support (2.27-3ubuntu1.6) ...
Setting up multiarch-support (2.27-3ubuntu1.6) ...
Selecting previously unselected package liblzo2-2:amd64.
(Reading database ... 4053 files and directories currently installed.)
Preparing to unpack .../00-liblzo2-2_2.08-1.2_amd64.deb ...
Unpacking liblzo2-2:amd64 (2.08-1.2) ...
Selecting previously unselected package libssl1.1:amd64.
Preparing to unpack .../01-libssl1.1_1.1.1-1ubuntu2.1~18.04.23_amd64.deb ...
Unpacking libssl1.1:amd64 (1.1.1-1ubuntu2.1~18.04.23) ...
Selecting previously unselected package openssl.
Preparing to unpack .../02-openssl_1.1.1-1ubuntu2.1~18.04.23_amd64.deb ...
Unpacking openssl (1.1.1-1ubuntu2.1~18.04.23) ...
Selecting previously unselected package ca-certificates.
Preparing to unpack .../03-ca-certificates_20230311ubuntu0.18.04.1_all.deb ...
Unpacking ca-certificates (20230311ubuntu0.18.04.1) ...
Selecting previously unselected package libexpat1:amd64.
Preparing to unpack .../04-libexpat1_2.2.5-3ubuntu0.9_amd64.deb ...
Unpacking libexpat1:amd64 (2.2.5-3ubuntu0.9) ...
Selecting previously unselected package libicu60:amd64.
Preparing to unpack .../05-libicu60_60.2-3ubuntu3.2_amd64.deb ...
Unpacking libicu60:amd64 (60.2-3ubuntu3.2) ...
Selecting previously unselected package libsqlite3-0:amd64.
Preparing to unpack .../06-libsqlite3-0_3.22.0-1ubuntu0.7_amd64.deb ...
Unpacking libsqlite3-0:amd64 (3.22.0-1ubuntu0.7) ...
Selecting previously unselected package libxml2:amd64.
Preparing to unpack .../07-libxml2_2.9.4+dfsg1-6.1ubuntu1.9_amd64.deb ...
Unpacking libxml2:amd64 (2.9.4+dfsg1-6.1ubuntu1.9) ...
Selecting previously unselected package krb5-locales.
Preparing to unpack .../08-krb5-locales_1.16-2ubuntu0.4_all.deb ...
Unpacking krb5-locales (1.16-2ubuntu0.4) ...
Selecting previously unselected package libkrb5support0:amd64.
Preparing to unpack .../09-libkrb5support0_1.16-2ubuntu0.4_amd64.deb ...
Unpacking libkrb5support0:amd64 (1.16-2ubuntu0.4) ...
Selecting previously unselected package libk5crypto3:amd64.
Preparing to unpack .../10-libk5crypto3_1.16-2ubuntu0.4_amd64.deb ...
Unpacking libk5crypto3:amd64 (1.16-2ubuntu0.4) ...
Selecting previously unselected package libkeyutils1:amd64.
Preparing to unpack .../11-libkeyutils1_1.5.9-9.2ubuntu2.1_amd64.deb ...
Unpacking libkeyutils1:amd64 (1.5.9-9.2ubuntu2.1) ...
Selecting previously unselected package libkrb5-3:amd64.
Preparing to unpack .../12-libkrb5-3_1.16-2ubuntu0.4_amd64.deb ...
Unpacking libkrb5-3:amd64 (1.16-2ubuntu0.4) ...
Selecting previously unselected package libgssapi-krb5-2:amd64.
Preparing to unpack .../13-libgssapi-krb5-2_1.16-2ubuntu0.4_amd64.deb ...
Unpacking libgssapi-krb5-2:amd64 (1.16-2ubuntu0.4) ...
Selecting previously unselected package libpsl5:amd64.
Preparing to unpack .../14-libpsl5_0.19.1-5build1_amd64.deb ...
Unpacking libpsl5:amd64 (0.19.1-5build1) ...
Selecting previously unselected package manpages.
Preparing to unpack .../15-manpages_4.15-1_all.deb ...
Unpacking manpages (4.15-1) ...
Selecting previously unselected package publicsuffix.
Preparing to unpack .../16-publicsuffix_20180223.1310-1_all.deb ...
Unpacking publicsuffix (20180223.1310-1) ...
Selecting previously unselected package binutils-common:amd64.
Preparing to unpack .../17-binutils-common_2.30-21ubuntu1~18.04.9_amd64.deb ...
Unpacking binutils-common:amd64 (2.30-21ubuntu1~18.04.9) ...
Selecting previously unselected package libbinutils:amd64.
Preparing to unpack .../18-libbinutils_2.30-21ubuntu1~18.04.9_amd64.deb ...
Unpacking libbinutils:amd64 (2.30-21ubuntu1~18.04.9) ...
Selecting previously unselected package binutils-x86-64-linux-gnu.
Preparing to unpack .../19-binutils-x86-64-linux-gnu_2.30-21ubuntu1~18.04.9_amd64.deb ...
Unpacking binutils-x86-64-linux-gnu (2.30-21ubuntu1~18.04.9) ...
Selecting previously unselected package binutils.
Preparing to unpack .../20-binutils_2.30-21ubuntu1~18.04.9_amd64.deb ...
Unpacking binutils (2.30-21ubuntu1~18.04.9) ...
Selecting previously unselected package cmake-data.
Preparing to unpack .../21-cmake-data_3.10.2-1ubuntu2.18.04.2_all.deb ...
Unpacking cmake-data (3.10.2-1ubuntu2.18.04.2) ...
Selecting previously unselected package libarchive13:amd64.
Preparing to unpack .../22-libarchive13_3.2.2-3.1ubuntu0.7_amd64.deb ...
Unpacking libarchive13:amd64 (3.2.2-3.1ubuntu0.7) ...
Selecting previously unselected package libroken18-heimdal:amd64.
Preparing to unpack .../23-libroken18-heimdal_7.5.0+dfsg-1ubuntu0.4_amd64.deb ...
Unpacking libroken18-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Selecting previously unselected package libasn1-8-heimdal:amd64.
Preparing to unpack .../24-libasn1-8-heimdal_7.5.0+dfsg-1ubuntu0.4_amd64.deb ...
Unpacking libasn1-8-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Selecting previously unselected package libheimbase1-heimdal:amd64.
Preparing to unpack .../25-libheimbase1-heimdal_7.5.0+dfsg-1ubuntu0.4_amd64.deb ...
Unpacking libheimbase1-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Selecting previously unselected package libhcrypto4-heimdal:amd64.
Preparing to unpack .../26-libhcrypto4-heimdal_7.5.0+dfsg-1ubuntu0.4_amd64.deb ...
Unpacking libhcrypto4-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Selecting previously unselected package libwind0-heimdal:amd64.
Preparing to unpack .../27-libwind0-heimdal_7.5.0+dfsg-1ubuntu0.4_amd64.deb ...
Unpacking libwind0-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Selecting previously unselected package libhx509-5-heimdal:amd64.
Preparing to unpack .../28-libhx509-5-heimdal_7.5.0+dfsg-1ubuntu0.4_amd64.deb ...
Unpacking libhx509-5-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Selecting previously unselected package libkrb5-26-heimdal:amd64.
Preparing to unpack .../29-libkrb5-26-heimdal_7.5.0+dfsg-1ubuntu0.4_amd64.deb ...
Unpacking libkrb5-26-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Selecting previously unselected package libheimntlm0-heimdal:amd64.
Preparing to unpack .../30-libheimntlm0-heimdal_7.5.0+dfsg-1ubuntu0.4_amd64.deb ...
Unpacking libheimntlm0-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Selecting previously unselected package libgssapi3-heimdal:amd64.
Preparing to unpack .../31-libgssapi3-heimdal_7.5.0+dfsg-1ubuntu0.4_amd64.deb ...
Unpacking libgssapi3-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Selecting previously unselected package libsasl2-modules-db:amd64.
Preparing to unpack .../32-libsasl2-modules-db_2.1.27~101-g0780600+dfsg-3ubuntu2.4_amd64.deb ...
Unpacking libsasl2-modules-db:amd64 (2.1.27~101-g0780600+dfsg-3ubuntu2.4) ...
Selecting previously unselected package libsasl2-2:amd64.
Preparing to unpack .../33-libsasl2-2_2.1.27~101-g0780600+dfsg-3ubuntu2.4_amd64.deb ...
Unpacking libsasl2-2:amd64 (2.1.27~101-g0780600+dfsg-3ubuntu2.4) ...
Selecting previously unselected package libldap-common.
Preparing to unpack .../34-libldap-common_2.4.45+dfsg-1ubuntu1.11_all.deb ...
Unpacking libldap-common (2.4.45+dfsg-1ubuntu1.11) ...
Selecting previously unselected package libldap-2.4-2:amd64.
Preparing to unpack .../35-libldap-2.4-2_2.4.45+dfsg-1ubuntu1.11_amd64.deb ...
Unpacking libldap-2.4-2:amd64 (2.4.45+dfsg-1ubuntu1.11) ...
Selecting previously unselected package libnghttp2-14:amd64.
Preparing to unpack .../36-libnghttp2-14_1.30.0-1ubuntu1_amd64.deb ...
Unpacking libnghttp2-14:amd64 (1.30.0-1ubuntu1) ...
Selecting previously unselected package librtmp1:amd64.
Preparing to unpack .../37-librtmp1_2.4+20151223.gitfa8646d.1-1_amd64.deb ...
Unpacking librtmp1:amd64 (2.4+20151223.gitfa8646d.1-1) ...
Selecting previously unselected package libcurl4:amd64.
Preparing to unpack .../38-libcurl4_7.58.0-2ubuntu3.24_amd64.deb ...
Unpacking libcurl4:amd64 (7.58.0-2ubuntu3.24) ...
Selecting previously unselected package libjsoncpp1:amd64.
Preparing to unpack .../39-libjsoncpp1_1.7.4-3_amd64.deb ...
Unpacking libjsoncpp1:amd64 (1.7.4-3) ...
Selecting previously unselected package librhash0:amd64.
Preparing to unpack .../40-librhash0_1.3.6-2_amd64.deb ...
Unpacking librhash0:amd64 (1.3.6-2) ...
Selecting previously unselected package libuv1:amd64.
Preparing to unpack .../41-libuv1_1.18.0-3_amd64.deb ...
Unpacking libuv1:amd64 (1.18.0-3) ...
Selecting previously unselected package cmake.
Preparing to unpack .../42-cmake_3.10.2-1ubuntu2.18.04.2_amd64.deb ...
Unpacking cmake (3.10.2-1ubuntu2.18.04.2) ...
Selecting previously unselected package gcc-7-base:amd64.
Preparing to unpack .../43-gcc-7-base_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking gcc-7-base:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package libisl19:amd64.
Preparing to unpack .../44-libisl19_0.19-1_amd64.deb ...
Unpacking libisl19:amd64 (0.19-1) ...
Selecting previously unselected package libmpfr6:amd64.
Preparing to unpack .../45-libmpfr6_4.0.1-1_amd64.deb ...
Unpacking libmpfr6:amd64 (4.0.1-1) ...
Selecting previously unselected package libmpc3:amd64.
Preparing to unpack .../46-libmpc3_1.1.0-1_amd64.deb ...
Unpacking libmpc3:amd64 (1.1.0-1) ...
Selecting previously unselected package cpp-7.
Preparing to unpack .../47-cpp-7_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking cpp-7 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package cpp.
Preparing to unpack .../48-cpp_4%3a7.4.0-1ubuntu2.3_amd64.deb ...
Unpacking cpp (4:7.4.0-1ubuntu2.3) ...
Selecting previously unselected package libcc1-0:amd64.
Preparing to unpack .../49-libcc1-0_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libcc1-0:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libgomp1:amd64.
Preparing to unpack .../50-libgomp1_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libgomp1:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libitm1:amd64.
Preparing to unpack .../51-libitm1_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libitm1:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libatomic1:amd64.
Preparing to unpack .../52-libatomic1_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libatomic1:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libasan4:amd64.
Preparing to unpack .../53-libasan4_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking libasan4:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package liblsan0:amd64.
Preparing to unpack .../54-liblsan0_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking liblsan0:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libtsan0:amd64.
Preparing to unpack .../55-libtsan0_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libtsan0:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libubsan0:amd64.
Preparing to unpack .../56-libubsan0_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking libubsan0:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package libcilkrts5:amd64.
Preparing to unpack .../57-libcilkrts5_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking libcilkrts5:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package libmpx2:amd64.
Preparing to unpack .../58-libmpx2_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libmpx2:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libquadmath0:amd64.
Preparing to unpack .../59-libquadmath0_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libquadmath0:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libgcc-7-dev:amd64.
Preparing to unpack .../60-libgcc-7-dev_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking libgcc-7-dev:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package gcc-7.
Preparing to unpack .../61-gcc-7_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking gcc-7 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package gcc.
Preparing to unpack .../62-gcc_4%3a7.4.0-1ubuntu2.3_amd64.deb ...
Unpacking gcc (4:7.4.0-1ubuntu2.3) ...
Selecting previously unselected package libc-dev-bin.
Preparing to unpack .../63-libc-dev-bin_2.27-3ubuntu1.6_amd64.deb ...
Unpacking libc-dev-bin (2.27-3ubuntu1.6) ...
Selecting previously unselected package linux-libc-dev:amd64.
Preparing to unpack .../64-linux-libc-dev_4.15.0-212.223_amd64.deb ...
Unpacking linux-libc-dev:amd64 (4.15.0-212.223) ...
Selecting previously unselected package libc6-dev:amd64.
Preparing to unpack .../65-libc6-dev_2.27-3ubuntu1.6_amd64.deb ...
Unpacking libc6-dev:amd64 (2.27-3ubuntu1.6) ...
Selecting previously unselected package libstdc++-7-dev:amd64.
Preparing to unpack .../66-libstdc++-7-dev_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking libstdc++-7-dev:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package g++-7.
Preparing to unpack .../67-g++-7_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking g++-7 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package g++.
Preparing to unpack .../68-g++_4%3a7.4.0-1ubuntu2.3_amd64.deb ...
Unpacking g++ (4:7.4.0-1ubuntu2.3) ...
Selecting previously unselected package libsasl2-modules:amd64.
Preparing to unpack .../69-libsasl2-modules_2.1.27~101-g0780600+dfsg-3ubuntu2.4_amd64.deb ...
Unpacking libsasl2-modules:amd64 (2.1.27~101-g0780600+dfsg-3ubuntu2.4) ...
Selecting previously unselected package make.
Preparing to unpack .../70-make_4.1-9.1ubuntu1_amd64.deb ...
Unpacking make (4.1-9.1ubuntu1) ...
Selecting previously unselected package manpages-dev.
Preparing to unpack .../71-manpages-dev_4.15-1_all.deb ...
Unpacking manpages-dev (4.15-1) ...
Setting up libquadmath0:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libgomp1:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libatomic1:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up manpages (4.15-1) ...
Setting up libexpat1:amd64 (2.2.5-3ubuntu0.9) ...
Setting up libicu60:amd64 (60.2-3ubuntu3.2) ...
Setting up libcc1-0:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up make (4.1-9.1ubuntu1) ...
Setting up libnghttp2-14:amd64 (1.30.0-1ubuntu1) ...
Setting up libldap-common (2.4.45+dfsg-1ubuntu1.11) ...
Setting up libuv1:amd64 (1.18.0-3) ...
Setting up libpsl5:amd64 (0.19.1-5build1) ...
Setting up libtsan0:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libsasl2-modules-db:amd64 (2.1.27~101-g0780600+dfsg-3ubuntu2.4) ...
Setting up linux-libc-dev:amd64 (4.15.0-212.223) ...
Setting up libmpfr6:amd64 (4.0.1-1) ...
Setting up libsasl2-2:amd64 (2.1.27~101-g0780600+dfsg-3ubuntu2.4) ...
Setting up cmake-data (3.10.2-1ubuntu2.18.04.2) ...
Setting up libroken18-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Setting up librtmp1:amd64 (2.4+20151223.gitfa8646d.1-1) ...
Setting up libkrb5support0:amd64 (1.16-2ubuntu0.4) ...
Setting up libxml2:amd64 (2.9.4+dfsg1-6.1ubuntu1.9) ...
Setting up librhash0:amd64 (1.3.6-2) ...
Setting up liblsan0:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up gcc-7-base:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up binutils-common:amd64 (2.30-21ubuntu1~18.04.9) ...
Setting up libmpx2:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up krb5-locales (1.16-2ubuntu0.4) ...
Setting up publicsuffix (20180223.1310-1) ...
Setting up libssl1.1:amd64 (1.1.1-1ubuntu2.1~18.04.23) ...
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Can't locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.26.1 /usr/local/share/perl/5.26.1 /usr/lib/x86_64-linux-gnu/perl5/5.26 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.26 /usr/share/perl/5.26 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 7.)
debconf: falling back to frontend: Teletype
Setting up libheimbase1-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Setting up openssl (1.1.1-1ubuntu2.1~18.04.23) ...
Setting up libsqlite3-0:amd64 (3.22.0-1ubuntu0.7) ...
Setting up libmpc3:amd64 (1.1.0-1) ...
Setting up libc-dev-bin (2.27-3ubuntu1.6) ...
Setting up libkeyutils1:amd64 (1.5.9-9.2ubuntu2.1) ...
Setting up libsasl2-modules:amd64 (2.1.27~101-g0780600+dfsg-3ubuntu2.4) ...
Setting up ca-certificates (20230311ubuntu0.18.04.1) ...
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Can't locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.26.1 /usr/local/share/perl/5.26.1 /usr/lib/x86_64-linux-gnu/perl5/5.26 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.26 /usr/share/perl/5.26 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 7.)
debconf: falling back to frontend: Teletype
Updating certificates in /etc/ssl/certs...
137 added, 0 removed; done.
Setting up manpages-dev (4.15-1) ...
Setting up libc6-dev:amd64 (2.27-3ubuntu1.6) ...
Setting up libitm1:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up liblzo2-2:amd64 (2.08-1.2) ...
Setting up libisl19:amd64 (0.19-1) ...
Setting up libjsoncpp1:amd64 (1.7.4-3) ...
Setting up libk5crypto3:amd64 (1.16-2ubuntu0.4) ...
Setting up libwind0-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Setting up libasan4:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up libbinutils:amd64 (2.30-21ubuntu1~18.04.9) ...
Setting up libarchive13:amd64 (3.2.2-3.1ubuntu0.7) ...
Setting up libcilkrts5:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up libasn1-8-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Setting up libubsan0:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up libhcrypto4-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Setting up libhx509-5-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Setting up libgcc-7-dev:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up cpp-7 (7.5.0-3ubuntu1~18.04) ...
Setting up libstdc++-7-dev:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up libkrb5-3:amd64 (1.16-2ubuntu0.4) ...
Setting up libkrb5-26-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Setting up libheimntlm0-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Setting up binutils-x86-64-linux-gnu (2.30-21ubuntu1~18.04.9) ...
Setting up cpp (4:7.4.0-1ubuntu2.3) ...
Setting up libgssapi-krb5-2:amd64 (1.16-2ubuntu0.4) ...
Setting up binutils (2.30-21ubuntu1~18.04.9) ...
Setting up libgssapi3-heimdal:amd64 (7.5.0+dfsg-1ubuntu0.4) ...
Setting up gcc-7 (7.5.0-3ubuntu1~18.04) ...
Setting up g++-7 (7.5.0-3ubuntu1~18.04) ...
Setting up gcc (4:7.4.0-1ubuntu2.3) ...
Setting up libldap-2.4-2:amd64 (2.4.45+dfsg-1ubuntu1.11) ...
Setting up g++ (4:7.4.0-1ubuntu2.3) ...
update-alternatives: using /usr/bin/g++ to provide /usr/bin/c++ (c++) in auto mode
update-alternatives: warning: skip creation of /usr/share/man/man1/c++.1.gz because associated file /usr/share/man/man1/g++.1.gz (of link group c++) doesn't exist
Setting up libcurl4:amd64 (7.58.0-2ubuntu3.24) ...
Setting up cmake (3.10.2-1ubuntu2.18.04.2) ...
Processing triggers for libc-bin (2.27-3ubuntu1.6) ...
Processing triggers for ca-certificates (20230311ubuntu0.18.04.1) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
Removing intermediate container 669548c95112
 ---> 86419bfbf930
Step 4/12 : COPY . print/
 ---> c0cdda623e5c
Step 5/12 : WORKDIR print
 ---> Running in 25696d97e204
Removing intermediate container 25696d97e204
 ---> 757f5777c630
Step 6/12 : RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
 ---> Running in 2615e5c321fc
-- The C compiler identification is GNU 7.5.0
-- The CXX compiler identification is GNU 7.5.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /print/_build
Removing intermediate container 2615e5c321fc
 ---> 3ccecb8c6522
Step 7/12 : RUN cmake --build _build
 ---> Running in bbd7330c2a9e
Scanning dependencies of target print
[ 25%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[ 50%] Linking CXX static library libprint.a
[ 50%] Built target print
Scanning dependencies of target demo
[ 75%] Building CXX object CMakeFiles/demo.dir/demo/main.cpp.o
[100%] Linking CXX executable demo
[100%] Built target demo
Removing intermediate container bbd7330c2a9e
 ---> bc1df8f4d74a
Step 8/12 : RUN cmake --build _build --target install
 ---> Running in 34d03f3b0a98
[ 50%] Built target print
[100%] Built target demo
Install the project...
-- Install configuration: "Release"
-- Installing: /print/_install/lib/libprint.a
-- Installing: /print/_install/include
-- Installing: /print/_install/include/print.hpp
-- Installing: /print/_install/cmake/print-config.cmake
-- Installing: /print/_install/cmake/print-config-release.cmake
-- Installing: /print/_install/bin/demo
Removing intermediate container 34d03f3b0a98
 ---> fdd7ac1cd024
Step 9/12 : ENV LOG_PATH /home/logs/log.txt
 ---> Running in 922e3f004497
Removing intermediate container 922e3f004497
 ---> 7ba91327d3dd
Step 10/12 : VOLUME /home/logs
 ---> Running in 0ff3bf8e8e79
Removing intermediate container 0ff3bf8e8e79
 ---> 9534013318bb
Step 11/12 : WORKDIR /print/_install/bin
 ---> Running in f36d9e0af93b
Removing intermediate container f36d9e0af93b
 ---> 6cd5ce1d90d1
Step 12/12 : ENTRYPOINT ./demo
 ---> Running in 6afd7ea923e8
Removing intermediate container 6afd7ea923e8
 ---> 45b3a0cc4e01
Successfully built 45b3a0cc4e01
Successfully tagged logger:latest

```

</p>
</details>
  
```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ docker images           
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
logger       latest    45b3a0cc4e01   8 seconds ago   336MB
ubuntu       18.04     f9a80a55f492   5 days ago      63.2MB

```

```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ mkdir logs

```

```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ sudo docker run -it -v "$(pwd)/logs/:/home/logs/" logger
text1
text2
text3

```

```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ sudo docker inspect logger

```

<details><summary>Output</summary>
<p>
  
```sh
[
    {
        "Id": "sha256:45b3a0cc4e01caabda4185f4dc59f0652ff8bd38e10e2ef7391bb5885a9fe203",
        "RepoTags": [
            "logger:latest"
        ],
        "RepoDigests": [],
        "Parent": "sha256:6cd5ce1d90d180d7cf0f2e8fc6f571a63311b08fe657a74120aecd25a00a4963",
        "Comment": "",
        "Created": "2023-06-05T05:52:52.103865381Z",
        "Container": "6afd7ea923e8c79d890baaa22743cc680459b170b3804aac50d6aa8a5583a6f7",
        "ContainerConfig": {
            "Hostname": "6afd7ea923e8",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LOG_PATH=/home/logs/log.txt"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "ENTRYPOINT [\"/bin/sh\" \"-c\" \"./demo\"]"
            ],
            "Image": "sha256:6cd5ce1d90d180d7cf0f2e8fc6f571a63311b08fe657a74120aecd25a00a4963",
            "Volumes": {
                "/home/logs": {}
            },
            "WorkingDir": "/print/_install/bin",
            "Entrypoint": [
                "/bin/sh",
                "-c",
                "./demo"
            ],
            "OnBuild": null,
            "Labels": {
                "org.opencontainers.image.ref.name": "ubuntu",
                "org.opencontainers.image.version": "18.04"
            }
        },
        "DockerVersion": "20.10.24+dfsg1",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LOG_PATH=/home/logs/log.txt"
            ],
            "Cmd": null,
            "Image": "sha256:6cd5ce1d90d180d7cf0f2e8fc6f571a63311b08fe657a74120aecd25a00a4963",
            "Volumes": {
                "/home/logs": {}
            },
            "WorkingDir": "/print/_install/bin",
            "Entrypoint": [
                "/bin/sh",
                "-c",
                "./demo"
            ],
            "OnBuild": null,
            "Labels": {
                "org.opencontainers.image.ref.name": "ubuntu",
                "org.opencontainers.image.version": "18.04"
            }
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 335594194,
        "VirtualSize": 335594194,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/3834c85d475f768d9b057167bdd3d1df1203c3004a29dca03277e88d85bec7f4/diff:/var/lib/docker/overlay2/e49a23023b9ad86eb1452223613b1f7d661024f266274b0c1457ec3b5f0d9239/diff:/var/lib/docker/overlay2/c62730ffc2403192c6f2412c5c80ac0bca7d24cd0898797d72e1d4cbe58e1933/diff:/var/lib/docker/overlay2/e3251b6e58bf345859bfbea80002478b8a9276151f3ae6f91c171f86d9906087/diff:/var/lib/docker/overlay2/affe56d8a67f5f2dac8ee71efde66d4ef05785f3d82747ea57ace80ba4bf88df/diff:/var/lib/docker/overlay2/1aaa682c376c957a7df0ae0c5c318dc4e23127d67f71471ffe1f1eff2062ccc2/diff",
                "MergedDir": "/var/lib/docker/overlay2/67e35c6153d495daca845e0c2f8e3ecc2f00f67ec9d91152fa9b181a9d00a8af/merged",
                "UpperDir": "/var/lib/docker/overlay2/67e35c6153d495daca845e0c2f8e3ecc2f00f67ec9d91152fa9b181a9d00a8af/diff",
                "WorkDir": "/var/lib/docker/overlay2/67e35c6153d495daca845e0c2f8e3ecc2f00f67ec9d91152fa9b181a9d00a8af/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:548a79621a426b4eb077c926eabac5a8620c454fb230640253e1b44dc7dd7562",
                "sha256:7beedd1557ab39ea486c9d28cd6272e6d08b758b64c8296ba3a2a5601afa394e",
                "sha256:88a71d359fd8c47c6d08df5a153b52011da20b95409c440958f2eda1428a370d",
                "sha256:638525199274df0656fc9c99e4f41a2b310565468763d879d200d9a2e2d7c691",
                "sha256:71d0aa11219acc6ad23dc1a5c8cc70dbbfa349cb87a9fc8404bc165123895852",
                "sha256:50cd452a52477c846d32522d400d81be5f9325843d34f182fbbf96bf55a3a833",
                "sha256:57b4fda61e7cf22faf19eb2974fd2c5b6d70ad8f1ba8082676b0a8db9674c158"
            ]
        },
        "Metadata": {
            "LastTagTime": "2023-06-05T01:52:52.213597285-04:00"
        }
    }
]

```
</p>
</details>

```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ cat logs/log.txt
text1
text2
text3

```

Пушу всё в репозиторий lab08
```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ git add -A

┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ git commit -m "first commit"

<details><summary>ResultOfCommand</summary>
  <p>
[master (root-commit) bead6f5] first commit
 346 files changed, 32205 insertions(+)
 create mode 100644 .Dockerfile.swo
 create mode 100644 .Dockerfile.swp
 create mode 100644 .github/workflows/CI.yml
 create mode 100644 .github/workflows/Linux.yml
 create mode 100644 .github/workflows/Windows.yml
 create mode 100644 CMakeLists.txt
 create mode 100644 CPackConfig.cmake
 create mode 100644 Dockerfile
 create mode 100644 README.md
 create mode 100644 build/CMakeCache.txt
 create mode 100644 build/CMakeFiles/3.25.1/CMakeCCompiler.cmake
 create mode 100644 build/CMakeFiles/3.25.1/CMakeCXXCompiler.cmake
 create mode 100755 build/CMakeFiles/3.25.1/CMakeDetermineCompilerABI_C.bin
 create mode 100755 build/CMakeFiles/3.25.1/CMakeDetermineCompilerABI_CXX.bin
 create mode 100644 build/CMakeFiles/3.25.1/CMakeSystem.cmake
 create mode 100644 build/CMakeFiles/3.25.1/CompilerIdC/CMakeCCompilerId.c
 create mode 100755 build/CMakeFiles/3.25.1/CompilerIdC/a.out
 create mode 100644 build/CMakeFiles/3.25.1/CompilerIdCXX/CMakeCXXCompilerId.cpp
 create mode 100755 build/CMakeFiles/3.25.1/CompilerIdCXX/a.out
 create mode 100644 build/CMakeFiles/CMakeDirectoryInformation.cmake
 create mode 100644 build/CMakeFiles/CMakeOutput.log
 create mode 100644 build/CMakeFiles/Export/272ceadb8458515b2ae4b5630a6029cc/print-config-noconfig.cmake
 create mode 100644 build/CMakeFiles/Export/272ceadb8458515b2ae4b5630a6029cc/print-config.cmake
 create mode 100644 build/CMakeFiles/Makefile.cmake
 create mode 100644 build/CMakeFiles/Makefile2
 create mode 100644 build/CMakeFiles/TargetDirectories.txt
 create mode 100644 build/CMakeFiles/cmake.check_cache
 create mode 100644 build/CMakeFiles/demo.dir/DependInfo.cmake
 create mode 100644 build/CMakeFiles/demo.dir/build.make
 create mode 100644 build/CMakeFiles/demo.dir/cmake_clean.cmake
 create mode 100644 build/CMakeFiles/demo.dir/compiler_depend.internal
 create mode 100644 build/CMakeFiles/demo.dir/compiler_depend.make
 create mode 100644 build/CMakeFiles/demo.dir/compiler_depend.ts
 create mode 100644 build/CMakeFiles/demo.dir/demo/main.cpp.o
 create mode 100644 build/CMakeFiles/demo.dir/demo/main.cpp.o.d
 create mode 100644 build/CMakeFiles/demo.dir/depend.make
 create mode 100644 build/CMakeFiles/demo.dir/flags.make
 create mode 100644 build/CMakeFiles/demo.dir/link.txt
 create mode 100644 build/CMakeFiles/demo.dir/progress.make
 create mode 100644 build/CMakeFiles/print.dir/DependInfo.cmake
 create mode 100644 build/CMakeFiles/print.dir/build.make
 create mode 100644 build/CMakeFiles/print.dir/cmake_clean.cmake
 create mode 100644 build/CMakeFiles/print.dir/cmake_clean_target.cmake
 create mode 100644 build/CMakeFiles/print.dir/compiler_depend.internal
 create mode 100644 build/CMakeFiles/print.dir/compiler_depend.make
 create mode 100644 build/CMakeFiles/print.dir/compiler_depend.ts
 create mode 100644 build/CMakeFiles/print.dir/depend.make
 create mode 100644 build/CMakeFiles/print.dir/flags.make
 create mode 100644 build/CMakeFiles/print.dir/link.txt
 create mode 100644 build/CMakeFiles/print.dir/progress.make
 create mode 100644 build/CMakeFiles/print.dir/sources/print.cpp.o
 create mode 100644 build/CMakeFiles/print.dir/sources/print.cpp.o.d
 create mode 100644 build/CMakeFiles/progress.marks
 create mode 100644 build/CPackConfig.cmake
 create mode 100644 build/CPackSourceConfig.cmake
 create mode 100644 build/Makefile
 create mode 100644 build/cmake_install.cmake
 create mode 100755 build/demo
 create mode 100644 build/libprint.a
 create mode 100644 demo/main.cpp
 create mode 100644 demo/print.hpp
 create mode 100644 formatter_ex_lib/CMakeLists.txt
 create mode 100644 formatter_ex_lib/_build/CMakeCache.txt
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/3.22.1/CMakeCCompiler.cmake
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/3.22.1/CMakeCXXCompiler.cmake
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/3.22.1/CMakeDetermineCompilerABI_C.bin
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/3.22.1/CMakeDetermineCompilerABI_CXX.bin
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/3.22.1/CMakeSystem.cmake
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/3.22.1/CompilerIdC/CMakeCCompilerId.c
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/3.22.1/CompilerIdC/a.out
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/3.22.1/CompilerIdCXX/CMakeCXXCompilerId.cpp
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/3.22.1/CompilerIdCXX/a.out
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/CMakeDirectoryInformation.cmake
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/CMakeOutput.log
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/Makefile.cmake
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/Makefile2
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/TargetDirectories.txt
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/cmake.check_cache
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/formatter_ex.dir/DependInfo.cmake
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/formatter_ex.dir/build.make
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/formatter_ex.dir/cmake_clean.cmake
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/formatter_ex.dir/cmake_clean_target.cmake
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/formatter_ex.dir/compiler_depend.make
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/formatter_ex.dir/compiler_depend.ts
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/formatter_ex.dir/depend.make
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/formatter_ex.dir/flags.make
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/formatter_ex.dir/formatter_ex.cpp.o
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/formatter_ex.dir/formatter_ex.cpp.o.d
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/formatter_ex.dir/link.txt
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/formatter_ex.dir/progress.make
 create mode 100644 formatter_ex_lib/_build/CMakeFiles/progress.marks
 create mode 100644 formatter_ex_lib/_build/Makefile
 create mode 100644 formatter_ex_lib/_build/cmake_install.cmake
 create mode 100644 formatter_ex_lib/_build/formatter/CMakeFiles/CMakeDirectoryInformation.cmake
 create mode 100644 formatter_ex_lib/_build/formatter/CMakeFiles/formatter.dir/DependInfo.cmake
 create mode 100644 formatter_ex_lib/_build/formatter/CMakeFiles/formatter.dir/build.make
 create mode 100644 formatter_ex_lib/_build/formatter/CMakeFiles/formatter.dir/cmake_clean.cmake
 create mode 100644 formatter_ex_lib/_build/formatter/CMakeFiles/formatter.dir/cmake_clean_target.cmake
 create mode 100644 formatter_ex_lib/_build/formatter/CMakeFiles/formatter.dir/compiler_depend.make
 create mode 100644 formatter_ex_lib/_build/formatter/CMakeFiles/formatter.dir/compiler_depend.ts
 create mode 100644 formatter_ex_lib/_build/formatter/CMakeFiles/formatter.dir/depend.make
 create mode 100644 formatter_ex_lib/_build/formatter/CMakeFiles/formatter.dir/flags.make
 create mode 100644 formatter_ex_lib/_build/formatter/CMakeFiles/formatter.dir/formatter.cpp.o
 create mode 100644 formatter_ex_lib/_build/formatter/CMakeFiles/formatter.dir/formatter.cpp.o.d
 create mode 100644 formatter_ex_lib/_build/formatter/CMakeFiles/formatter.dir/link.txt
 create mode 100644 formatter_ex_lib/_build/formatter/CMakeFiles/formatter.dir/progress.make
 create mode 100644 formatter_ex_lib/_build/formatter/CMakeFiles/progress.marks
 create mode 100644 formatter_ex_lib/_build/formatter/Makefile
 create mode 100644 formatter_ex_lib/_build/formatter/cmake_install.cmake
 create mode 100644 formatter_ex_lib/_build/formatter/libformatter.a
 create mode 100644 formatter_ex_lib/_build/libformatter_ex.a
 create mode 100644 formatter_ex_lib/formatter_ex.cpp
 create mode 100644 formatter_ex_lib/formatter_ex.h
 create mode 100644 formatter_lib/CMakeLists.txt
 create mode 100644 formatter_lib/_build/CMakeCache.txt
 create mode 100644 formatter_lib/_build/CMakeFiles/3.22.1/CMakeCCompiler.cmake
 create mode 100644 formatter_lib/_build/CMakeFiles/3.22.1/CMakeCXXCompiler.cmake
 create mode 100644 formatter_lib/_build/CMakeFiles/3.22.1/CMakeDetermineCompilerABI_C.bin
 create mode 100644 formatter_lib/_build/CMakeFiles/3.22.1/CMakeDetermineCompilerABI_CXX.bin
 create mode 100644 formatter_lib/_build/CMakeFiles/3.22.1/CMakeSystem.cmake
 create mode 100644 formatter_lib/_build/CMakeFiles/3.22.1/CompilerIdC/CMakeCCompilerId.c
 create mode 100644 formatter_lib/_build/CMakeFiles/3.22.1/CompilerIdC/a.out
 create mode 100644 formatter_lib/_build/CMakeFiles/3.22.1/CompilerIdCXX/CMakeCXXCompilerId.cpp
 create mode 100644 formatter_lib/_build/CMakeFiles/3.22.1/CompilerIdCXX/a.out
 create mode 100644 formatter_lib/_build/CMakeFiles/CMakeDirectoryInformation.cmake
 create mode 100644 formatter_lib/_build/CMakeFiles/CMakeOutput.log
 create mode 100644 formatter_lib/_build/CMakeFiles/Makefile.cmake
 create mode 100644 formatter_lib/_build/CMakeFiles/Makefile2
 create mode 100644 formatter_lib/_build/CMakeFiles/TargetDirectories.txt
 create mode 100644 formatter_lib/_build/CMakeFiles/cmake.check_cache
 create mode 100644 formatter_lib/_build/CMakeFiles/formatter.dir/DependInfo.cmake
 create mode 100644 formatter_lib/_build/CMakeFiles/formatter.dir/build.make
 create mode 100644 formatter_lib/_build/CMakeFiles/formatter.dir/cmake_clean.cmake
 create mode 100644 formatter_lib/_build/CMakeFiles/formatter.dir/cmake_clean_target.cmake
 create mode 100644 formatter_lib/_build/CMakeFiles/formatter.dir/compiler_depend.make
 create mode 100644 formatter_lib/_build/CMakeFiles/formatter.dir/compiler_depend.ts
 create mode 100644 formatter_lib/_build/CMakeFiles/formatter.dir/depend.make
 create mode 100644 formatter_lib/_build/CMakeFiles/formatter.dir/flags.make
 create mode 100644 formatter_lib/_build/CMakeFiles/formatter.dir/formatter.cpp.o
 create mode 100644 formatter_lib/_build/CMakeFiles/formatter.dir/formatter.cpp.o.d
 create mode 100644 formatter_lib/_build/CMakeFiles/formatter.dir/link.txt
 create mode 100644 formatter_lib/_build/CMakeFiles/formatter.dir/progress.make
 create mode 100644 formatter_lib/_build/CMakeFiles/progress.marks
 create mode 100644 formatter_lib/_build/Makefile
 create mode 100644 formatter_lib/_build/cmake_install.cmake
 create mode 100644 formatter_lib/_build/libformatter.a
 create mode 100644 formatter_lib/formatter.cpp
 create mode 100644 formatter_lib/formatter.h
 create mode 100644 hello_world_application/CMakeLists.txt
 create mode 100644 hello_world_application/_build/CMakeCache.txt
 create mode 100644 hello_world_application/_build/CMakeFiles/3.22.1/CMakeCCompiler.cmake
 create mode 100644 hello_world_application/_build/CMakeFiles/3.22.1/CMakeCXXCompiler.cmake
 create mode 100644 hello_world_application/_build/CMakeFiles/3.22.1/CMakeDetermineCompilerABI_C.bin
 create mode 100644 hello_world_application/_build/CMakeFiles/3.22.1/CMakeDetermineCompilerABI_CXX.bin
 create mode 100644 hello_world_application/_build/CMakeFiles/3.22.1/CMakeSystem.cmake
 create mode 100644 hello_world_application/_build/CMakeFiles/3.22.1/CompilerIdC/CMakeCCompilerId.c
 create mode 100644 hello_world_application/_build/CMakeFiles/3.22.1/CompilerIdC/a.out
 create mode 100644 hello_world_application/_build/CMakeFiles/3.22.1/CompilerIdCXX/CMakeCXXCompilerId.cpp
 create mode 100644 hello_world_application/_build/CMakeFiles/3.22.1/CompilerIdCXX/a.out
 create mode 100644 hello_world_application/_build/CMakeFiles/CMakeDirectoryInformation.cmake
 create mode 100644 hello_world_application/_build/CMakeFiles/CMakeOutput.log
 create mode 100644 hello_world_application/_build/CMakeFiles/Makefile.cmake
 create mode 100644 hello_world_application/_build/CMakeFiles/Makefile2
 create mode 100644 hello_world_application/_build/CMakeFiles/TargetDirectories.txt
 create mode 100644 hello_world_application/_build/CMakeFiles/cmake.check_cache
 create mode 100644 hello_world_application/_build/CMakeFiles/example.dir/DependInfo.cmake
 create mode 100644 hello_world_application/_build/CMakeFiles/example.dir/build.make
 create mode 100644 hello_world_application/_build/CMakeFiles/example.dir/cmake_clean.cmake
 create mode 100644 hello_world_application/_build/CMakeFiles/example.dir/compiler_depend.internal
 create mode 100644 hello_world_application/_build/CMakeFiles/example.dir/compiler_depend.make
 create mode 100644 hello_world_application/_build/CMakeFiles/example.dir/compiler_depend.ts
 create mode 100644 hello_world_application/_build/CMakeFiles/example.dir/depend.make
 create mode 100644 hello_world_application/_build/CMakeFiles/example.dir/flags.make
 create mode 100644 hello_world_application/_build/CMakeFiles/example.dir/hello_world.cpp.o
 create mode 100644 hello_world_application/_build/CMakeFiles/example.dir/hello_world.cpp.o.d
 create mode 100644 hello_world_application/_build/CMakeFiles/example.dir/link.txt
 create mode 100644 hello_world_application/_build/CMakeFiles/example.dir/progress.make
 create mode 100644 hello_world_application/_build/CMakeFiles/progress.marks
 create mode 100644 hello_world_application/_build/Makefile
 create mode 100644 hello_world_application/_build/cmake_install.cmake
 create mode 100644 hello_world_application/_build/example
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/CMakeDirectoryInformation.cmake
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/DependInfo.cmake
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/build.make
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/cmake_clean.cmake
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/cmake_clean_target.cmake
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/compiler_depend.internal
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/compiler_depend.make
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/compiler_depend.ts
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/depend.make
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/flags.make
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/formatter_ex.cpp.o
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/formatter_ex.cpp.o.d
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/link.txt
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/progress.make
 create mode 100644 hello_world_application/_build/formatter_ex/CMakeFiles/progress.marks
 create mode 100644 hello_world_application/_build/formatter_ex/Makefile
 create mode 100644 hello_world_application/_build/formatter_ex/cmake_install.cmake
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/CMakeDirectoryInformation.cmake
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/DependInfo.cmake
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/build.make
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/cmake_clean.cmake
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/cmake_clean_target.cmake
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/compiler_depend.internal
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/compiler_depend.make
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/compiler_depend.ts
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/depend.make
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/flags.make
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/formatter.cpp.o
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/formatter.cpp.o.d
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/link.txt
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/progress.make
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/CMakeFiles/progress.marks
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/Makefile
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/cmake_install.cmake
 create mode 100644 hello_world_application/_build/formatter_ex/formatter/libformatter.a
 create mode 100644 hello_world_application/_build/formatter_ex/libformatter_ex.a
 create mode 100644 hello_world_application/hello_world.cpp
 create mode 100644 include/print.hpp
 create mode 100644 logs/log.txt
 create mode 100644 solver_application/CMakeLists.txt
 create mode 100644 solver_application/_build/CMakeCache.txt
 create mode 100644 solver_application/_build/CMakeFiles/3.22.1/CMakeCCompiler.cmake
 create mode 100644 solver_application/_build/CMakeFiles/3.22.1/CMakeCXXCompiler.cmake
 create mode 100644 solver_application/_build/CMakeFiles/3.22.1/CMakeDetermineCompilerABI_C.bin
 create mode 100644 solver_application/_build/CMakeFiles/3.22.1/CMakeDetermineCompilerABI_CXX.bin
 create mode 100644 solver_application/_build/CMakeFiles/3.22.1/CMakeSystem.cmake
 create mode 100644 solver_application/_build/CMakeFiles/3.22.1/CompilerIdC/CMakeCCompilerId.c
 create mode 100644 solver_application/_build/CMakeFiles/3.22.1/CompilerIdC/a.out
 create mode 100644 solver_application/_build/CMakeFiles/3.22.1/CompilerIdCXX/CMakeCXXCompilerId.cpp
 create mode 100644 solver_application/_build/CMakeFiles/3.22.1/CompilerIdCXX/a.out
 create mode 100644 solver_application/_build/CMakeFiles/CMakeDirectoryInformation.cmake
 create mode 100644 solver_application/_build/CMakeFiles/CMakeOutput.log
 create mode 100644 solver_application/_build/CMakeFiles/Makefile.cmake
 create mode 100644 solver_application/_build/CMakeFiles/Makefile2
 create mode 100644 solver_application/_build/CMakeFiles/TargetDirectories.txt
 create mode 100644 solver_application/_build/CMakeFiles/cmake.check_cache
 create mode 100644 solver_application/_build/CMakeFiles/example.dir/DependInfo.cmake
 create mode 100644 solver_application/_build/CMakeFiles/example.dir/build.make
 create mode 100644 solver_application/_build/CMakeFiles/example.dir/cmake_clean.cmake
 create mode 100644 solver_application/_build/CMakeFiles/example.dir/compiler_depend.internal
 create mode 100644 solver_application/_build/CMakeFiles/example.dir/compiler_depend.make
 create mode 100644 solver_application/_build/CMakeFiles/example.dir/compiler_depend.ts
 create mode 100644 solver_application/_build/CMakeFiles/example.dir/depend.make
 create mode 100644 solver_application/_build/CMakeFiles/example.dir/equation.cpp.o
 create mode 100644 solver_application/_build/CMakeFiles/example.dir/equation.cpp.o.d
 create mode 100644 solver_application/_build/CMakeFiles/example.dir/flags.make
 create mode 100644 solver_application/_build/CMakeFiles/example.dir/link.txt
 create mode 100644 solver_application/_build/CMakeFiles/example.dir/progress.make
 create mode 100644 solver_application/_build/CMakeFiles/progress.marks
 create mode 100644 solver_application/_build/Makefile
 create mode 100644 solver_application/_build/cmake_install.cmake
 create mode 100644 solver_application/_build/example
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/CMakeDirectoryInformation.cmake
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/DependInfo.cmake
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/build.make
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/cmake_clean.cmake
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/cmake_clean_target.cmake
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/compiler_depend.internal
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/compiler_depend.make
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/compiler_depend.ts
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/depend.make
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/flags.make
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/formatter_ex.cpp.o
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/formatter_ex.cpp.o.d
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/link.txt
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/formatter_ex.dir/progress.make
 create mode 100644 solver_application/_build/formatter_ex/CMakeFiles/progress.marks
 create mode 100644 solver_application/_build/formatter_ex/Makefile
 create mode 100644 solver_application/_build/formatter_ex/cmake_install.cmake
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/CMakeDirectoryInformation.cmake
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/DependInfo.cmake
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/build.make
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/cmake_clean.cmake
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/cmake_clean_target.cmake
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/compiler_depend.internal
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/compiler_depend.make
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/compiler_depend.ts
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/depend.make
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/flags.make
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/formatter.cpp.o
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/formatter.cpp.o.d
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/link.txt
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/formatter.dir/progress.make
 create mode 100644 solver_application/_build/formatter_ex/formatter/CMakeFiles/progress.marks
 create mode 100644 solver_application/_build/formatter_ex/formatter/Makefile
 create mode 100644 solver_application/_build/formatter_ex/formatter/cmake_install.cmake
 create mode 100644 solver_application/_build/formatter_ex/formatter/libformatter.a
 create mode 100644 solver_application/_build/formatter_ex/libformatter_ex.a
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/CMakeDirectoryInformation.cmake
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/progress.marks
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/solver_lib.dir/DependInfo.cmake
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/solver_lib.dir/build.make
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/solver_lib.dir/cmake_clean.cmake
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/solver_lib.dir/cmake_clean_target.cmake
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/solver_lib.dir/compiler_depend.internal
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/solver_lib.dir/compiler_depend.make
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/solver_lib.dir/compiler_depend.ts
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/solver_lib.dir/depend.make
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/solver_lib.dir/flags.make
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/solver_lib.dir/link.txt
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/solver_lib.dir/progress.make
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/solver_lib.dir/solver.cpp.o
 create mode 100644 solver_application/_build/solver_lib/CMakeFiles/solver_lib.dir/solver.cpp.o.d
 create mode 100644 solver_application/_build/solver_lib/Makefile
 create mode 100644 solver_application/_build/solver_lib/cmake_install.cmake
 create mode 100644 solver_application/_build/solver_lib/libsolver_lib.a
 create mode 100644 solver_application/equation.cpp
 create mode 100644 solver_lib/CMakeLists.txt
 create mode 100644 solver_lib/_build/CMakeCache.txt
 create mode 100644 solver_lib/_build/CMakeFiles/3.22.1/CMakeCCompiler.cmake
 create mode 100644 solver_lib/_build/CMakeFiles/3.22.1/CMakeCXXCompiler.cmake
 create mode 100644 solver_lib/_build/CMakeFiles/3.22.1/CMakeDetermineCompilerABI_C.bin
 create mode 100644 solver_lib/_build/CMakeFiles/3.22.1/CMakeDetermineCompilerABI_CXX.bin
 create mode 100644 solver_lib/_build/CMakeFiles/3.22.1/CMakeSystem.cmake
 create mode 100644 solver_lib/_build/CMakeFiles/3.22.1/CompilerIdC/CMakeCCompilerId.c
 create mode 100644 solver_lib/_build/CMakeFiles/3.22.1/CompilerIdC/a.out
 create mode 100644 solver_lib/_build/CMakeFiles/3.22.1/CompilerIdCXX/CMakeCXXCompilerId.cpp
 create mode 100644 solver_lib/_build/CMakeFiles/3.22.1/CompilerIdCXX/a.out
 create mode 100644 solver_lib/_build/CMakeFiles/CMakeDirectoryInformation.cmake
 create mode 100644 solver_lib/_build/CMakeFiles/CMakeOutput.log
 create mode 100644 solver_lib/_build/CMakeFiles/Makefile.cmake
 create mode 100644 solver_lib/_build/CMakeFiles/Makefile2
 create mode 100644 solver_lib/_build/CMakeFiles/TargetDirectories.txt
 create mode 100644 solver_lib/_build/CMakeFiles/cmake.check_cache
 create mode 100644 solver_lib/_build/CMakeFiles/progress.marks
 create mode 100644 solver_lib/_build/CMakeFiles/solver_lib.dir/DependInfo.cmake
 create mode 100644 solver_lib/_build/CMakeFiles/solver_lib.dir/build.make
 create mode 100644 solver_lib/_build/CMakeFiles/solver_lib.dir/cmake_clean.cmake
 create mode 100644 solver_lib/_build/CMakeFiles/solver_lib.dir/cmake_clean_target.cmake
 create mode 100644 solver_lib/_build/CMakeFiles/solver_lib.dir/compiler_depend.internal
 create mode 100644 solver_lib/_build/CMakeFiles/solver_lib.dir/compiler_depend.make
 create mode 100644 solver_lib/_build/CMakeFiles/solver_lib.dir/compiler_depend.ts
 create mode 100644 solver_lib/_build/CMakeFiles/solver_lib.dir/depend.make
 create mode 100644 solver_lib/_build/CMakeFiles/solver_lib.dir/flags.make
 create mode 100644 solver_lib/_build/CMakeFiles/solver_lib.dir/link.txt
 create mode 100644 solver_lib/_build/CMakeFiles/solver_lib.dir/progress.make
 create mode 100644 solver_lib/_build/CMakeFiles/solver_lib.dir/solver.cpp.o
 create mode 100644 solver_lib/_build/CMakeFiles/solver_lib.dir/solver.cpp.o.d
 create mode 100644 solver_lib/_build/Makefile
 create mode 100644 solver_lib/_build/cmake_install.cmake
 create mode 100644 solver_lib/_build/libsolver_lib.a
 create mode 100644 solver_lib/solver.cpp
 create mode 100644 solver_lib/solver.h
 create mode 100644 sources/activate
 create mode 100644 sources/print.cpp

</p>
</details>
```
```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ git push origin master      
Username for 'https://github.com': GOSICK070404
Password for 'https://GOSICK070404@github.com': 
Enumerating objects: 285, done.
Counting objects: 100% (285/285), done.
Delta compression using up to 2 threads
Compressing objects: 100% (269/269), done.
Writing objects: 100% (285/285), 112.95 KiB | 1.53 MiB/s, done.
Total 285 (delta 154), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (154/154), done.
To https://github.com/GOSICK070404/lab08
 * [new branch]      master -> master
 ```
```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ git rm .Dockerfile.swp                                     
rm '.Dockerfile.swp'
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ git commit -m "second commit" 
[master a8aefea] second commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 delete mode 100644 .Dockerfile.swp
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ git push origin master       
Username for 'https://github.com': GOSICK070404
Password for 'https://GOSICK070404@github.com': 
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 2 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 230 bytes | 230.00 KiB/s, done.
Total 2 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/GOSICK070404/lab08
   bead6f5..a8aefea  master -> master

```
```console
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ git rm .Dockerfile.swo       
rm '.Dockerfile.swo'
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ git commit -m "second commit"
[master 1c3b5f8] second commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 delete mode 100644 .Dockerfile.swo
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab08]
└─$ git push origin master       
Username for 'https://github.com': GODICK070404
Password for 'https://GODICK070404@github.com': 
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 2 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 227 bytes | 227.00 KiB/s, done.
Total 2 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/GOSICK070404/lab08
   a8aefea..1c3b5f8  master -> master


```


```
