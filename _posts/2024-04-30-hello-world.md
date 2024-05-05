---
layout: single
title: "Docker Hello World"
categories : Docker
tag: [Build, figlet]
toc: true
toc_sticky: true
author_profile: false
---

Docker을 이용해서 Hello World 작업을 해보자.

## 파일 생성
```bash
snoopy_kr@iMac ~ % cat message
Hello World...!!!

snoopy_kr@iMac ~ % cat Dockerfile
FROM alpine:latest
RUN apk update && apk add figlet
ADD ./message /message
CMD cat /message | figlet
```
## build
```bash
snoopy_kr@iMac ~ % docker build --tag hello:1.0 .
```

## 확인
```bash
snoopy_kr@iMac ~ % docker images
REPOSITORY                           TAG                                                     IMAGE ID       CREATED          SIZE
hello                                1.0                                                     0e664f242f0d   29 seconds ago   8.55MB
```

## 실행
```bash
snoopy_kr@iMac ~ % docker run hello:1.0
 _   _      _ _        __        __         _     _       _ _ _ 
| | | | ___| | | ___   \ \      / /__  _ __| | __| |     | | | |
| |_| |/ _ \ | |/ _ \   \ \ /\ / / _ \| '__| |/ _` |     | | | |
|  _  |  __/ | | (_) |   \ V  V / (_) | |  | | (_| |_ _ _|_|_|_|
|_| |_|\___|_|_|\___/     \_/\_/ \___/|_|  |_|\__,_(_|_|_|_|_|_)
```

## [참고] image layer 정보
```bash
snoopy_kr@iMac ~ % docker inspect f2f58050ed69
[
    {
        "Id": "sha256:f2f58050ed69eac4d2aee85fc952086f4a6610facf22179131315c75d690923b",
        "RepoTags": [
            "hello:1.0"
        ],
        "RepoDigests": [],
        "Parent": "",
        "Comment": "buildkit.dockerfile.v0",
        "Created": "2023-07-17T15:10:21.136519804Z",
        "Container": "",
        "ContainerConfig": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": null,
            "Cmd": null,
            "Image": "",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "DockerVersion": "",
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
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "cat /message | figlet"
            ],
            "ArgsEscaped": true,
            "Image": "",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 9882663,
        "VirtualSize": 9882663,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/v3u382w8hud9jbm3f6gf9j6ad/diff:/var/lib/docker/overlay2/0b25a55534d49835f6adea025e6e8a21b493499040fec16b922f834482f00bc3/diff",
                "MergedDir": "/var/lib/docker/overlay2/ikdnvkvx5j8fxe817r9j3x2lc/merged",
                "UpperDir": "/var/lib/docker/overlay2/ikdnvkvx5j8fxe817r9j3x2lc/diff",
                "WorkDir": "/var/lib/docker/overlay2/ikdnvkvx5j8fxe817r9j3x2lc/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:78a822fe2a2d2c84f3de4a403188c45f623017d6a4521d23047c9fbb0801794c",
                "sha256:716ded151116ee171e8e9814056d5845e4b61537c541ff52e0a19e4a53e677fe",
                "sha256:2c48e7f83623eba43ce3998e15a949dbdf20a83a4d18343465be75defb93d12b"
            ]
        },
        "Metadata": {
            "LastTagTime": "2023-07-17T15:10:21.176766133Z"
        }
    }
]
```
## [참고] run 형식
```
$ docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```