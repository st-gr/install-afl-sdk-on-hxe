---
title: Install AFL SDK on HANA Express
author: st-gr
date: 5/6/2025
mainfont: Helvetica, Arial, sans-serif
fontsize: 18px
---

Install AFL SDK on HANA Express
===============================

---
***Guide to install the SAP Application Function Library SDK on SAP HANA Express***

**Author:** *st-gr*

# Chapter One: Motivation
Creating AFL plugins enables you to enhance SAP HANA functionality beyond the standard deployed functional scope. For example, there is ***no native Base64 encoding and decoding*** available on HANA, and SAP is currently not considering this for the near future, as seen in [Provide Base64 decode/encode as native database function][1]. With Cloud and SaaS deployments, vendors like to lock you into their walled garden, making extensibility even more important. Having such low-level extensibility capabilities is detrimental to the goal of having a stable cloud environment where the same service/server is shared among many customers. However, this should not apply when one operates dedicated instances such as S/4 HANA private cloud.

Anything that can be compiled in C/C++ within the AFL constraints can be implemented. Think big!

[1]: https://influence.sap.com/sap/ino/#/idea/295931

---

# Chapter Two: Preparations

This guide requires you to have the following software ready.

## HANA Express 2.0

You need to have HANA Express 2.0 installed and running on your virtualization software such as VMWare, VirtualBox, and others.
[SAP HANA trial][2] 

The standard HANA Express 2.0 *Open Virtual Appliance (.ova) container* can't be deployed on Hyper-V. If you want to use Hyper-V then you can follow my guide [SAP HANA Express 2.0 on Hyper-V][3].
I use the fully-loaded version with database + XS Advanced applications in this guide. The database only version works as well, but you won't have a WebIDE to play with.

## AFL SDK 2.0

You need to have an authorized user download software from SAP.
Since HANA Express has HDB AFL pre-installed you don't need to download `HDB_AFL_LINUX_X86_64`, but you need the HANA AFL SDK from SAP.

1. Navigate to [me.sap.com][4] and click the tile **Download Software**

2. Type in the search term `SAP HANA AFL SDK 2.0`

3. Find the `SAP HANA AFL SDK 2.0 (SUPPORT PACKAGES AND PATCHES)`

4. Choose the system architecture, for HANA Express 2.0, this is `LINUX ON X86_64 64BIT`.

5. Download the latest version of the **SAP HANA AFL Software Development Kit 2.0**
   This is currently `IMDBAFLSDK20_034_0-80002169.SAR`, release date November, 23 2018

![Download AFL SDK](/assets/me-sap-com-download-afl-sdk.png)

6. You need the SAPCAR tool to extract the archive contents, here run on Windows within a PowerShell 5.1:

````
PS> .\SAPCAR.EXE -xvf IMDBAFLSDK20_034_0-80002169.SAR
````
output:
````
SAPCAR: processing archive IMDBAFLSDK20_034_0-80002169.SAR (version 2.01)
x AFL_Developer_Manual_HANA2_SPS03.pdf
x AFL_SDK_Guide_HANA2_SPS03.pdf
x SAP_HANA_AFL_SDK_2_13_0.tgz
SAPCAR: 3 file(s) extracted
````

7. Read the manuals  
   Now it is time to read the SDK Guide and Developer Manual as I assume from now on that you have read those documents.
   
   If you've got scared (risk of destabilizing HDB, no support = the usual, etc.) of the risks involved then you should refrain from leveraging AFL altogether.
   
8. Read OSS notes

* 2190908 - Customer use of the Application Function Library SDK for SAP HANA
* 2046767 - Creating, dropping, and executing AFLLANG wrapper procedures

[2]: https://www.sap.com/products/data-cloud/hana/express-trial.html
[3]: https://github.com/st-gr/deploy-hxe-on-hyper-v/blob/main/README.md
[4]: https://me.sap.com/servicessupport

---

## Chapter Three: Install AFL SDK

The SDK archive `SAP_HANA_AFL_SDK_2_13_0.tgz` we extracted earlier from the .SAR file needs to be uploaded to the HANA Express Appliance and then extracted and made public with an environment variable.

After that we are determining and installing the proper GCC compiler version that will let us compile AFL functions.

### Upload and install SDK archive
Connect to your booted HANA Express 2.0 virtual machine. I am using putty to `hxehost.localdomain`.

Login with the `hxeadm` user and the password you defined at initial boot of the HXE appliance. I use Putty.

**1. Determine home folder** of `hxeadm` and get its path

````
hxeadm@hxehost:/usr/sap/HXE/HDB90> cd ~
hxeadm@hxehost:/usr/sap/HXE/home> pwd
/usr/sap/HXE/home
hxeadm@hxehost:/usr/sap/HXE/home> ls
bin  Downloads  root_key.bck
````

**2. Create a folder for the SDK** in the users home

````
mkdir ~/AFL_SDK_2_13_0
````

**3. Copy the SDK archive to HANA Express 2.0**  

I am using WinSCP to copy the file from my Windows host to the HXE guest VM.

![Download AFL SDK](/assets/winscp-upload-sdk-archive.png)

**4. Extract the SDK archive on HXE**  

Assuming you are still logged in to the terminal.

````
cd ~/AFL_SDK_2_13_0/
tar -xzvf SAP_HANA_AFL_SDK_2_13_0.tgz
````
Validate SDK folder contents
````
hxeadm@hxehost:/usr/sap/HXE/home/AFL_SDK_2_13_0> ls -ltra
total 82252
-rw-r-----  1 hxeadm sapsys     5563 Oct 16  2018 README
drwxr-x---  3 hxeadm sapsys       18 Oct 16  2018 include
drwxr-x---  2 hxeadm sapsys       51 Oct 16  2018 libs
drwxr-x---  9 hxeadm sapsys      146 Oct 16  2018 .
drwxr-x---  3 hxeadm sapsys       19 Oct 16  2018 demos
drwxr-x---  2 hxeadm sapsys      158 Oct 16  2018 cust
drwxr-x---  2 hxeadm sapsys     8192 Oct 16  2018 docs
drwxr-x---  4 hxeadm sapsys       56 Oct 16  2018 LCMsdk
drwxr-x---  3 hxeadm sapsys      125 Oct 16  2018 tools
-rw-r-----  1 hxeadm sapsys 84198678 Nov 20  2018 SAP_HANA_AFL_SDK_2_13_0.tgz
````

You can delete the .tgz file

    rm SAP_HANA_AFL_SDK_2_13_0.tgz

**5. Read the `README` file**

    less README  # press space to page and q to exit

**6. Permanently configure the variables to the SDK folder**

1. Open the .bashrc file:  
  `vi ~/.bashrc`
  
2. Add the environment variable:  

    * Press i to enter vi insert mode.
    * Scroll to the end of the file and add the following lines:

````
export HANA_SDK_PATH=/usr/sap/HXE/home/AFL_SDK_2_13_0
export PATH=$HANA_SDK_PATH/tools:$PATH
````

3. Save and close the file:

    * Press Esc to exit insert mode
    * Type `:wq` and press Enter to save the changes and exit `vi`.

4. Apply the changes:

    * To apply the changes immediately, run:  
      `source ~/.bashrc`

5. Validate using `which` where we expect the path to the executable:

````
# which aflconfigure
/usr/sap/HXE/home/AFL_SDK_2_13_0/tools/aflconfigure
````

We have permanently set the `HANA_SDK_PATH` environment variable and added the SDK tools directory to the `PATH`. This will ensure that the environment variables are set every time a new terminal session is started.

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

> These four RPMs include the 11.3 compiler driver, C++ front end, standard library headers, and runtime.
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

# Chapter Five: Prepare and build first AFL

## Prepare

Instead of interacting with the HDB using hdbsql we would like to make use of the WebIDE that comes with the full version of the HANA Express 2.0 Appliance.

### 1. Create and configure the HDB user

    hdbsql -i 90 -d HXE -u SYSTEM 'CREATE USER ST-GR PASSWORD "Time4Coffee&Cake" NO FORCE_FIRST_PASSWORD_CHANGE;'
    
#### 1.1 Grant general DB access

````
hdbsql -i 90 -d HXE -u SYSTEM -p <P*a*S*s*W*o*R*d> -m <<EOF
-- Assign built-in roles
GRANT CONTENT_ADMIN TO ST-GR;
GRANT MODELING       TO ST-GR;
GRANT MONITORING     TO ST-GR;
EOF
````

### 2. Create and configure the XSA User

Get the URL of the xsa-cockpit service:

    xs apps | grep xsa-cockpit | awk '{print $NF}'

#### 2.1 Access the XSA Cockpit

Open a browser like Firefox and navigate to the URL from above: `https://hxehost.localdomain:51035`

#### 2.2 Log In with Administrator credentials

* Use the built-in `XSA_ADMIN` account, or your `SYSTEM` user if you’ve granted it equivalent XSA rights

![First Log In to XSA Cockpit](/assets/xsa-cockpit-first-logon.png)

#### 2.3 Create a New User

1. In the left menu, select **User Management > New User**.
2. Enter the **User Name, Email,** and **Password**, then click **Create**

![New user](/assets/xsa-new-user.png)

#### 2.4 Assign XSA Role Collections

1. In **User Management**, locate your new user and click the **Actions** column > **Assign Role Collections**.

2. Click **Add** and search for the following role collections:

    * **XS_CONTROLLER_USER**: Allows interaction with orgs and spaces.

    * **XS_AUTHORIZATION_ADMIN**: Grants access to the XSA administration tools.

    * **DEVX_DEVELOPER**: Enables development of XSA-native apps.

3. Click **OK**, then **Save** to apply the roles 

### 3. Grant Access to SAP Web IDE

#### 3.1 Locate the Web IDE URL

By default, the SAP Web IDE is deployed as an XSA application on port 53075. Run xs apps on the server to verify the exact URL:

    xs apps | grep webide | awk '{print $NF}'

In an incognito browser window navigate to:

    https://hxehost.localdomain:53075
    
Change initial password of new user and wait for WebIDE to load.

#### 3.2 Assign the WebIDE Role Collection

1. In the XSA Cockpit, go to **Security** > **Role Collections**.

2. Find or create a collection for Web IDE users (often named **WebIDE_Developer**).

3. In that collection, include the role template `webide!i1`, Role Template `WebIDE_Developer` to enable all Web IDE functionality 

4. Assign this collection to your user via User Management > Assign Role Collections, then Save.

### 4. Add the ScriptServer

HANA AFL need the scripserver to run. Let us add it to the HXE database:

    hdbsql -i 90 -d SystemDB -u SYSTEM "ALTER DATABASE HXE ADD 'scriptserver';"

## Compile and deploy demo intro AFL

Because each SAP source file carries a strict ‘all‑rights‑reserved’ notice, I cannot reproduce the code here verbatim. Under fair‑use principles, however, I may cite brief excerpts for educational commentary. 

Personally, I believe demo projects that accompany an SDK should be published under an open‑source license. 

It may simply be that the author is reserving the material for a more formal venue - perhaps even a future SAP PRESS title - and is therefore reluctant to release the code publicly at this stage.

The instructions in the `$HANA_SDK_PATH/README` file have some gotchas that we need to be aware of. For our purposes, we'll focus on the demo compilation and deployment.

First, navigate to the AFL project directory `cd $HANA_SDK_PATH/demos/intro`.

The general workflow involves these main steps, detailed further in the SDK's README:

1.  **Project Setup**: Initialize your build environment with `aflconfigure <AFL_NAME>`. For the demo, this is `aflconfigure intro`. This creates a reference makefile.

````
hxeadm@hxehost:/usr/sap/HXE/home/AFL_SDK_2_13_0/demos/intro> aflconfigure intro
Checking required files... OK!
Validating AFX... OK!
Generating supporting files... OK!
Creating Makefile for release configuration (-O2)... OK!

See "Makefile" for build settings. Currently set to release without debug symbols!
Run "make" to (re)build your AFL!
Run "make installer" to build an installation package for your AFL!
````

This created the following files:

````
> find . -type f -mtime 0
./afl_intro.h
./afl_introFactory.h
./afl_introIspc.cpp
./afl_introWrapper.h
./Makefile
````

2.  **Compilation**: Build the shared library using `make`.

````
hxeadm@hxehost:/usr/sap/HXE/home/AFL_SDK_2_13_0/demos/intro> make
g++ -O2 -std=c++11 -Wall -fPIC -fvisibility=hidden -Wno-strict-aliasing -I/usr/sap/HXE/home/AFL_SDK_2_13_0/include -c afl_introIspc.cpp -o afl_introIspc.o
g++ -O2 -std=c++11 -Wall -fPIC -fvisibility=hidden -Wno-strict-aliasing -I/usr/sap/HXE/home/AFL_SDK_2_13_0/include -c eval.cpp -o eval.o
g++ -O2 -std=c++11 -Wall -fPIC -fvisibility=hidden -Wno-strict-aliasing -I/usr/sap/HXE/home/AFL_SDK_2_13_0/include -c intro.cpp -o intro.o
g++ -o libaflintro.so afl_introIspc.o eval.o intro.o -shared -lhdbaflsdk -Wl,-no-undefined -L/usr/sap/HXE/home/AFL_SDK_2_13_0/libs
Run "make installer" to build an installation package for your AFL!
````

3.  **Package Creation**: Generate the installation package with `make installer`. This might ask for details on its first run and will output an `<installer directory>`.

When we execute it we get an error:

````
hxeadm@hxehost:/usr/sap/HXE/home/AFL_SDK_2_13_0/demos/intro> make installer
Creating installation package...
# : skipped some lines here
Creating manifest...   File "<string>", line 1
    import os.path; print os.path.relpath('/usr/sap/HXE/home/AFL_SDK_2_13_0/demos/intro/pack/manifest','/usr/sap/HXE/home/AFL_SDK_2_13_0/demos/intro')
                          ^
# : skipped some more lines
OK!
Creating script.pm... OK!
Creating installation package... SAP HANA Installer Runtime Error:
perl runtime error (calling symbol "BuildPackage::BuildPackage::main"):
Could not access manifest '/usr/sap/HXE/home/AFL_SDK_2_13_0/demos/intro/' : Invalid argument

ERROR: Creation of installation package failed
````

The error occurs because the `$HANA_SDK_PATH/tools/aflpack` script uses Python 2's print statement syntax (print os.path.relpath(...)) which is a SyntaxError in Python 3, where print is a function (print(...)). The Python code is executed within the copyWithReplacement function.

When the AFL SDK was created in 2018 the SLES release 11 contained Python 2. This and the fact that there is no newer release of the SDK makes me wonder if this technology will be supported in the future.

We could either install Python 2 and set the environment variable `export PYTHON=python2` before `make install` or patch the `aflpack` script.

Let's commit a cardinal sin and patch it:

````
sed -i.bak '/^[[:space:]]*MANIFEST_FILE=`\${PYTHON:-python} -c "/s#import os\.path; print \(os\.path\.relpath('\''${OUTPUT_DIR}/manifest'\'','\''$INPUT_DIR'\'')\)#from __future__ import print_function; import os.path; print(\1)#' $HANA_SDK_PATH/tools/aflpack
````

Let us repeat the command and we are back in business:

````
hxeadm@hxehost:/usr/sap/HXE/home/AFL_SDK_2_13_0/demos/intro> make installer
Creating installation package...
/usr/sap/HXE/home/AFL_SDK_2_13_0/tools/aflpack --read-from installer.properties --query-missing --write-to installer.properties
AFL:              intro
Vendor:           sap
Version:          1.0.0.0
Description:      AFL SDK introduction example
Input Directory:  /usr/sap/HXE/home/AFL_SDK_2_13_0/demos/intro
Output Directory: /usr/sap/HXE/home/AFL_SDK_2_13_0/demos/intro/pack

Checking AFL name (starting with lower case, only letters and underscores)... OK!
Checking description (starting with a letter and at least two characters long)... OK!
Checking version format (MAJOR.MINOR.MICRO.PATCH)... OK!
Checking required files... OK!
Creating manifest... OK!
Creating allpackages.xml... OK!
Creating InstallParams.xml... OK!
Creating script.pm... OK!
Creating installation package... OK!

Installation settings are stored in installer.properties
Installation package is located at /usr/sap/HXE/home/AFL_SDK_2_13_0/demos/intro/pack/installer
````

**Deploying to HANA**:
Before running `hdbinst` from the `<installer directory>/installer` (as `root`):
*   Ensure the **HANA database is operational**.  
    `hdbsql -i 90 -d SystemDB -u SYSTEM "SELECT CURRENT_TIMESTAMP FROM DUMMY;"`

*   Confirm the **scriptserver is enabled** (as covered previously).  
    Otherwise you will see something like this:  
    `* 2048: column store error: search table error:  [34091] No ScriptServer available (see SAPNote 1650957) SQLSTATE: HY000`
*   Be prepared for `hdbinst` to **restart the HANA system**.
*   The HANA system revision should be equal to or higher than the SDK's revision.

**Versioning for Updates**:
To update an existing AFL, you must modify its version in the `installer.properties` file. The HANA installer checks `AFL_MAJOR_VERSION` and `AFL_MINOR_VERSION`. For a successful update, the new version must be greater (e.g., increment the minor version if the major version is unchanged). Failure to do so will prevent `hdbinst` from applying the update. Refer to the README for specifics on version hierarchy.

Our user hxeadm has root privileges. That is why we didn't have to switch our login to root or use `sudo`.
````
# cd /usr/sap/HXE/home/AFL_SDK_2_13_0/demos/intro/pack/installer
hxeadm@hxehost:/usr/sap/HXE/home/AFL_SDK_2_13_0/demos/intro/pack/installer> ./hdbinst
AFL SDK introduction example installation kit detected.


SAP HANA Lifecycle Management - AFL SDK introduction example Installation 1.0.0.0.0
***********************************************************************************



Checking installation...
Preparing package 'sap_afl_sdk_intro'...
Installing AFL SDK introduction example to /hana/shared/HXE/exe/linuxx86_64/plugins/sap_afl_sdk_intro_1.0.0.0.0_03e0ce9a1a5945584826e8a30d59844c334c5c5c...
Installing package 'sap_afl_sdk_intro'...
Stopping system...
  Stopping 9 processes on host 'hxehost.localdomain' (worker, xs_worker):
    Stopping on 'hxehost.localdomain' (worker, xs_worker): hdbcompileserver, hdbdaemon, hdbdiserver, hdbindexserver, hdbnameserver, hdbscriptserver, hdbwebdispatcher, hdbxscontroller, hdbxsuaaserver
  All server processes stopped on host 'hxehost.localdomain' (worker, xs_worker).
Activating plugin...
Starting system...
Starting 9 processes on host 'hxehost.localdomain' (worker, xs_worker):
    Starting on 'hxehost.localdomain' (worker, xs_worker): hdbcompileserver, hdbdaemon, hdbdiserver, hdbindexserver, hdbnameserver, hdbscriptserver, hdbwebdispatcher, hdbxscontroller, hdbxsuaaserver
    Starting on 'hxehost.localdomain' (worker, xs_worker): hdbcompileserver, hdbdaemon, hdbdiserver, hdbindexserver, hdbscriptserver, hdbwebdispatcher, hdbxscontroller, hdbxsuaaserver
    Starting on 'hxehost.localdomain' (worker, xs_worker): hdbdaemon, hdbdiserver, hdbindexserver, hdbscriptserver, hdbwebdispatcher, hdbxscontroller, hdbxsuaaserver
    Starting on 'hxehost.localdomain' (worker, xs_worker): hdbdaemon, hdbdiserver, hdbwebdispatcher, hdbxscontroller, hdbxsuaaserver
    Starting on 'hxehost.localdomain' (worker, xs_worker): hdbdaemon, hdbxscontroller, hdbxsuaaserver
    Starting on 'hxehost.localdomain' (worker, xs_worker): hdbdaemon, hdbxscontroller
  All server processes started on host 'hxehost.localdomain' (worker, xs_worker).
Installation done
Log file written to '/var/tmp/hdb_sap_afl_sdk_intro_2025-05-08_15.14.43_2556/hdbinst_sap_afl_sdk_intro.log' on host 'hxehost'.
````

### Dev notes

On my machine this takes about 18-20 minutes to complete a deployment. Therefore testing new functionality becomes time intensive. I would strongly advise to test your code locally with mock data and only when the local tests succeeded move on to deployment and actual tests on your HANA DB. There is no debugger for Llang functions.

There are trace files you can check:

````
# cd /hana/shared/HXE/HDB90/hxehost.localdomain/trace/DB_HXE
# ls -1 scriptserver*.trc
scriptserver_alert_hxehost.localdomain.trc
scriptserver_hxehost.localdomain.39040.000.trc
scriptserver_hxehost.localdomain.39043.000.trc
````
Finding exceptions:
````
# cat scriptserver_hxehost.localdomain.39040.000.trc | grep exception
[25704]{201494}[1/215124] 2025-05-08 17:08:42.786374 e ceExecute        cePopCustomLjit.cpp(00605) : _SYS_AFL.INTRO_AREA:DEMONSTRATE_NON_FATAL_EXCEPTION: [423] (range 3) AFL error exception: call canceled
````

You must be an experienced C++ developer who knows how to use tools like Valgrind and is aware of pitfalls of low level user space programming.

## Test the deployed intro AFL

Follow the instructions in the `$HANA_SDK_PATH/demos/intro/invoke_intro.sql` file.

Again, I can't run you through the steps here as also this file has a strict copyright notice on top.

I advise you to study the file closely as it demonstrates many concepts when interacting with the AFL SDK.

Here is how to execute it:

    # change to the folder, if not done so already
    cd $HANA_SDK_PATH/demos/intro
    
    # Execute the tests provided with the intro AFL
    hdbsql -i 90 -d HXE -u SYSTEM -p <P*a*S*s*W*o*R*d> -I invoke_intro.sql

You may see error messages like *invalid user name .., AFL error: ...* These are normal as the test script cleans up prior artifacts before each run. If you haven't run tests before, you'll see errors. Some errors are intentional, triggered by SQL exceptions within the AFL code for testing purposes.

However, I will show how to use the SQL console in WebIDE with the *ST-GR* user we created earlier.

Only now that the intro AFL plugin was installed, we can make use of its roles.

Let us grant our user *ST-GR* to execute the intro AFL:

    # Enter SQL console
    hdbsql -i 90 -d HXE -u SYSTEM -p <P*a*S*s*W*o*R*d>
    -- authorize
    GRANT AFL__SYS_AFL_INTRO_AREA_EXECUTE TO ST-GR;
    
Also, the authorization to create or erase AFL parameters (optional):

    GRANT AFLPM_CREATOR_ERASER_EXECUTE TO ST-GR;

Logon to WebIDE using the *ST-GR* user account:

    https://hxehost.localdomain:51035/

Add a database connection to our HANA Express 2.0 Appliance on `hxehost.localdomain`.

![Adding a DB connection in WebIDE](/assets/webide-db-connection.png)

Open a SQL console.

![Open SQL console to the connected DB](/assets/webide-open-sql-console.png)

Execute test for the string concatenation intro AFL function.

![Test the intro AFL string concat function](/assets/webide-sql-test-intro-afl-string-concat.png)


---

## License

This documentation is licensed under a [Creative Commons Attribution 4.0 International License](LICENSE).

---