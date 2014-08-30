Android ADB File Manager (aafm)
====

An Android ADB-based file manager [created](https://github.com/sole/aafm) by [Soledad Penadés](http://soledadpenades.com).  It is continued here because development has ceased on the original repository.

![Screenshot](http://i.imgur.com/g8CJTEu.png)

**aafm** uses ADB (one of the command line tools provided with the official Android SDK) to facilitate easy file transfers to/from any Android device.

## Installing ##

#### Debian/Ubuntu ####

Execute the following in a terminal:

```
sudo add-apt-repository ppa:tslocum/aafm
sudo apt-get update
sudo apt-get install aafm
```

**Note:** If you run into any problems, please submit an issue including a log of apt's output.

#### Mac ####

First, download and install [pyGTK for Mac](http://sourceforge.net/projects/macpkg/files/PyGTK/2.24.0/PyGTK.pkg/download).  Then execute the following in a terminal:

```
git clone git://github.com/tslocum/aafm.git ~/Applications/aafm
cd ~/Applications/aafm
python setup.py install
sudo cp -R aafm.app /Applications/
```

**Note:** Unlike the Debian/Ubuntu package, if you don't have ADB installed yet, you'll need to install it yourself.  See the **Install the Android SDK** instructions below.

#### Windows ####

Pre-built executables will not be available within the near future.  For now, download the source package and execute **aafm-gui.py** directly from the **src** folder.  Note that you will need to install Python, GTK and the pyGTK module if you don't have them already.

## Building ##

### Requirements ###

Python with PyGTK bindings, GTK, git, and the Android SDK.

Getting these should be fairly straightforward if you're running any decent Linux distribution. If you're using Windows I believe there are next-next-next installers for everything. This leaves us with the third option which is Mac OS. Up until recently there wasn't an easy way to get any PyGTK software working in Mac OS, but turns out you can now download a binary build of PyGTK for Mac OS with which aafm works quite well!

It can be downloaded from http://afb.users.sourceforge.net/zero-install/PyGTK.pkg ([more info](http://python.6.n6.nabble.com/Testing-PyGTK-installer-for-Mac-OS-X-td1948270.html)). Simply run the installer once downloaded, and then follow the instructions below as if you were running a proper Linux system.

One note though: there are a few hiccups with the Mac version, but they are mostly cosmetic and won't prevent you from enjoying the software. Feel free to help correcting them if you have the know-how!

### Clone repository ###

Clone this repository to a place you fancy. For example, your Applications folder.

```git clone git://github.com/tslocum/aafm.git ~/Applications/aafm```

### Install the Android SDK ###

If it's not installed yet, download the SDK from its page and follow its instructions: http://developer.android.com/sdk/index.html

Basically (at least in Linux) just download a zip file and unpack it to a known location. In my case it's ```~/Applications/android-sdk-linux_86```. Once that is done, you need to make sure that the ADB tool is readily accessible from a shell (which is what **aafm** uses internally).

So to try that out, open a new terminal and type ```adb```. If it works, you should get a long help message that starts with something like ```Android Debug Bridge version 1.0.26```. If it doesn't work, you'll get something akin to ```adb: command not found```.

In case it doesn't work, you need to add the path to ADB to the environment PATH variable. In Linux this is done by editing a file called ```.bashrc``` in your home folder (```.bash_profile``` in Mac OS). Locate a line that looks like ```PATH=$PATH``` and make it look like this:

    PATH=$PATH:~/Applications/android-sdk-linux_86/platform-tools

The line above appends ADB's path to whatever value $PATH held before. The path might be different, according to wherever you've installed the Android SDK.

For more information on ADB and a list of its features, read over here: http://developer.android.com/guide/developing/tools/adb.html

Also, I haven't tried it myself, but it seems that it's possible to download and build a reduced subset of the Android SDK only, including ADB and a few more tools. This doesn't require Java installed in the system. This page describes how: http://lackingrhoticity.blogspot.com/2010/02/how-to-build-adb-android-debugger.html

### Close terminal and open it again ###

So the changes to the PATH get current. In Mac OS you might need to log in and out too.

### Configure udev rules (if in Linux) ###

**Note:** This step may not be required.  Try using the program, and if you encounter a "Insufficient permissions for device" error, you will need to do the following.

You need to let the system know that when you connect your USB device (i.e. the tablet) it should allow you, as a non-root user, to access it.  This is done by adding a new file that contains so called udev rules.

For example, in Ubuntu 10.10 you would add a file in ```/etc/udev/rules.d/51-android.rules``` with the following content:

    # Samsung
    SUBSYSTEM=="usb", SYSFS{idVendor}=="04e8", MODE="0777"

You can find out the "idVendor" value by running lsusb in a terminal. That will output a list of the currently connected USB devices, such as for example this:

```
Bus 004 Device 006: ID 05ac:8218 Apple, Inc. 
Bus 004 Device 003: ID 0a5c:4500 Broadcom Corp. BCM2046B1 USB 2.0 Hub (part of BCM2046 Bluetooth)
Bus 004 Device 002: ID 05ac:0237 Apple, Inc. Internal Keyboard/Trackpad (ISO)
Bus 004 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 003 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 004: ID 18d1:4e12 Google Inc. Nexus One Phone (Debug)
Bus 001 Device 002: ID 05ac:8507 Apple, Inc. 
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

Since I'd like to add support for browsing my Nexus One phone (because *aafm* works with every Android device that adb can connect to), I just need to look at the device with id *18d1:4e12*, and add the following line to the udev rules file:

```
SUBSYSTEM=="usb", SYSFS{idVendor}=="18d1", MODE="0666"
```

Save it, and then change the file permissions:

    chmod a+r /etc/udev/rules.d/51-android.rules

To make sure it worked, connect the device and try to run ```adb devices``` in a terminal. If it's working properly, you should see a list more or less like this:

    List of devices attached 
    4342354131444B534652	device

The numbers aren't important, the important bit is that you see ```device``` instead of ```??????```.

If it isn't, you might need to either disconnect the device and try again, or reload udev so that the rules are actually loaded. Try with this:

    sudo /etc/init.d/udev restart

If everything else fails, try to log in and out again, or maybe even restart the system.

More information on udev rules and Android can be found on the official Android development guide: http://developer.android.com/guide/developing/device.html

### Enable Debug mode in the device ###

Go to _Settings → Applications → Development_ and make sure the USB debugging checkbox is ticked. You might get a scary warning saying debugging allows you to do nasty things--just ignore it.

### Execute aafm ###

To execute it, cd to the place where it's been cloned:

    cd ~/Applications/aafm/src/

And simply execute it:

    ./aafm-gui.py

If for some odd reason it has lost the executable permission, you can add it:

    chmod +x ./aafm-gui.py

Or simply execute it using Python:

    python ./aafm-gui.py

Once you're satisfied it's working, you can also make a launcher or add it to your Gnome menu, of course!

## Using it ##

If everything works (and why shouldn't it?) you should get a window divided in two panels. The left side represents your host computer, and initially should show the files of the aafm directory, since you launched it from there. The right side represents your Android device's files --so it needs to be connected to the computer, and _USB debugging_ must be enabled in the device.
You can navigate just as you would do with your favourite file explorer. Files can be dragged from one to another panel, directories created, and files renamed (hint: right click and explore the options the contextual menu offers you!). You can also drag from Nautilus (in GNOME) into the device panel, to copy files to the device, or drag _to_ Nautilus, for copying files from the device.

Be warned that currently the progress reporting is a bit hackish and with large files it will appear as if the window has got frozen. It hasn't--it's just waiting for the ADB transfer to finish. In the future this should be fixed, but I haven't come up with the best solution yet.

## License ##

Original code copyright (C) 2011-2012 Soledad Penades (http://soledadpenades.com).

This software is licensed under a GPL V3 license. Please read the accompanying LICENSE.md file for more details.

## Attributions ##

- Written by [Sole](http://soledadpenades.com)
- Nice usability ideas from [Mr.doob](http://mrdoob.com/)
- [FamFamFam icons](http://www.famfamfam.com/lab/icons/)
- XDS with PyGTK [tutorial](http://rodney.id.au/dev/gnome/an-xds-example)
- Issues and patches from [Toby Collett](https://github.com/thjc), [Peter Sinnott](https://github.com/psinnott) and [Alexalex89](https://github.com/Alexalex89).
- Development continued by [tslocum](https://github.com/tslocum)

## Hacking ##

I'm by no means a GTK/Python/ADB/Android expert. I'm just learning so this project will surely contain many things that can be improved or that are plain wrong, so feel free to clone the repository and submit pull requests :-)

In order to make your life a bit easier I'll roughly show what each file does:

* **Aafm.py** - a class that communicates with an Android device, using ADB via shell commands. Takes care of copying and reading files, listing and parsing directories, etc.
* **aafm-gui.py** - this is the GTK front-end. Takes care of building the window with the host and device panels, and issuing instructions to Aafm when the user requests something to be done.
* **TreeViewFile.py** - a utility class that encapsulates a GTKTreeView and some more things in order to show file listings.
* **MultiDragTreeView.py** - an awesome class developed by the guys of Quod Libet, that allows more than one element of a TreeView to be selected and dragged around.

As you can see, an **aafm-cli.py** GUI counterpart is missing. There was one at the beginning but I didn't redo it when I rewrote most of the code from scratch. Feel free to... you know what, if you're interested in having a CLI version.

This was initially developed in an Ubuntu Linux 10.10 system. I thought it wouldn't work on other systems, but it seems people are using it in a lot of places though. Some environments where it's known to work:

- Ubuntu 10.10, 11.04, 11.10
- Arch Linux
- Windows (!!!)

## Suggestions and bug reports ##

If you'd like to have a certain feature or you've found an unreported bug, please add it to the issue tracker at https://github.com/tslocum/aafm/issues
