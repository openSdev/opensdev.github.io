---
layout: post
title: Build nvme-cli v2.0
subtitle: Extra steps to create deb package
---

Install required tools

```
$ sudo apt install build-essential make git autoconf libtool pkg-config uuid-dev zlib1g-dev liblzo2-dev libzstd-dev meson isort python3-autopep8 flake8 mypy libhugetlbfs-dev swig debhelper dh-make
```

For clean build create a build directory.

```
$ mkdir nvme-cli_build
$ cd nvme-cli_build
```

Download latest nvme-cli.
```
$ git clone https://github.com/linux-nvme/nvme-cli.git
$ cd nvme-cli
$ git checkout -b stable-v2.0 v2.0
```

Configure and Build nvme-cli.
```
$ meson .build
$ ninja -C .build
```

We can install all the executables and libraries to the system.
Install nvme-cli.
```
$ meson install -C .build
```

For creating .deb package, first compile (do no install) the nvme-cli.
```
$ dh_make --createorig -p nvme-cli_2.0
$ dh_auto_configure --buildsystem=meson
$ dpkg-buildpackage -rfakeroot -us -uc -b
```

Now, deb package is created inside the "nvme-cli_build" directory.
Install the .deb package in the destination system.
```
sudo dpkg -i ./nvme-cli_2.0-1_amd64.deb
```

For cleaning the build directory.
```
$ ninja -C .build -t clean
$ rm -rf .build
$ find subprojects/* ! -name '*.wrap' -type d -exec rm -rf {} +
$ rm -f cscope.*
$ rm -f tags
$ rm -f ../nvme-cli-*
$ rm -f ../nvme-cli_*
```
