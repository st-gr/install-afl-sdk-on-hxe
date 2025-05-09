---
title: Install AFL SDK on HANA Express - Chapter 5
author: st-gr
date: 5/6/2025
mainfont: Helvetica, Arial, sans-serif
fontsize: 18px
---

Install AFL SDK on HANA Express
===============================

***Guide to install the SAP Application Function Library SDK on SAP HANA Express***

**Author:** *st-gr*

[<< Previous Chapter](chapter-4-determine-and-install-gcc-version.md) | [Content Table](README.md)

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

[<< Previous Chapter](chapter-4-determine-and-install-gcc-version.md) | [Content Table](README.md)