Hyper-V Integration
========================

Nanos can run on hyper-v.

## Pre-requisites

Hyper-v is only supported on Windows 10/11 Enterprise, Pro and Education.

You need to enable the next Windows features:
- [WSL 2](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
- [Hyper-v](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v).


After enabling the required windows features you may be able to manage virtual machines with ops commands from a wsl shell with administrator permissions.

If you see the error message

```
exec: "qemu-img": exeuctable file not found in $PATH
```

install qemu-utils in WSL2.

As of right now OPS on Windows requires to be ran in WSL2 with admin
privileges. You can start WSL2 by right-clicking on it.

## Image Operations
### Create Image

```sh
$ ops image create <elf_file|program> -i <image_name> -t hyper-v
```

### List Images

```sh
$ ops image list -t hyper-v
+------+--------------------------------------+---------+----------------+
| NAME |                 PATH                 |  SIZE   |   CREATEDAT    |
+------+--------------------------------------+---------+----------------+
| main | /home/xyz/.ops/vhdx-images/main.vhdx | 33.6 MB | 21 minutes ago |
+------+--------------------------------------+---------+----------------+
```

### Delete Image

```
$ ops delete image <image_name> -t hyper-v
```

## Instance Operations
### Create Instance

```sh
$ ops instance create <image_name> -t hyper-v
```

### List Instances

```sh
$ ops instance list -t hyper-v
+------+---------+----------------+--------------+------+
| NAME | STATUS  | CREATED        | PRIVATE IPS  | PORT |
+------+---------+----------------+--------------+------+
| main | running | 21 minutes ago | 192.168.1.75 | 8080 |
+------+--------------------------------------+---------+
```

### Delete Instance

```sh
$ ops instance delete main -t hyper-v
```
