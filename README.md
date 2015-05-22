Exoflex Repo Manifests for the Yocto Project Build System
=============================================
This repository provides Repo manifests to setup the Yocto build system for 
supported the Exoflex using Gumstix Overo.

The Yocto Project allows the creation of custom linux distributions for embedded
systems, including Gumstix-based systems.  It is a collection of git
repositories known as *layers* each of which provides *recipes* to build
software packages as well as configuration information.

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
   
To learn more about repo, look at http://source.android.com/source/version-control.html 

**3.  Fetch all the repositories:**

    $ repo sync

Now go turn on the coffee machine as this may take 20 minutes depending on
your connection.

**4.  Initialize the Yocto Project Build Environment.**

    $ export TEMPLATECONF=meta-exoflex/conf 
    $ source ./poky/oe-init-build-env

This copies default configuration information into the **poky/build/conf**
directory and sets up some environment variables for the build system.

**5.  Build an image:**

This process downloads several gigabytes of source code and then proceeds to
do an awful lot of compilation so make sure you have plenty of space (25GB
minimum), and expect a day or so of build time depending on your network
connection.  Don't worry---it is just the first build that takes a while.

    $ bitbake gumstix-console-image

If everything goes well, you should have a compressed root filesystem
tarball as well as kernel and bootloader binaries available in your
**tmp/deploy/images/{ overo | duovero | pepper }** directory.  If you run into problems, the most likely
candidate is missing software packages.  Check out
http://www.yoctoproject.org/docs/current/yocto-project-qs/yocto-project-qs.html#resources
for the list of required packages for operating system. Also, take
a look to be sure your operating system is supported:
https://wiki.yoctoproject.org/wiki/Distribution_Support


**6. Create a bootable micro SD card:**

You are one step closer to booting your Gumstix with the new image you built! 
First you have to create two partitions: `boot` and `rootfs`. We have included 
a small script to help you out with it. Change your directory to 
**poky/meta-gumstix-extras/scripts** and you should see a shell script named mk2partsd.
Pop in your micro SD card to your card writer, and find out the location of 
the block device by running `dmesg`. Now you can run the script as following:

    $ sudo ./mk2partsd <block device> 
    
If you get an error, make sure your operating system has not automatically mounted 
the drives. 

Once this is successful, go ahead and mount both the drives. 

**7. Flash your image:**

Almost there. Now you just have to populate the card with the image you built. 
Go to the deploy directory as mentioned in step 5:

    $ cd build/tmp/deploy/images/overo
    
Write the bootloader, kernel and the root file system into your card:

    $ cp MLO u-boot.img /media/boot
    $ sudo tar xaf gumstix-console-image-overo.tar.bz2 -C /media/rootfs --strip-components=1

Note that in Gumstix Yocto Project 1.7, you no longer have to copy the kernel image into `/media/boot`.
A kernel image (zImage) is included in the root file system.

And you should make sure all the files are written:

    $ sync

Hooray you are done!

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

    $ bitbake gumstix-console-image

