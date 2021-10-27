---
layout: post
title: Cross compile mtd-utils 2.1.3
subtitle: For arm and arm64
---

Install required tools

```
$ sudo apt install build-essential gcc-arm-linux-gnueabihf git git-email autoconf libtool pkg-config uuid-dev zlib1g-dev liblzo2-dev libzstd-dev
```

Cross compilation of mtd-utils involve cross-compliling dependencies too.

For clean build create a build directory.

```
$ mkdir mtdutils_build
$ cd mtdutils_build
$ BUILD_DIR=`pwd`
```

Download latest mtd-utils.
```
$ git clone -b v2.1.3 git://git.infradead.org/mtd-utils.git
```
or
```
$ wget ftp://ftp.infradead.org/pub/mtd-utils/mtd-utils-2.1.2.tar.bz2
$ tar -xjf mtd-utils-2.1.3.tar.bz2
$ mv mtd-utils-2.1.3 mtd-utils
```

Download zilb.
```
$ wget http://www.zlib.net/zlib-1.2.11.tar.gz
$ tar -xzf zlib-1.2.11.tar.gz
```

Download lzo.
```
$ wget http://www.oberhumer.com/opensource/lzo/download/lzo-2.10.tar.gz
$ tar -xzf lzo-2.10.tar.gz
```

Download e2fsprogs.
```
$ git clone -b v1.45.6 https://git.kernel.org/pub/scm/fs/ext2/e2fsprogs.git
```
or
```
$ wget https://git.kernel.org/pub/scm/fs/ext2/e2fsprogs.git/snapshot/e2fsprogs-1.45.6.tar.gz
$ tar -xzf e2fsprogs-1.45.6.tar.gz
$ mv e2fsprogs-1.45.6 e2fsprogs
```

Download zstd.
```
$ git clone --branch v1.4.5 https://github.com/facebook/zstd.git
```
or
```
$ wget https://github.com/facebook/zstd/releases/download/v1.4.5/zstd-1.4.5.tar.gz
$ tar -xzf zstd-1.4.5.tar.gz
$ mv zstd-1.4.5 zstd
```

Download openssl.
```
$ git clone -b OpenSSL_1_1_1-stable https://github.com/openssl/openssl.git
```
or
```
$ wget https://github.com/openssl/openssl/archive/OpenSSL_1_1_1-stable.zip
$ unzip OpenSSL_1_1_1-stable.zip
$ mv OpenSSL_1_1_1-stable openssl
```

Before cross-compiling we need to set some environment variables.

For ARM64:
```
$ CROSS=/path/to/aarch64-linux-gnu-
$ CROSS_NE=/path/to/aarch64-linux-gnu
```
For ARM:
```
$ CROSS=/path/to/arm-linux-gnueabihf-
$ CROSS_NE=/path/to/arm-linux-gnueabihf
```
Below are for both:
```
$ export CC=${CROSS}gcc
$ export AR=${CROSS}ar
$ export LD=${CROSS}ld
$ export STRIP=${CROSS}strip
$ export NM=${CROSS}nm
$ export RANLIB=${CROSS}ranlib
$ export OBJDUMP=${CROSS}objdump
```

Also, we need to specify the install path.

```
$ mkdir install
$ INSTALL_PATH=$(cd install; pwd)
```

Cross-compile zlib.
```
$ cd zlib-1.2.11
$ unset LDFLAGS
$ ./configure --static --prefix=$INSTALL_PATH
$ make
$ make install
$ cd $BUILD_DIR
```

Cross-compile lzo.
```
$ cd lzo-2.10
$ unset LDFLAGS
$ ./configure --host=$CROSS_NE --enable-static --disable-shared CC=${CROSS}gcc AR=${AR} LD=${LD} NM=${NM} RANLIB=${RANLIB} STRIP="${STRIP}" OBJDUMP=${OBJDUMP} --prefix=$INSTALL_PATH
$ make
$ make install
$ cd $BUILD_DIR
```

Cross-compile e2fsprogs.
```
$ cd e2fsprogs
$ unset LDFLAGS
$ ./configure --host=$CROSS_NE CC=${CROSS}gcc --with-udev-rules-dir=$INSTALL_PATH --with-crond-dir=$INSTALL_PATH --with-systemd-unit-dir=$INSTALL_PATH --prefix=$INSTALL_PATH
$ make
$ make install
$ make install-libs
$ cd lib/uuid/
$ make install
$ cd $BUILD_DIR
```

Cross-compile zstd.
```
$ cd zstd
$ make allzstd
$ cp lib/zstd.h $INSTALL_PATH/include/
$ cp lib/deprecated/zbuff.h $INSTALL_PATH/include/
$ cp lib/common/zstd_errors.h $INSTALL_PATH/include/
$ cp lib/dictBuilder/zdict.h $INSTALL_PATH/include/
$ cp lib/libzstd.a $INSTALL_PATH/lib/
$ cp lib/libzstd.pc $INSTALL_PATH/lib/pkgconfig/
$ cp lib/libzstd.so $INSTALL_PATH/lib/
$ cp lib/libzstd.so.1 $INSTALL_PATH/lib/
$ cp lib/libzstd.so.1.4.5 $INSTALL_PATH/lib/
```

Cross-compile openssl.
Set "arch_openssl" as "aarch64" for ARM64.
You can get the more info with command "./Configure os/compiler".
```
$ cd openssl
$ arch_openssl=armv4
$ ./Configure linux-$arch_openssl no-idea no-mdc2 no-rc5 no-ssl3 --prefix=$INSTALL_PATH
$ make depend
$ make
$ make install
$ cd $BUILD_DIR
```

Cross-compile mtd-utils.
```
$ cd mtd-utils
$ ./autogen.sh
$ ./configure --host=$CROSS_NE CC=${CROSS}gcc --prefix=$INSTALL_PATH LDFLAGS=-L${INSTALL_PATH}/lib CFLAGS="-DWITHOUT_XATTR -I${INSTALL_PATH}/include -g -O2 -static" LZO_CFLAGS=-I${INSTALL_PATH}/include/lzo/ ZLIB_CFLAGS=-I${INSTALL_PATH}/include ZSTD_CFLAGS=-I${INSTALL_PATH}/include --enable-static=yes
$ make
$ make install
$ cd $BUILD_DIR
```

Finally, you can find all the executables from mtd-utils in '$INSTALL_PATH/sbin'.
