---
title: Install AFL SDK on HANA Express - Chapter 3
author: st-gr
date: 5/6/2025
mainfont: Helvetica, Arial, sans-serif
fontsize: 18px
---

Install AFL SDK on HANA Express
===============================

***Guide to install the SAP Application Function Library SDK on SAP HANA Express***

**Author:** *st-gr*

[<< Previous Chapter](chapter-2-preparations.md) | [Content Table](README.md) | [Next Chapter >>](chapter-4-determine-and-install-gcc-version.md)

---

## Chapter Three: Install AFL SDK

The SDK archive `SAP_HANA_AFL_SDK_2_13_0.tgz` we extracted earlier from the .SAR file needs to be uploaded to the HANA Express Appliance and then extracted and made public with an environment variable.

After that we are determining and installing the proper GCC compiler version that will let us compile AFL functions.

### Upload and install SDK archive
Connect to your booted HANA Express 2.0 virtual machine. I am using PuTTY to `hxehost.localdomain`.

Login with the `hxeadm` user and the password you defined at initial boot of the HXE appliance.

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

The following will make use of `vi` in steps 1 to 3. If you don't like to use `vi` then you could also use the following `sed` command to do the same:

````
sed -i '$a\export HANA_SDK_PATH=/usr/sap/HXE/home/AFL_SDK_2_13_0\nexport PATH=$HANA_SDK_PATH/tools:$PATH' ~/.bashrc
````

> This command:
> - `-i`: Edits the file in-place
> - `$a`: Appends the text after the last line of the file
> - `\n`: Creates a new line between the two export statements
> - `~/.bashrc`: Targets the .bashrc file in the user's home directory

If you used the `sed` alternative then skip to step 4.

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

[<< Previous Chapter](chapter-2-preparations.md) | [Content Table](README.md) | [Next Chapter >>](chapter-4-determine-and-install-gcc-version.md)