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

```
$ curl -LO "https://github.com/guanzhi/GmSSL/archive/master.zip"
$ unzip master.zip
$ ./config ; make ; sudo make install
```

## Troubleshooting the error
```
$ gmssl version
gmssl: symbol lookup error: gmssl: undefined symbol: BIO_debug_callback, version OPENSSL_1_1_0d
```

`$ export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH`

## OpenSSL breaks
`openssl: symbol lookup error: openssl: undefined symbol: BIO_debug_callback, version OPENSSL_1_1_0d`

## Makefile inspection
`$ sudo make uninstall ; sudo make clean`

## Inspecting the `gmssl` binary
```
$ file gmssl
gmssl: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=52bb0f6115f7c840ef03fa854fff33e97602da8a, for GNU/Linux 3.2.0, not stripped
$ readelf -d gmssl

Dynamic section at offset 0xa1cd8 contains 29 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libssl.so.1.1]
 0x0000000000000001 (NEEDED)             Shared library: [libcrypto.so.1.1]
 0x0000000000000001 (NEEDED)             Shared library: [libpthread.so.0]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
 0x000000000000000c (INIT)               0x2d000
 0x000000000000000d (FINI)               0x82ca4
 0x0000000000000019 (INIT_ARRAY)         0xa25b0
 0x000000000000001b (INIT_ARRAYSZ)       8 (bytes)
 0x000000000000001a (FINI_ARRAY)         0xa25b8
 0x000000000000001c (FINI_ARRAYSZ)       8 (bytes)
 0x000000006ffffef5 (GNU_HASH)           0x308
 0x0000000000000005 (STRTAB)             0x8378
 0x0000000000000006 (SYMTAB)             0x3c8
 0x000000000000000a (STRSZ)              24908 (bytes)
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000015 (DEBUG)              0x0
 0x0000000000000003 (PLTGOT)             0xa3000
 0x0000000000000002 (PLTRELSZ)           31680 (bytes)
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000017 (JMPREL)             0x25248
 0x0000000000000007 (RELA)               0xf018
 0x0000000000000008 (RELASZ)             90672 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x000000006ffffffb (FLAGS_1)            Flags: PIE
 0x000000006ffffffe (VERNEED)            0xef68
 0x000000006fffffff (VERNEEDNUM)         4
 0x000000006ffffff0 (VERSYM)             0xe4c4
 0x000000006ffffff9 (RELACOUNT)          3737
 0x0000000000000000 (NULL)               0x0
```

## Using ldd and patchelf to modify binary
https://www.fatalerrors.org/a/0tl00jk.html

Installed: patchelf, chrpath, 

```
$ patchelf gmssl --print-rpath libssl.so.1.1
patchelf: getting info about 'libssl.so.1.1': No such file or directory
```

```
$ patchelf --print-needed gmssl
libssl.so.1.1
libcrypto.so.1.1
libpthread.so.0
libc.so.6
```

```
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
```

Using `nm` we can enumerate all the symbols in a given dynamic library (`.so`) file:

```
$ nm /usr/local/gmssl/lib/libcrypto.so.1.1
0000000000090a60 T a2d_ASN1_OBJECT
000000000009d7c0 T a2i_ASN1_ENUMERATED
000000000009d470 T a2i_ASN1_INTEGER
... # continues to 7,738 lines
```

`nm` gives us thousands of lines of output; luckily we can `grep` the output for `BIO_debug_callback`, the exact symbol GmSSL says is undefined:

```
$ nm /usr/local/gmssl/lib/libcrypto.so.1.1 | grep -i bio_debug
00000000000ae2d0 T BIO_debug_callback
```

Looks like the program will run if we can point it to the correct library, GmSSL's version of `libcrypto.so.1.1`. To check, let's compare that file with OpenSSL's version:

```
$ nm /usr/local/lib/libcrypto.so.1.1
nm: /usr/local/lib/libcrypto.so.1.1: no symbols
```

The problem is confirmed - GmSSL is looking in OpenSSL's library for symbols that don't exist. So how can we change where GmSSL looks for runtime libraries? A common tool is `chrpath` which sadly won't work for us, as our GmSSL binary currently lacks any `RUNPATH`/`RPATH` tags:

```
$ readelf -d gmssl | grep -E "RUNPATH|RPATH"
[no output]
```

Instead we can install and run `patchelf` to modify the `RUNPATH`, which lists directories containing dependency libraries:

```
$ sudo apt install patchelf
$ sudo patchelf --force-rpath --set-rpath /usr/local/gmssl/lib gmssl
```

And... success! GmSSL now runs perfectly:

```
$ gmssl version
GmSSL 2.5.4 - OpenSSL 1.1.0d  19 Jun 2019
```

https://trugman-internals.com/elf-loaders-libraries-executables/
https://docs.oracle.com/cd/E19957-01/806-0641/6j9vuquit/index.html
https://crypto.stackexchange.com/questions/11278/do-any-non-us-ciphers-exist
https://www.cnblogs.com/wonz/p/14117225.html
https://carnegieendowment.org/2019/05/30/encryption-debate-in-china-pub-79216
https://stackoverflow.com/questions/13769141/can-i-change-rpath-in-an-already-compiled-binary
