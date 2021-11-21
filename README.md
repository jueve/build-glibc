# Build glibc from source

We'll go through the following steps.

0. Preparing an environment
1. Get the soruce code
2. Check dependencies
3. Create a directory to build
4. Set configurations
5. Build

In this page, I'll build `glib-2.31` on [Ubuntu 20.04 in Docker](https://hub.docker.com/_/ubuntu).

## Preparing an environment

Create a docker container and working directory named `workdir`.

```
$ docker run -it ubuntu:20.04 /bin/bash
# apt-get update
# apt-get upgrade
# apt-get install -y wget git
# cd ~
# mkdir workdir
# cd workdir
# pwd
/root/workdir
```

## Get the source code

There are two options to get it, I think.

- Using `git`
- Download compressed file(like `.tar.gz`)

### Using git

```
# git clone https://sourceware.org/git/glibc.git
# git --work-tree=/root/workdir/glibc --git-dir=/root/workdir/glibc/.git checkout glibc-2.31
```

or

```
# git clone https://sourceware.org/git/glibc.git
# git --work-tree=/root/workdir/glibc --git-dir=/root/workdir/glibc/.git checkout release/2.31/master
```

### Download compressed file

```
# wget https://ftp.gnu.org/gnu/glibc/glibc-2.31.tar.gz
# tar -xvf glibc-2.31.tar.gz
# mv glibc-2.31 glibc
```

The [mirror list](https://www.gnu.org/prep/ftp.html) is also available.

## Check dependencies

There is the `INSTALL` file and says below. We need to install some of them with `apt-get` and `pip`.

```
Recommended Tools for Compilation
=================================

We recommend installing the following GNU tools before attempting to
build the GNU C Library:

   * GNU 'make' 4.0 or newer

     As of relase time, GNU 'make' 4.2.1 is the newest verified to work
     to build the GNU C Library.

   * GCC 6.2 or newer

     GCC 6.2 or higher is required.  In general it is recommended to use
     the newest version of the compiler that is known to work for
     building the GNU C Library, as newer compilers usually produce
     better code.  As of release time, GCC 9.2.1 is the newest compiler
     verified to work to build the GNU C Library.

     For multi-arch support it is recommended to use a GCC which has
     been built with support for GNU indirect functions.  This ensures
     that correct debugging information is generated for functions
     selected by IFUNC resolvers.  This support can either be enabled by
     configuring GCC with '--enable-gnu-indirect-function', or by
     enabling it by default by setting 'default_gnu_indirect_function'
     variable for a particular architecture in the GCC source file
     'gcc/config.gcc'.

     You can use whatever compiler you like to compile programs that use
     the GNU C Library.

     Check the FAQ for any special compiler issues on particular
     platforms.

   * GNU 'binutils' 2.25 or later

     You must use GNU 'binutils' (as and ld) to build the GNU C Library.
     No other assembler or linker has the necessary functionality at the
     moment.  As of release time, GNU 'binutils' 2.32 is the newest
     verified to work to build the GNU C Library.

   * GNU 'texinfo' 4.7 or later

     To correctly translate and install the Texinfo documentation you
     need this version of the 'texinfo' package.  Earlier versions do
     not understand all the tags used in the document, and the
     installation mechanism for the info files is not present or works
     differently.  As of release time, 'texinfo' 6.6 is the newest
     verified to work to build the GNU C Library.

   * GNU 'awk' 3.1.2, or higher

     'awk' is used in several places to generate files.  Some 'gawk'
     extensions are used, including the 'asorti' function, which was
     introduced in version 3.1.2 of 'gawk'.  As of release time, 'gawk'
     version 5.0.1 is the newest verified to work to build the GNU C
     Library.

   * GNU 'bison' 2.7 or later

     'bison' is used to generate the 'yacc' parser code in the 'intl'
     subdirectory.  As of release time, 'bison' version 3.4.1 is the
     newest verified to work to build the GNU C Library.

   * Perl 5

     Perl is not required, but if present it is used in some tests and
     the 'mtrace' program, to build the GNU C Library manual.  As of
     release time 'perl' version 5.30.1 is the newest verified to work
     to build the GNU C Library.

   * GNU 'sed' 3.02 or newer

     'Sed' is used in several places to generate files.  Most scripts
     work with any version of 'sed'.  As of release time, 'sed' version
     4.5 is the newest verified to work to build the GNU C Library.

   * Python 3.4 or later

     Python is required to build the GNU C Library.  As of release time,
     Python 3.7.4 is the newest verified to work for building and
     testing the GNU C Library.

   * PExpect 4.0

     The pretty printer tests drive GDB through test programs and
     compare its output to the printers'.  PExpect is used to capture
     the output of GDB, and should be compatible with the Python version
     in your system.  As of release time PExpect 4.3 is the newest
     verified to work to test the pretty printers.

   * GDB 7.8 or later with support for Python 2.7/3.4 or later

     GDB itself needs to be configured with Python support in order to
     use the pretty printers.  Notice that your system having Python
     available doesn't imply that GDB supports it, nor that your
     system's Python and GDB's have the same version.  As of release
     time GNU 'debugger' 8.3 is the newest verified to work to test the
     pretty printers.

     Unless Python, PExpect and GDB with Python support are present, the
     printer tests will report themselves as 'UNSUPPORTED'.  Notice that
     some of the printer tests require the GNU C Library to be compiled
     with debugging symbols.

If you change any of the 'configure.ac' files you will also need

   * GNU 'autoconf' 2.69 (exactly)

and if you change any of the message translation files you will need

   * GNU 'gettext' 0.10.36 or later

     As of release time, GNU 'gettext' version 0.19.8.1 is the newest
     version verified to work to build the GNU C Library.

You may also need these packages if you upgrade your source tree using
patches, although we try to avoid this.
```

```
# apt-get install -y gcc make gdb
# apt-get install -y texinfo gawk bison sed
# apt-get install -y python3-dev python3-pip python-is-python3
# pip install pexpect
```

## Create a directory to build

Almost of the descriptions below are written in `/root/workdir/glibc/INSTALL`. To build glibc, we have to create another directory in `workdir`.

```
# mkdir build
# cd build
# pwd
/root/workdir/build
```

## Set configuration

We set the configuration executing `/root/workdir/glibc/configure` from `build`.

```
# ../glibc/configure \
  --prefix=/root/workdir/install \
  --host=x86_64-linux-gnu \
  --build=x86_64-linux-gnu \
  CC="gcc -m64" \
  CXX="g++ -m64" \
  CFLAGS="-O2" \
  CXXFLAGS="-O2"
```

Actually, some options may be verbose. Or if you want to build with 32 bit, type below.

```
# apt-get install -y gcc-multilib g++-multilib
```

```
# ../glibc/configure \
  --prefix=/root/workdir/install \
  --host=i686-linux \
  --build=i686-linux \
  CC="gcc -m32" \
  CXX="g++ -m32" \
  CFLAGS="-O2 -march=i686" \
  CXXFLAGS="-O2 -march=i686"
```

Notice: `-march=i386` is unsupported and fails.

The binary files are installed the directory in which we specified as `--prefix`. We can see the details of options using `/root/workdir/glibc/configure --help` and `less /root/workdir/glibc/INSTALL`. Also, `gcc --target-help` is helpful to specify the CPU architecture.


After outputting logs, we got some files.

```
# ls -la
total 104
drwxr-xr-x 3 root root  4096 Nov 21 09:23 .
drwxr-xr-x 1 root root  4096 Nov 21 09:19 ..
drwxr-xr-x 2 root root  4096 Nov 21 09:23 bits
-rw-r--r-- 1 root root  8089 Nov 21 09:23 config.h
-rw-r--r-- 1 root root 35099 Nov 21 09:23 config.log
-rw-r--r-- 1 root root  4295 Nov 21 09:23 config.make
-rwxr-xr-x 1 root root 33393 Nov 21 09:23 config.status
-rw-r--r-- 1 root root   581 Nov 21 09:23 Makefile
```

## Build

Now, let's build the source.

```
# make
```

We can also execute test with `make check` but it's optional. After that, we can execute install.

```
# make install
```

(When I executed `make check`, some tests always failed. And honestly, I can't understand why. Most of them were related with `nptl`.)

Now, we got binary files.

```
# cd /root/workdir/install
# ls -la
total 40
drwxr-xr-x 10 root root 4096 Nov 21 09:48 .
drwxr-xr-x  3 root root 4096 Nov 21 09:48 ..
drwxr-xr-x  2 root root 4096 Nov 21 09:48 bin
drwxr-xr-x  2 root root 4096 Nov 21 09:48 etc
drwxr-xr-x 22 root root 4096 Nov 21 09:48 include
drwxr-xr-x  4 root root 4096 Nov 21 09:48 lib
drwxr-xr-x  3 root root 4096 Nov 21 09:48 libexec
drwxr-xr-x  2 root root 4096 Nov 21 09:48 sbin
drwxr-xr-x  5 root root 4096 Nov 21 09:48 share
drwxr-xr-x  3 root root 4096 Nov 21 09:48 var
```

```
# ls -la ./lib
total 14440
drwxr-xr-x  4 root root    4096 Nov 21 09:48 .
drwxr-xr-x 10 root root    4096 Nov 21 09:48 ..
drwxr-xr-x  2 root root    4096 Nov 21 09:48 audit
-rw-r--r--  1 root root    2168 Nov 21 09:48 crt1.o
-rw-r--r--  1 root root    1360 Nov 21 09:48 crti.o
-rw-r--r--  1 root root    1104 Nov 21 09:48 crtn.o
drwxr-xr-x  2 root root   12288 Nov 21 09:48 gconv
-rw-r--r--  1 root root    2872 Nov 21 09:48 gcrt1.o
-rwxr-xr-x  1 root root  203208 Nov 21 09:48 ld-2.31.so
lrwxrwxrwx  1 root root      10 Nov 21 09:48 ld-linux-x86-64.so.2 -> ld-2.31.so
-rwxr-xr-x  1 root root   23880 Nov 21 09:48 libanl-2.31.so
-rw-r--r--  1 root root   23780 Nov 21 09:48 libanl.a
lrwxrwxrwx  1 root root      11 Nov 21 09:48 libanl.so -> libanl.so.1
lrwxrwxrwx  1 root root      14 Nov 21 09:48 libanl.so.1 -> libanl-2.31.so
-rwxr-xr-x  1 root root   16456 Nov 21 09:48 libBrokenLocale-2.31.so
-rw-r--r--  1 root root    1878 Nov 21 09:48 libBrokenLocale.a
lrwxrwxrwx  1 root root      20 Nov 21 09:48 libBrokenLocale.so -> libBrokenLocale.so.1
lrwxrwxrwx  1 root root      23 Nov 21 09:48 libBrokenLocale.so.1 -> libBrokenLocale-2.31.so
-rwxr-xr-x  1 root root 2133816 Nov 21 09:48 libc-2.31.so
-rw-r--r--  1 root root 5401664 Nov 21 09:48 libc.a
-rw-r--r--  1 root root   26690 Nov 21 09:48 libc_nonshared.a
-rwxr-xr-x  1 root root   45440 Nov 21 09:48 libcrypt-2.31.so
-rw-r--r--  1 root root   56924 Nov 21 09:48 libcrypt.a
lrwxrwxrwx  1 root root      13 Nov 21 09:48 libcrypt.so -> libcrypt.so.1
lrwxrwxrwx  1 root root      16 Nov 21 09:48 libcrypt.so.1 -> libcrypt-2.31.so
-rw-r--r--  1 root root     369 Nov 21 09:48 libc.so
lrwxrwxrwx  1 root root      12 Nov 21 09:48 libc.so.6 -> libc-2.31.so
-rwxr-xr-x  1 root root   19328 Nov 21 09:48 libdl-2.31.so
-rw-r--r--  1 root root   16754 Nov 21 09:48 libdl.a
lrwxrwxrwx  1 root root      10 Nov 21 09:48 libdl.so -> libdl.so.2
lrwxrwxrwx  1 root root      13 Nov 21 09:48 libdl.so.2 -> libdl-2.31.so
-rw-r--r--  1 root root    1498 Nov 21 09:48 libg.a
-rw-r--r--  1 root root 3180092 Nov 21 09:48 libm-2.31.a
-rwxr-xr-x  1 root root 1458920 Nov 21 09:48 libm-2.31.so
-rw-r--r--  1 root root     174 Nov 21 09:48 libm.a
-rw-r--r--  1 root root    1864 Nov 21 09:48 libmcheck.a
-rwxr-xr-x  1 root root   23096 Nov 21 09:48 libmemusage.so
-rw-r--r--  1 root root     190 Nov 21 09:48 libm.so
lrwxrwxrwx  1 root root      12 Nov 21 09:48 libm.so.6 -> libm-2.31.so
-rwxr-xr-x  1 root root  188432 Nov 21 09:48 libmvec-2.31.so
-rw-r--r--  1 root root  364606 Nov 21 09:48 libmvec.a
lrwxrwxrwx  1 root root      12 Nov 21 09:48 libmvec.so -> libmvec.so.1
lrwxrwxrwx  1 root root      15 Nov 21 09:48 libmvec.so.1 -> libmvec-2.31.so
-rwxr-xr-x  1 root root  126352 Nov 21 09:48 libnsl-2.31.so
lrwxrwxrwx  1 root root      14 Nov 21 09:48 libnsl.so.1 -> libnsl-2.31.so
-rwxr-xr-x  1 root root   46720 Nov 21 09:48 libnss_compat-2.31.so
lrwxrwxrwx  1 root root      18 Nov 21 09:48 libnss_compat.so -> libnss_compat.so.2
lrwxrwxrwx  1 root root      21 Nov 21 09:48 libnss_compat.so.2 -> libnss_compat-2.31.so
-rwxr-xr-x  1 root root   41944 Nov 21 09:48 libnss_db-2.31.so
lrwxrwxrwx  1 root root      14 Nov 21 09:48 libnss_db.so -> libnss_db.so.2
lrwxrwxrwx  1 root root      17 Nov 21 09:48 libnss_db.so.2 -> libnss_db-2.31.so
-rwxr-xr-x  1 root root   30984 Nov 21 09:48 libnss_dns-2.31.so
lrwxrwxrwx  1 root root      15 Nov 21 09:48 libnss_dns.so -> libnss_dns.so.2
lrwxrwxrwx  1 root root      18 Nov 21 09:48 libnss_dns.so.2 -> libnss_dns-2.31.so
-rwxr-xr-x  1 root root   61480 Nov 21 09:48 libnss_files-2.31.so
lrwxrwxrwx  1 root root      17 Nov 21 09:48 libnss_files.so -> libnss_files.so.2
lrwxrwxrwx  1 root root      20 Nov 21 09:48 libnss_files.so.2 -> libnss_files-2.31.so
-rwxr-xr-x  1 root root   31712 Nov 21 09:48 libnss_hesiod-2.31.so
lrwxrwxrwx  1 root root      18 Nov 21 09:48 libnss_hesiod.so -> libnss_hesiod.so.2
lrwxrwxrwx  1 root root      21 Nov 21 09:48 libnss_hesiod.so.2 -> libnss_hesiod-2.31.so
-rwxr-xr-x  1 root root   16792 Nov 21 09:48 libpcprofile.so
-rwxr-xr-x  1 root root  159096 Nov 21 09:48 libpthread-2.31.so
-rw-r--r--  1 root root  472604 Nov 21 09:48 libpthread.a
lrwxrwxrwx  1 root root      15 Nov 21 09:48 libpthread.so -> libpthread.so.0
lrwxrwxrwx  1 root root      18 Nov 21 09:48 libpthread.so.0 -> libpthread-2.31.so
-rwxr-xr-x  1 root root  102352 Nov 21 09:48 libresolv-2.31.so
-rw-r--r--  1 root root  128090 Nov 21 09:48 libresolv.a
lrwxrwxrwx  1 root root      14 Nov 21 09:48 libresolv.so -> libresolv.so.2
lrwxrwxrwx  1 root root      17 Nov 21 09:48 libresolv.so.2 -> libresolv-2.31.so
-rwxr-xr-x  1 root root   51288 Nov 21 09:48 librt-2.31.so
-rw-r--r--  1 root root   85906 Nov 21 09:48 librt.a
lrwxrwxrwx  1 root root      10 Nov 21 09:48 librt.so -> librt.so.1
lrwxrwxrwx  1 root root      13 Nov 21 09:48 librt.so.1 -> librt-2.31.so
-rwxr-xr-x  1 root root   25816 Nov 21 09:48 libSegFault.so
-rwxr-xr-x  1 root root   46520 Nov 21 09:48 libthread_db-1.0.so
lrwxrwxrwx  1 root root      17 Nov 21 09:48 libthread_db.so -> libthread_db.so.1
lrwxrwxrwx  1 root root      19 Nov 21 09:48 libthread_db.so.1 -> libthread_db-1.0.so
-rwxr-xr-x  1 root root   18480 Nov 21 09:48 libutil-2.31.so
-rw-r--r--  1 root root   15624 Nov 21 09:48 libutil.a
lrwxrwxrwx  1 root root      12 Nov 21 09:48 libutil.so -> libutil.so.1
lrwxrwxrwx  1 root root      15 Nov 21 09:48 libutil.so.1 -> libutil-2.31.so
-rw-r--r--  1 root root    1072 Nov 21 09:48 Mcrt1.o
-rw-r--r--  1 root root    2088 Nov 21 09:48 Scrt1.o
```
