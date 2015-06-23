Exoflex Repo Manifests for Yocto
================================
This repository provides Repo manifests to setup the Yocto build system for 
building Exoflex (Gumstix Overo) images.

It follows the basic procedure recommend by Gumstix so that the Exoflex
images can be kept current.

Repo is a tool that enables the management of many git repositories given a 
single *manifest* file.  Tell repo to fetch a manifest from this repository and
it will fetch the git repositories specified in the manifest and, by doing so,
setup a Yocto Project build environment for you!

Getting Started
---------------
**1.  Install Repo.**

Download the Repo script:

    $ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > repo

Make it executable:

    $ chmod a+x repo

Move it on to your system path:

    $ sudo mv repo /usr/local/bin/

If it is correctly installed, you should see a Usage message when invoked
with the help flag.

    $ repo --help

**2.  Initialize a Repo client.**

Create an empty directory to hold your working files:

    $ mkdir yocto
    $ cd yocto

Tell Repo where to find the manifest:

    $ repo init -u git://github.com/chrisw957/yocto-manifest.git 

A successful initialization will end with a message stating that Repo is
initialized in your working directory. Your directory should now
contain a .repo directory where repo control files such as the manifest are
stored but you should not need to touch this directory.
   
**3.  Fetch all the repositories:**

    $ repo sync

Now go turn on the coffee machine as this may take 20 minutes depending on
your connection.

**4.  Initialize the Yocto Project Build Environment.**

    $ export TEMPLATECONF=meta-exoflex/conf 
    $ source ./poky/oe-init-build-env

This copies default configuration information into the **poky/build/conf**
directory and sets up some environment variables for the build system.

**5.  Build a console image:**

This process downloads several gigabytes of source code and then proceeds to
do an awful lot of compilation so make sure you have plenty of space (25GB
minimum), and expect a day or so of build time depending on your network
connection.  Don't worry---it is just the first build that takes a while.

    $ bitbake exoflex-console-image

If everything goes well, you should have a compressed root filesystem
tarball as well as kernel and bootloader binaries available in your
**tmp/deploy/images/overo** directory.  If you run into problems, the most likely
candidate is missing software packages.  Check out
http://www.yoctoproject.org/docs/current/yocto-project-qs/yocto-project-qs.html#resources
for the list of required packages for operating system. Also, take
a look to be sure your operating system is supported:
https://wiki.yoctoproject.org/wiki/Distribution_Support

**6.  Build the Exofile image:**

This is the graphical image which actually runs the exoflex appliction.  It is
considerably more complex, and takes longer to build.  We call it di-image because
that what is was called in the older repository.

    $ bitbake di-image

**7. Create a bootable micro SD card:**

Pop in your micro SD card to your card writer, and find out the location of 
the block device by running `dmesg`. Now you can run the script below,
specifying the block device (such as /dev/sdc) and the image name
(such as exoflex-console-image-overo)

    $ cd ~/yocto/poky/meta-exoflex/scripts
    $ sudo ./mkcard.sh <block device> di-image-overo 
    
Once this is successful, unmount/eject the card. 


**8. Erase old u-boot env settings**

Put the micro SD card into the exoflex, and power on the system.  Immediately
press a key to stop u-boot from auto-booting. 

Ease the current u-boot environment settings stored in Flash, then reset the
exoflex and let it boot normally.

     u-boot> nand erase 240000 20000
     u-boot> reset

Done!

Staying Up to Date
------------------
To pick up the latest changes for all source repositories, run:

    $ repo sync

Enter the Yocto Project build environment:

    $ source poky/oe-init-build-env

    If you forget to setup these environment variables prior to bitbaking, your 
    OS will complain that it can't find bitbake on the path.  Don't try to
    install bitbake using a package manager, just run the above command.

You can then rebuild as before:

    $ bitbake exoflex-console-image

About the Exoflex Console Image
-------------------------------
The exoflex-console-image is a basic image which includes the driver for the
kd035fm LCD display.  You should see "Tux" during boot, followed by the login
prompt on the LCD.  

Login from the serial console and run fb-test to verify the LCD is operating 
in the correct color depth.

