---
title: "Installing and using GmSSL on a Kali VM"
date: 2021-08-14T22:15:11Z
---
# Installing and using GmSSL on a Kali VM

## What is GmSSL?
[GmSSL](https://gmssl.org/) is an [open-source](https://github.com/guanzhi/GmSSL/) utility branched from OpenSSL and designed to support algorithms, hashes, and ciphers deemed suitable and thus mandated by the State Cryptography Administration of China, also referred to within the CCP as the Office of the Central Cryptography Leading Group. It is developed and maintained by Peking University's Information Security Laboratory and used by companies such as Baidu, Huawei, and Tencent. Development of [GmSSL V3](https://github.com/guanzhi/gmssl-v3-dev) is ongoing.

## Installation
Right off the bat, installation was tricky:

```
$ curl -LO "https://github.com/guanzhi/GmSSL/archive/master.zip"
$ unzip master.zip ; cd GmSSL-master
$ ./config
gmssl "glob" is not exported by the File::Glob module
...
```

This is caused by a line in two files, `Configure` and `test/build.info`:

```
use if $^O ne "VMS", 'File::Glob' => qw/glob/;
```

Now we change `qw/glob/;` to `qw/:glob/;` in `Configure:18` and `test/build.info:339`:

```
use if $^O ne "VMS", 'File::Glob' => qw/:glob/;
```

Now, installing GmSSL works fine but trying to run `gmssl` throws an error:

```
$ curl -LO "https://github.com/guanzhi/GmSSL/archive/master.zip"
$ unzip master.zip
$ ./config ; make ; sudo make install
...
$ gmssl version
gmssl: symbol lookup error: gmssl: undefined symbol: BIO_debug_callback, version OPENSSL_1_1_0d
```

This is a problem with `libssl.so.1.1` and `libcrypto.so.1.1`, versions of which are required by both GmSSL and OpenSSL.

As GmSSL is designed to serve as a full replacement for OpenSSL, its `Makefile` puts its own copies of all the necessary files, including libraries, in a folder with the `gmssl` binary, `/usr/local/gmssl` by default. It's laid out like so:

```
$ tree /usr/local/gmssl -L 2
/usr/local/gmssl
├── bin
│   ├── c_rehash
│   ├── gmssl			# GmSSL binary
│   └── openssl -> gmssl
├── include
│   └── openssl
├── lib
│   ├── engines-1.1
│   ├── libcrypto.a
│   ├── libcrypto.so -> libcrypto.so.1.1
│   ├── libcrypto.so.1.1	# dynamic library
│   ├── libssl.a
│   ├── libssl.so -> libssl.so.1.1
│   ├── libssl.so.1.1		# dynamic library
│   └── pkgconfig
├── share
│   ├── doc
│   └── man
└── ssl
    ├── certs
    ├── misc
    ├── openssl.cnf
    ├── openssl.cnf.dist
    └── private
```

An easy solution would be to add the directory containing the dynamic libraries to the `LD_LIBRARY_PATH` environment variable, but this simultaneously fixes GmSSL and may break OpenSSL. It would be better and more elegant to keep GmSSL entirely self-contained in its `/usr/local/gmssl` directory. 

Depending on what's installed on your system, running `openssl` may instead call the soft link to `gmssl`:

```
$ gmssl version
GmSSL 2.5.4 - OpenSSL 1.1.0d  19 Jun 2019
$ openssl version
GmSSL 2.5.4 - OpenSSL 1.1.0d  19 Jun 2019
```

How can we run `gmssl` and `openssl` separately, each referencing their own necessary libraries? It's obvious already, but we can run `file` on `gmssl` to confirm it's looking for dynamic libraries rather than having them built into the binary during compilation. We can also use `readelf` to see that `libssl.so.1.1` is the first of four shared libraries required by GmSSL and accessed immediately upon running:

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
...
```

## Using `ldd` and `patchelf` to modify the binary
We can also use `patchelf` to see which dependencies are required by `gmssl` and what its `RUNPATH` (or depricated `RPATH`) is. This way we can confirm that 'gmssl' first checks for `libssl.so.1.1`; we also see that `gmssl` has no set `RUNPATH`:

```
$ patchelf gmssl --print-rpath
[no output]
$ patchelf --print-needed gmssl
libssl.so.1.1
libcrypto.so.1.1
libpthread.so.0
libc.so.6
```

But which `libssl.so.1.1` file is GmSSL trying to use, if not the one inside the `/usr/local/gmssl` directory? We can run `ldd` in verbose mode to view the full paths of each library, revealing that GmSSL is instead using the libraries in `/usr/local/lib`. `ldd` further breaks down each library by version, allowing us to match the original error thrown by GmSSL with the output `libcrypto.so.1.1 (OPENSSL_1_1_0d)`.

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
        ...
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
$ sudo patchelf --force-rpath --set-rpath /usr/local/gmssl/lib gmssl
```

And... success! GmSSL and OpenSSL now run perfectly side-by-side:

```
$ gmssl version
GmSSL 2.5.4 - OpenSSL 1.1.0d  19 Jun 2019
$ openssl version
OpenSSL 1.1.1k  25 Mar 2021
```

https://trugman-internals.com/elf-loaders-libraries-executables/
https://docs.oracle.com/cd/E19957-01/806-0641/6j9vuquit/index.html
https://crypto.stackexchange.com/questions/11278/do-any-non-us-ciphers-exist
https://www.cnblogs.com/wonz/p/14117225.html
https://carnegieendowment.org/2019/05/30/encryption-debate-in-china-pub-79216
https://github.com/guanzhi/GmSSL/issues/811
https://www.fatalerrors.org/a/0tl00jk.html
https://stackoverflow.com/questions/13769141/can-i-change-rpath-in-an-already-compiled-binary
