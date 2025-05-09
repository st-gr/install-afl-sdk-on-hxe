---
title: Install AFL SDK on HANA Express - Chapter 4
author: st-gr
date: 5/6/2025
mainfont: Helvetica, Arial, sans-serif
fontsize: 18px
---

Install AFL SDK on HANA Express
===============================

***Guide to install the SAP Application Function Library SDK on SAP HANA Express***

**Author:** *st-gr*

[<< Previous Chapter](chapter-3-install-afl-sdk.md) | [Content Table](README.md) | [Next Chapter >>](chapter-5-prepare-and-build-first-afl.md)

---

## Chapter Four: Determine and install GCC version

By reading the documentation we learned that AFL are coded in C++ 11 and should be compiled with a gcc compiler ideally matching the compiler that was used to compile the HANA DB itself. This will prevent potential [ABI][5] conflicts.

### Determine the compiler version of a HDB executable

By reading the documentation we learned that AFL run as an instance on the script server. If we match the compiler version of the `hdbscriptserver` executable then chances are good that our AFL will have "no" ABI conflicts when executed.

Get the compiler version:

````
# readelf -p .comment $(which hdbscriptserver)

String dump of section '.comment':
  [     1]  GCC: (SUSE Linux) 4.8.5
  [    19]  GCC: ('SAP release 11.3.0-sap7') 11.3.0
````

In our case, **GCC 11.3.0 (SAP release)** was probably the primary compiler used to build the executable, while the GCC 4.8.5 references might come from system libraries or components that were linked into the final executable.

### Install GCC 11.3.0 from RPM packages

SUSE Enterprise Linux is commercial software. Without a paid registration we won't be able to download packages using zypper.

We are **Not registered**:
````
# sudo SUSEConnect --status
[{"identifier":"SLES","version":"15.3","arch":"x86_64","status":"Not Registered"}]
````

However, for our testing purposes we can make use of OpenSUSE RPM packages and manually install them.

Follow these steps to manually install GCC 11.3 on SUSE Linux Enterprise 15 using RPM packages.

#### Step 1: Create a Working Directory

```bash
mkdir ~/gcc11 && cd ~/gcc11
```

> Just to keep files together.

#### Step 2: Download the Core 11.3 Toolchain Packages

```bash
base=https://download.opensuse.org/distribution/leap/15.5/repo/oss/x86_64

wget $base/gcc11-11.3.0+git1637-150000.1.11.2.x86_64.rpm
wget $base/gcc11-c++-11.3.0+git1637-150000.1.11.2.x86_64.rpm
wget $base/libstdc++6-devel-gcc11-11.3.0+git1637-150000.1.11.2.x86_64.rpm
```

> These three RPMs include the 11.3 compiler driver, C++ front end, standard library headers, and runtime.
> We will use the libgcc version that is installed on HXE: `rpm -q libgcc_s1` -> `libgcc_s1-13.2.1+git8285-150000.1.9.1.x86_64`

#### Step 3: Install the Core Packages (Ignore Dependencies for Now)

```bash
sudo rpm -Uvh --nodeps *.rpm
```

> This will install the RPMs; we’ll resolve missing dependencies in the next steps.

---

#### Step 4: Download and Install `cpp11` Front-End

```bash
wget https://download.opensuse.org/repositories/devel:/gcc/SLE-15/x86_64/cpp11-11.5.0+git3328-150300.2.1.x86_64.rpm
sudo rpm -Uvh cpp11-11.5.0+git3328-150300.2.1.x86_64.rpm
```

> `cpp11` provides **cc1** and **/usr/bin/cpp-11**. This 11.5 back-end works with the 11.3 driver.

#### Step 5: Temporarily Add openSUSE Leap 15.3 OSS Repo

```bash
sudo zypper ar -p98 https://download.opensuse.org/distribution/leap/15.3/repo/oss/ openSUSE-Leap-15_3-OSS
sudo zypper ref openSUSE-Leap-15_3-OSS
```

> Leap and SLE 15 share ABI compatibility.

#### Step 6: Install Startup Headers

```bash
sudo zypper in --from openSUSE-Leap-15_3-OSS glibc-devel
```

> Installs `glibc-devel`, `libxcrypt-devel`, `linux-glibc-devel` to satisfy compile-time requirements.

#### Step 7: (Optional) Disable the Leap Repo

```bash
sudo zypper mr -d openSUSE-Leap-15_3-OSS
```

> Keeps your VM stable and offline-friendly.

#### Step 8: Register GCC 11 with `update-alternatives`

```bash
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 110 \
  --slave /usr/bin/g++ g++ /usr/bin/g++-11 \
  --slave /usr/bin/cpp cpp /usr/bin/cpp-11 \
  --slave /usr/bin/cc  cc  /usr/bin/gcc-11
```

> This sets `/usr/bin/gcc` and friends to point to version 11.3.

#### Step 9: Smoke-Test the Installation

```bash
echo 'int main(){return 0;}' > test.c
gcc -Wall -O2 test.c -o test && ./test && echo "GCC OK"
```

> Should print `GCC OK` if everything is working.

#### Result

* gcc --version and g++ --version → 11.3.0
* Compilation, assembly, linking and execution succeed.
* All work was done without SUSE registration; only public openSUSE mirrors were used.

We now have a clean, offline‑capable GCC 11.3 environment on SLES 15 SP3.

![gcc and g++ version 11.3.0 installed](/assets/gcc-11-3-0-installed.png)

### Excurse: Procedure to install gcc 11 on a registered SLES

#### Step 1:  Enable (or check) the Development Tools module

    sudo SUSEConnect -p sle-module-development-tools/15.3/x86_64
    
> If the module is already registered, SUSEConnect prints “Product already registered” and does nothing 

#### Step 2:  Refresh repositories

    sudo zypper ref
    
#### Step 3:  Install the GCC 11 tool‑chain

    sudo zypper install gcc11 gcc11-c++

> Those two packages pull in the runtime (`libgcc_s1`) and the matching C++ headers automatically 

#### Optional extras

    sudo zypper install gcc11-info gcc11-locale libstdc++6-devel-gcc11

* _gcc11‑info / ‑locale_: manuals and translated messages.
* _libstdc++6‑devel‑gcc11_: C++ standard‑library headers for building C++17/20 code (name differs from openSUSE) 

#### Step 4:  Make GCC 11 the default compiler (system‑wide)

````
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 110 \
                         --slave   /usr/bin/g++ g++ /usr/bin/g++-11 \
                         --slave   /usr/bin/cpp cpp /usr/bin/cpp-11 \
                         --slave   /usr/bin/cc  cc  /usr/bin/gcc-11
````

> The single master/slave call keeps _gcc, g++, cpp_ and _cc_ in sync. To switch back later: `sudo update‑alternatives --config gcc`.

#### Step 5:  Verify

````
gcc --version    # should print 11.3.0 or 11.5.x, depending on the current maintenance update
g++ --version

echo 'int main(){return 0;}' > test.c
gcc test.c -o test && ./test && echo "✔ tool‑chain OK"
````

##### Why the versions you see might be 11.5.x instead of 11.3.0

SUSE ships GCC 11.5 maintenance updates (same ABI) in the Development Tools module, so a fully patched SP3/SP4 box will install 11.5.0 by default.
If you must stay on 11.3 exactly, lock the package version:

    sudo zypper addlock gcc11 gcc11-c++ libstdc++6-devel-gcc11


[5]: https://en.wikipedia.org/wiki/Application_binary_interface

---

[<< Previous Chapter](chapter-3-install-afl-sdk.md) | [Content Table](README.md) | [Next Chapter >>](chapter-5-prepare-and-build-first-afl.md)