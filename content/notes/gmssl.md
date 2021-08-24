---
title: "Installing and using GmSSL on a Kali VM"
date: 2021-08-14T22:15:11Z
---
# Installing and using GmSSL on a Kali VM

## What is GmSSL?
[GmSSL](https://gmssl.org/) is an [open-source](https://github.com/guanzhi/GmSSL/) utility branched from OpenSSL and designed to support algorithms, hashes, and ciphers deemed suitable and thus mandated by the State Cryptography Administration of China, also referred to within the CCP as the Office of the Central Cryptography Leading Group. It is developed and maintained by Peking University's Information Security Laboratory and used by companies such as Baidu, Huawei, and Tencent. 

Development of [GmSSL V3](https://github.com/guanzhi/gmssl-v3-dev) is ongoing.

## Installation
Installing GmSSL went as expected.

$ curl -LO "https://github.com/guanzhi/GmSSL/archive/master.zip"
$ unzip master.zip
$ ./config ; make ; sudo make install

## Troubleshooting the error
$ gmssl version
gmssl: symbol lookup error: gmssl: undefined symbol: BIO_debug_callback, version OPENSSL_1_1_0d

$ export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

## OpenSSL breaks
openssl: symbol lookup error: openssl: undefined symbol: BIO_debug_callback, version OPENSSL_1_1_0d

## Makefile inspection
$ sudo make uninstall ; sudo make clean

## Using ldd and patchelf to modify binary
https://www.fatalerrors.org/a/0tl00jk.html

Installed: patchelf, chrpath, 

$ patchelf gmssl --print-rpath libssl.so.1.1
patchelf: getting info about 'libssl.so.1.1': No such file or directory

$ patchelf --print-needed gmssl
libssl.so.1.1
libcrypto.so.1.1
libpthread.so.0
libc.so.6

$ ldd -v gmssl
        linux-vdso.so.1 (0x00007fff817d2000)
        libssl.so.1.1 => /usr/local/lib/libssl.so.1.1 (0x00007f42660f4000)
        libcrypto.so.1.1 => /usr/local/lib/libcrypto.so.1.1 (0x00007f4265e00000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f4265dde000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f4265c19000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f4265c13000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f426625c000)

        Version information:
        ./gmssl:
                libpthread.so.0 (GLIBC_2.2.5) => /lib/x86_64-linux-gnu/libpthread.so.0
                libc.so.6 (GLIBC_2.14) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_2.3) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_2.7) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_2.2.5) => /lib/x86_64-linux-gnu/libc.so.6
                libssl.so.1.1 (OPENSSL_1_1_0d) => /usr/local/lib/libssl.so.1.1
                libcrypto.so.1.1 (OPENSSL_1_1_0d) => /usr/local/lib/libcrypto.so.1.1
        /usr/local/lib/libssl.so.1.1:
                libpthread.so.0 (GLIBC_2.2.5) => /lib/x86_64-linux-gnu/libpthread.so.0
                libc.so.6 (GLIBC_2.14) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_2.4) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_2.3.4) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_2.2.5) => /lib/x86_64-linux-gnu/libc.so.6
                libcrypto.so.1.1 (OPENSSL_1_1_0d) => /usr/local/lib/libcrypto.so.1.1
                libcrypto.so.1.1 (OPENSSL_1_1_0i) => /usr/local/lib/libcrypto.so.1.1
                libcrypto.so.1.1 (OPENSSL_1_1_0f) => /usr/local/lib/libcrypto.so.1.1
                libcrypto.so.1.1 (OPENSSL_1_1_1) => /usr/local/lib/libcrypto.so.1.1
                libcrypto.so.1.1 (OPENSSL_1_1_0) => /usr/local/lib/libcrypto.so.1.1
        /usr/local/lib/libcrypto.so.1.1:
                libdl.so.2 (GLIBC_2.2.5) => /lib/x86_64-linux-gnu/libdl.so.2
                libpthread.so.0 (GLIBC_2.2.5) => /lib/x86_64-linux-gnu/libpthread.so.0
                libc.so.6 (GLIBC_2.15) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_2.14) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_2.4) => /lib/x86_64-linux-gnu/libc.so.6
        /lib/x86_64-linux-gnu/libpthread.so.0:
                ld-linux-x86-64.so.2 (GLIBC_2.2.5) => /lib64/ld-linux-x86-64.so.2
                ld-linux-x86-64.so.2 (GLIBC_PRIVATE) => /lib64/ld-linux-x86-64.so.2
                libc.so.6 (GLIBC_2.7) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_2.14) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_2.3.2) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_2.4) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_2.2.5) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_PRIVATE) => /lib/x86_64-linux-gnu/libc.so.6
        /lib/x86_64-linux-gnu/libc.so.6:
                ld-linux-x86-64.so.2 (GLIBC_2.3) => /lib64/ld-linux-x86-64.so.2
                ld-linux-x86-64.so.2 (GLIBC_PRIVATE) => /lib64/ld-linux-x86-64.so.2
        /lib/x86_64-linux-gnu/libdl.so.2:
                ld-linux-x86-64.so.2 (GLIBC_PRIVATE) => /lib64/ld-linux-x86-64.so.2
                libc.so.6 (GLIBC_PRIVATE) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_2.4) => /lib/x86_64-linux-gnu/libc.so.6
                libc.so.6 (GLIBC_2.2.5) => /lib/x86_64-linux-gnu/libc.so.6

$ nm /usr/local/gmssl/lib/libcrypto.so.1.1
0000000000090a60 T a2d_ASN1_OBJECT
000000000009d7c0 T a2i_ASN1_ENUMERATED
000000000009d470 T a2i_ASN1_INTEGER
...7738 lines

$ nm /usr/local/gmssl/lib/libcrypto.so.1.1 | grep -i bio_debug
00000000000ae2d0 T BIO_debug_callback

Compare with...

$ nm /usr/local/lib/libcrypto.so.1.1
nm: /usr/local/lib/libcrypto.so.1.1: no symbols

`sudo patchelf --force-rpath --set-rpath /usr/local/gmssl/lib gmssl`

https://trugman-internals.com/elf-loaders-libraries-executables/
https://docs.oracle.com/cd/E19957-01/806-0641/6j9vuquit/index.html
https://crypto.stackexchange.com/questions/11278/do-any-non-us-ciphers-exist
https://www.cnblogs.com/wonz/p/14117225.html
https://carnegieendowment.org/2019/05/30/encryption-debate-in-china-pub-79216
https://stackoverflow.com/questions/13769141/can-i-change-rpath-in-an-already-compiled-binary
