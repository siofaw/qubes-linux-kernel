# qubes-linux-kernel
Qubes component: linux-kernel

VERY experimental. Adds patches from the [Linux Hardened](https://github.com/copperhead/linux-hardened) project, plus various patches from the [LKML](https://patchwork.kernel.org/project/LKML/list/) that address the Meltdown/Spectre issue until official fixes are merged upstream (and maybe other issues too that are discovered over time). Could break at any moment (or not even compile??); provided as-is for anyone that wants to play around with it, with no guarantees of support. Will need a Retpoline enabled compiler to fully take advantage of all of the security features offered by Retpoline (Currently gcc 7.3 out now and 8.1 to be released in March).

**Current Status**:  Includes patches from the [Feb 15, 2018](http://lkml.iu.edu/hypermail/linux/kernel/1802.1/05645.html) set of proposed x86 PTI and Spectre related fixes and updates. Compiles and boots successfully using a standard Fedora 25 build environment.

**Backporting Retpoline-enabled GCC to Fedora 25**

Prerequisite: Fedora 25 AppVM that can already compile a standard Qubes kernel.

Two options:

1) Compile Retpoline-enabled gcc yourself (currently, the 7.3 and 8.1 branches have it officially merged).

2) Backport Retpoline-enabled gcc 7.3.x RPMs from Fedora 27 Updates to Fedora 25:

- Grab all of the following Fedora 27 Updates RPMs (or the latest in the 7.3.x series) from an RPM source that you trust (Possible option: [RPMFind](https://www.rpmfind.net/linux/rpm2html/) ).
```
cpp-7.3.1-2.fc27.x86_64.rpm
gcc-7.3.1-2.fc27.x86_64.rpm
gcc-c++-7.3.1-2.fc27.x86_64.rpm
gcc-gdb-plugin-7.3.1-2.fc27.x86_64.rpm
gcc-gfortran-7.3.1-2.fc27.x86_64.rpm
gcc-gnat-7.3.1-2.fc27.x86_64.rpm
gcc-go-7.3.1-2.fc27.x86_64.rpm
gcc-objc-7.3.1-2.fc27.x86_64.rpm
gcc-objc++-7.3.1-2.fc27.x86_64.rpm
gcc-plugin-devel-7.3.1-2.fc27.x86_64.rpm
glibc-2.26-24.fc27.x86_64.rpm
glibc-all-langpacks-2.26-24.fc27.x86_64.rpm
glibc-common-2.26-24.fc27.x86_64.rpm
glibc-headers-2.26-24.fc27.x86_64.rpm
isl-0.16.1-3.fc27.x86_64.rpm
libasan-7.3.1-2.fc27.x86_64.rpm
libasan-static-7.3.1-2.fc27.x86_64.rpm
libatomic-7.3.1-2.fc27.x86_64.rpm
libatomic-static-7.3.1-2.fc27.x86_64.rpm
libcilkrts-7.3.1-2.fc27.x86_64.rpm
libcilkrts-static-7.3.1-2.fc27.x86_64.rpm
libgcc-7.3.1-2.fc27.x86_64.rpm
libgccjit-7.3.1-2.fc27.x86_64.rpm
libgccjit-devel-7.3.1-2.fc27.x86_64.rpm
libgfortran-7.3.1-2.fc27.x86_64.rpm
libgfortran-static-7.3.1-2.fc27.x86_64.rpm
libgnat-7.3.1-2.fc27.x86_64.rpm
libgnat-devel-7.3.1-2.fc27.x86_64.rpm
libgnat-static-7.3.1-2.fc27.x86_64.rpm
libgo-7.3.1-2.fc27.x86_64.rpm
libgo-devel-7.3.1-2.fc27.x86_64.rpm
libgomp-7.3.1-2.fc27.x86_64.rpm
libgomp-offload-nvptx-7.3.1-2.fc27.x86_64.rpm
libgo-static-7.3.1-2.fc27.x86_64.rpm
libitm-7.3.1-2.fc27.x86_64.rpm
libitm-devel-7.3.1-2.fc27.x86_64.rpm
libitm-static-7.3.1-2.fc27.x86_64.rpm
liblsan-7.3.1-2.fc27.x86_64.rpm
liblsan-static-7.3.1-2.fc27.x86_64.rpm
libobjc-7.3.1-2.fc27.x86_64.rpm
libquadmath-7.3.1-2.fc27.x86_64.rpm
libquadmath-devel-7.3.1-2.fc27.x86_64.rpm
libquadmath-static-7.3.1-2.fc27.x86_64.rpm
libstdc++-7.3.1-2.fc27.x86_64.rpm
libstdc++-devel-7.3.1-2.fc27.x86_64.rpm
libstdc++-docs-7.3.1-2.fc27.x86_64.rpm
libstdc++-static-7.3.1-2.fc27.x86_64.rpm
libtsan-7.3.1-2.fc27.x86_64.rpm
libtsan-static-7.3.1-2.fc27.x86_64.rpm
libubsan-7.3.1-2.fc27.x86_64.rpm
libubsan-static-7.3.1-2.fc27.x86_64.rpm

```
Alternatively, you could download the src rpms and compile things yourself (that exercise is left to the reader).
- In the Fedora 25 AppVM, install all RPMs using <code>sudo rpm -ivh --force *.rpm</code>. If complaints about missing system packages appear, make note of them, then in the Fedora 25 TemplateVM, install the Fedora 25 equivalents of the missing packages (<code> sudo dnf install *packagename*</code>). Then shut down the TemplateVM, reboot the AppVM, and then try installing the new rpms again. Keep repeating until the backported rpms install cleanly.
- Verify that your AppVM is now using the new 7.3 version of gcc by running <code> gcc -v </code>. It should look something like this:
```
[user@KernelBuilder-fc25 gcc]$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/7/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-redhat-linux
Configured with: ../configure --enable-bootstrap --enable-languages=c,c++,objc,obj-c++,fortran,ada,go,lto --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-shared --enable-threads=posix --enable-checking=release --enable-multilib --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-gcc-major-version-only --with-linker-hash-style=gnu --enable-plugin --enable-initfini-array --with-isl --enable-libmpx --enable-offload-targets=nvptx-none --without-cuda-driver --enable-gnu-indirect-function --with-tune=generic --with-arch_32=i686 --build=x86_64-redhat-linux
Thread model: posix
gcc version 7.3.1 20180130 (Red Hat 7.3.1-2) (GCC) 
```
- Once done, run <code>make rpms</code> in the kernel build directory and the build process will use the new Retpoline-enabled gcc compiler instead of the standard Fedora 25 version. To make the compiler changes permanent, install the backported rpms into the TemplateVM instead of the AppVM.
- You can verify that Spectre v1 mitigations and Retpoline protection for Spectre v2 are applied by running:
```
grep . /sys/devices/system/cpu/vulnerabilities/spectre*
```
in dom0 or VM. On a Sandy Bridge system, the output looks something like this (Spectre v2 display output may depend on your CPU):
```
user@disp1:~$ grep . /sys/devices/system/cpu/vulnerabilities/spectre*
/sys/devices/system/cpu/vulnerabilities/spectre_v1:Mitigation: __user pointer sanitization
/sys/devices/system/cpu/vulnerabilities/spectre_v2:Mitigation: Full generic retpoline

```
