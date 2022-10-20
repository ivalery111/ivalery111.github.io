---
layout: post
title: "[RU][UEFI][RPi3]: Часть 0: Сборка и запуск UEFI на RPi3B+"
published: true
---
# Введение
В данной заметке хочу описать сборку и запуск UEFI на RPi3+.

# Соглашения
Корневой каталог у нас - UEFI  
`UEFI$` - Команда выполняется в основной системе  
`UEFI#` - Команда выполняется в контейнере  
Флеш карта определилась как `dev/sda`  

# Сборочное окружение
Для удобства сборку будем проводить с помощью Docker.
Необходимый Dockerfile:
```Dockerfile
FROM ubuntu:22.04

ARG DEBIAN_FRONTEND=noninteractive

RUN apt update &&               \
    apt install -y              \
    git                         \
    make                        \
    vim                         \
    build-essential             \
    bison                       \
    flex                        \
    libssl-dev                  \
    gcc-arm-linux-gnueabi       \
    gcc-arm-linux-gnueabihf     \
    gcc-aarch64-linux-gnu       \
    python2                     \
    python3                     \
    python3-pip                 \
    uuid-dev                    \
    libncurses-dev              \
    iasl

CMD [ "/usr/bin/bash", "--login" ]
```
Разместим Dockerfile в каталоге UEFI/Dockerfiles

# edk2 & CO
Для сборки UEFI нам понадобятся следующие репозитории:
- [edk2 tag: edk2-stable202208](https://github.com/tianocore/edk2)
- [edk2-platforms](https://github.com/tianocore/edk2-platforms)
- [edk2-non-osi](https://github.com/tianocore/edk2-non-osi)  

Добавим их в корень проекта как submodules:
```shell
UEFI$ # edk2 submodule
UEFI$ git submodule add git@github.com:tianocore/edk2.git && cd edk2 && git checkout edk2-stable202208 && cd ..
...
UEFI$ git add ... && git commit ...
UEFI$ cd edk2 && git submodule update --init --recursive && cd ..
...
UEFI$
UEFI$ # edk2-platforms submodule
UEFI$ git submodule add git@github.com:tianocore/edk2-platforms.git
...
UEFI$ git add ... && git commit ...
UEFI$ cd edk2-platforms && git submodule update --init --recursive && cd ..
...
UEFI$
UEFI$ # edk2-platforms submodule
UEFI$ git submodule add git@github.com:tianocore/edk2-non-osi.git
...
UEFI$ git add ... && git commit ...
UEFI$
```

Настало время воспользоваться нашим docker'ом и собрать UEFI.
Начнём со сборки docker контейнера:
```bash
UEFI/Dockerfiles$ docker build -t uefi-builder -f Dockerfile .
Sending build context to Docker daemon   2.56kB
...
Successfully tagged uefi-builder:latest
UEFI/Dockerfiles$
```

Смонтируем наш текущий каталог и "провалимся" в контейнер:
```bash
UEFI$ docker run --rm -it -v $(pwd):$(pwd) --workdir=$(pwd) uefi-builder
root@...:.../UEFI#
```

Настроим окружение для сборки UEFI
```
root@...:.../UEFI# export WORKSPACE=$(pwd) && \
				   export PACKAGES_PATH=$(pwd)/edk2:$(pwd)/edk2-platforms:$(pwd)/edk2-non-osi && \
				   export NUM_CPUS=$((`getconf _NPROCESSORS_ONLN` + 2))
root@...:.../UEFI# . edk2/edksetup.sh
...
root@...:.../UEFI# make -C edk2/BaseTools
...
make: Leaving directory '/home/valery/src/Posts/UEFI/edk2/BaseTools'
root@...:.../UEFI#
```

## Сборка UEFI для RPi3:
```bash
root@...:.../UEFI# GCC5_AARCH64_PREFIX=aarch64-linux-gnu- build -n $NUM_CPUS -a AARCH64 -t GCC5 -p edk2-platforms/Platform/RaspberryPi/RPi3/RPi3.dsc
...
root@...:.../UEFI#
```

Необходимый нам в дальнешем `RPI_EFI.fd` файл находится в `Build/RPi3/DEBUG_GCC5/FV/`:
```bash
root@...:.../UEFI# ll Build/RPi3/DEBUG_GCC5/FV/
total 10900
drwxr-xr-x  3 root root    4096 Oct 18 18:34 ./
...
-rw-r--r--  1 root root 2031616 Oct 18 18:34 RPI_EFI.fd              <<<---
root@...:.../UEFI#
```

Также нам потребуются пре-собраные(pre-build) файлы прошивки RPi3. Необходимые файлы:
- [bcm2710-rpi-3-b.dtb](https://github.com/raspberrypi/firmware/blob/master/boot/bcm2710-rpi-3-b-plus.dtb)
- [bootcode.bin](https://github.com/raspberrypi/firmware/blob/master/boot/bootcode.bin)
- [fixup.dat](https://github.com/raspberrypi/firmware/blob/master/boot/fixup.dat)
- [start.elf](https://github.com/raspberrypi/firmware/blob/master/boot/start.elf)
- [miniuart-bt.dtbo](https://github.com/raspberrypi/firmware/blob/master/boot/overlays/miniuart-bt.dtbo)
- config.txt  

Эти файлы находятся в `binaries/pre-build/rpi3`

### Разметка флеш карты и запись прошивки
Для загрузки UEFI нам нужно создать раздел `FAT16` или `FAT32` и скопировать туда все pre-build файлы + собранный `RPI_EFI.fd`
```bash
UEFI$ sudo parted /dev/sda -- mklabel msdos
UEFI$ sudo parted /dev/sda -- mkpart primary fat32 0% 100%
UEFI$ sudo mount /dev/sda1 /mnt
UEFI$ sudo cp binaries/pre-build/rpi3/* /mnt
UEFI$ sudo cp Build/RPi3/DEBUG_GCC5/FV/RPI_EFI.fd /mnt
UEFI$ sync
UEFI$ sudo umount /mnt
```

Далее вставляем флешку в RPi, запускаем и должны видеть логи в UART, а после уже приглашение в Boot Manager:)

На этом всё.

# Возможные проблемы
- Плохая флеш карта

# Полезные ссылки
- [Raspberry Pi Firmware](https://github.com/raspberrypi/firmware)
- [Raspberry Pi Platform](https://github.com/tianocore/edk2-platforms/tree/master/Platform/RaspberryPi/RPi3)
