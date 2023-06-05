## Laboratory work VI by Polyakov Andrey


## Homework
```sh
┌──(kali㉿kali)-[~]
└─$ cd /home/kali/GOSICK070404/workspace/projects/
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects]
└─$  mkdir lab06
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects]
└─$ cd lab06                                      
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab06]
└─$ git clone https://github.com/GOSICK070404/lab04-1
Cloning into 'lab04-1'...
remote: Enumerating objects: 217, done.
remote: Counting objects: 100% (217/217), done.
remote: Compressing objects: 100% (91/91), done.
remote: Total 217 (delta 116), reused 213 (delta 114), pack-reused 0
Receiving objects: 100% (217/217), 84.57 KiB | 468.00 KiB/s, done.
Resolving deltas: 100% (116/116), done.
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab06]
└─$ git remote remove origin
fatal: not a git repository (or any of the parent directories): .git
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab06]
└─$ git remote remove origin
fatal: not a git repository (or any of the parent directories): .git
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab06]
└─$ git init                
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint: 
hint:   git config --global init.defaultBranch <name>
hint: 
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint: 
hint:   git branch -m <name>
Initialized empty Git repository in /home/kali/GOSICK070404/workspace/projects/lab06/.git/
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab06]
└─$ git remote remove origin
error: No such remote: 'origin'
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab06]
└─$ git remote add origin https://github.com/GOSICK070404/lab06                    
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab06]
└─$ git remote remove origin                                   
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab06]
└─$ git remote add origin https://github.com/GOSICK070404/lab06
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/GOSICK070404/workspace/projects/lab06]
└─$ cd lab04-1
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/…/workspace/projects/lab06/lab04-1]
└─$ git init                                                   
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint: 
hint:   git config --global init.defaultBranch <name>
hint: 
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint: 
hint:   git branch -m <name>
Initialized empty Git repository in /home/kali/GOSICK070404/workspace/projects/lab06/lab04-1/.git/
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/…/workspace/projects/lab06/lab04-1]
└─$ git remote remove origin                                   
error: No such remote: 'origin'
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/…/workspace/projects/lab06/lab04-1]
└─$ git remote add origin https://github.com/GOSICK070404/lab06
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/…/workspace/projects/lab06/lab04-1]
└─$ git remote remove origin                                   
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/…/workspace/projects/lab06/lab04-1]
└─$ git remote add origin https://github.com/GOSICK070404/lab06
                                                                                                                                                           ```
                                                                                                                                                           ```sh
┌──(kali㉿kali)-[~/…/workspace/projects/lab06/lab04-1]
└─$ nano CPackConfig.cmake
```
```sh
 include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_CONTACT donotdisturb@yandex.ru)
set(CPACK_PACKAGE_VERSION_MAJOR ${PRINT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PRINT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PRINT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_TWEAK ${PRINT_VERSION_TWEAK})
set(CPACK_PACKAGE_VERSION ${PRINT_VERSION})

set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for printing")

set(CPACK_RESOURCE_FILE_README \${CMAKE_CURRENT_SOURCE_DIR}/README.md)

set(CPACK_RPM_PACKAGE_NAME "solver")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "print-solver")
set(CPACK_RPM_PACKAGE_VERSION CPACK_PACKAGE_VERSION)

set(CPACK_DEBIAN_PACKAGE_NAME "solver")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_VERSION CPACK_PACKAGE_VERSION)

include(CPack)                                                                                                                                              ```
```sh
┌──(kali㉿kali)-[~/…/workspace/projects/lab06/lab04-1]
└─$ nano CMakeLists.txt
```
```sh
cmake_minimum_required(VERSION 3.4)
project(lab06)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include_directories("formatter_lib")
include_directories("formatter_ex_lib")
include_directories("solver_lib")

add_library(formatter_lib STATIC "formatter_lib/formatter.cpp")
add_library(formatter_ex_lib STATIC "formatter_ex_lib/formatter_ex.cpp")
add_library(solver_lib STATIC "solver_lib/solver.cpp")

add_executable(solver "solver_application/equation.cpp")

target_link_libraries(solver solver_lib formatter_ex_lib formatter_lib)

include(CPackConfig.cmake)
```
```sh
┌──(kali㉿kali)-[~/…/workspace/projects/lab06/lab04-1]
└─$ cd .github/workflows
                                                                                                                                                            ```
                                                                                                                                                            ```sh
┌──(kali㉿kali)-[~/…/lab06/lab04-1/.github/workflows]
└─$ nano CI.yml
```
```sh
  name: CMake

on:
  push:
    branches: [main]
    tags: -"v*0.*"
  pull_request:
    branches: [main]

env:
  BUILD_TYPE: Release

jobs:
  build_Linux:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Configure Solver
        run: cmake -H. -B_build -DCPACK_GENERATOR="TGZ"

      - name: Build Solver
        run: cmake --build _build --target package

      - name: Make Solver Package
        run: cd _build && cpack -G "DEB" &&
             cpack -G "RPM" &&
             mkdir ../artifacts &&
             mv *.tar.gz ../artifacts/ &&
             mv *.deb ../artifacts/ &&
             mv *.rpm ../artifacts/
      - name: Publish
        uses: actions/upload-artifact@v2
        with:
          name: DebRpm
          path: artifacts/                                                                                                                                  ```
```sh
┌──(kali㉿kali)-[~/…/lab06/lab04-1/.github/workflows]
└─$ cd .. 
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/…/projects/lab06/lab04-1/.github]
└─$ cd ..
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/…/workspace/projects/lab06/lab04-1]
└─$ git add -A                                                                                                                                                                                                 
┌──(kali㉿kali)-[~/…/workspace/projects/lab06/lab04-1]
└─$ git tag v0.5
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/…/workspace/projects/lab06/lab04-1]
└─$  git push origin main --tags
error: src refspec main does not match any
error: failed to push some refs to 'https://github.com/GOSICK070404/lab06'
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/…/workspace/projects/lab06/lab04-1]
└─$  git push origin master --tags
Username for 'https://github.com': GOSICK070404
Password for 'https://GOSICK070404@github.com': 
Enumerating objects: 217, done.
Counting objects: 100% (217/217), done.
Delta compression using up to 2 threads
Compressing objects: 100% (205/205), done.
Writing objects: 100% (217/217), 84.93 KiB | 3.54 MiB/s, done.
Total 217 (delta 114), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (114/114), done.
To https://github.com/GOSICK070404/lab06
 * [new branch]      master -> master
 * [new tag]         v0.5 -> v0.5
 ```
```
Copyright (c) 2015-2021 The ISC Authors
```
