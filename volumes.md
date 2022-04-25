Volumes
========================

Volumes are one or more drives you can attach to an instance.

## Create Volume

A volume can be created using `ops volume create <vol_name>`.

You can specify the volume size with the flag `-s`. The values should respect the next examples format.
```
* 512 (512 bytes)
* 56k (56 KiloBytes)
* 500m (500 MegaBytes)
* 10g (10 GigaBytes)
```

If you have a directory in your filesystem which you want to access in the unikernel you can use the flag `-d`. Ops will create the volume and copy the directory files to the volume filesystem.

Combining the previous flags a command with the next format.
```sh
$ ops volume create <vol_name> -s <size> -d <path_to_dir>
```

Someone who needs to create a volume to host 9.6 GigaBytes of assets can use the volume create command the next way.
```sh
$ ops volume create my-assets -s 10g -d ./my-assets
```

You can check the existing ops volumes with `ops volume list`.
```sh
$ ops volume list
+--------------------------------------+------+--------+-----------+---------------------------------------------------------------------+--------------------------------+----------+
|                 UUID                 | NAME | STATUS | SIZE (GB) |                              LOCATION                               |            CREATED             | ATTACHED |
+--------------------------------------+------+--------+-----------+---------------------------------------------------------------------+--------------------------------+----------+
| e1a9a40e-8d2b-19b7-61b7-57c43f362aaa | vol  |        | 1.1 GB    | /home/xyz/.ops/volumes/vol:e1a9a40e-8d2b-19b7-61b7-57c43f362aaa.raw | 2021-03-06 09:03:07.21 +0000   |          |
|                                      |      |        |           |                                                                     | GMT                            |          |
+--------------------------------------+------+--------+-----------+---------------------------------------------------------------------+--------------------------------+----------+

```

## Setup Mount Path

Mounting a volume in an instance requires an image to know beforehand which volumes it should expect and the directories where the volumes will be mounted. The unikernel recognizes the volume by its name/label, UUID, or virtual id. The virtual id may be used when you wish to have the same volume name or mount a different volume with a different volume name to the same mount point. The virtual id syntax is currently supported on GCP, Azure and AWS.

You can specify mounts details using the command flag or the configuration file.

Using configuration file, you have to set the `Mounts` property.
```JSON
{
  "Mounts": {
    "<vol_uuid_or_vol_name_or_virtual_id>": "<path_in_unikernel_where_vol_will_be_mounted>"
  }
}

// Volume Name Example
{
  "Mounts": {
    "vol": "/files"
  }
}

// Virtual ID Example
{
  "Mounts": {
    "%1": "/files"
  }
}

```
And, then you can use the configuration file on creating the image.
```
$ ops image create <image_name> -c <configuration_file>
```

If you want to test quickly you can also use the `mounts` command flag instead.
```sh
$ ops image create <image_name> --mounts <vol_name_or_vol_uuid_or_virtual_id>/<path_to_mount>

// Volume Name Example
$ ops image create webapp --mounts vol:/files

// Virtual ID Example
$ ops image create webapp --mounts %1:/files
```

The mounts must be specified on creating an image. All commands that require to build an image accept the mounts details.
```sh
$ ops run <...>

$ ops build <...>

$ ops image create <...>

$ ops pkg load <...>

$ ops deploy <...>
```

## Attach

If your image was created with the mounts details specified and you would like to attach a volume to a running instance use next command.
```
$ ops volume attach <instance_name> <volume_name>
```

## Detach

You can detach a volume from one instance using `ops detach <instance_name> <volume_name>`

## Cloud Providers

All mounting volumes requirements applies to the cloud providers.

Ops uses cloud providers APIs to execute volumes operations on their infrastructure.

The commands format is the same, we just need to add the target cloud provider flag.
```sh
$ ops volume create <vol_name> -t <cloud_provider>

$ ops image create <image_name> --mounts <vol_name>:<mount_path> -t <cloud_provider>

$ ops instance create <image_name> --instance-name <instance_name> -t <cloud_provider>

$ ops volume attach <instance_name> <vol_name> -t <cloud_provider>
```

You can create arbitrarily large empty volumes on providers such as AWS
by specifying the requested size. AWS will re-size the volume to
whatever you set it as.

Sometimes it might be helpful to know when a volume is attached and
detached at run-time as AWS supports attaching/detaching volumes at
run-time.

You can look at the pseudo-file of ```/proc/mounts``` to determine
whether or not the volume is available. For instance:

Using this config file:
```
➜  g cat config.json
{
  "Mounts": {
    "bob": "/bob"
  }
}
```

and this program:

```
➜  g cat main.go
package main

import (
        "fmt"
        "os"
)

func main() {
        body, err := os.ReadFile("/proc/mounts")
        if err != nil {
                fmt.Println(err)
        }
        fmt.Print(string(body))
}
```

## Wrap-up

Mounting volumes require us to create an image with the mounts details, launch an instance using the image and attaching a volume. Locally, you are able to mount a volume with 2 commands.
```sh
$ ops volume create vol -s 1g -d ./files

$ ops run webapp --mounts vol:/assets
```

As mentioned above you can mount multiple volumes with the config.json
or pass multiple mounts flags like so:

```
$ ops run g --mounts vol1:/vol1 --mounts vol2:/vol2
```

See an example in our [tutorial](https://nanovms.com/dev/tutorials/working-with-unikernel-volumes-in-nanos).
