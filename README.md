# TestDisk, PhotoRec, and QPhotoRec 7.2-WIP
#### for 64 bit Debian-based systems

-----------------

Author: deltabravozulu

Date: 04 August 2020 (hbd Dylan!)

-----------------

### Background:

-----------------

I wanted to install QPhotoRec, which is created by the maker of TestDisk and PhotoRec. They do not have a standalone version or anyting installable from any package manager I am familiar with. Instead, you traditionally instal TestDisk via `sudo apt install testdisk -y` which comes only with testdisk and photorec.

The .deb package I [found](https://askubuntu.com/questions/393076/is-there-a-usable-gui-version-of-photorec-or-testdisk) online (which was mentioned everywhere...) was for the first edition. It was meant to be a part of Parted Magic and would generally fail out rather than allowing one to bypass the [opening screen](https://linuxforums.org.uk/index.php?topic=13365.0) and actually use it. Rather than hacking around on that particular debian package (okay, I did and I figured out why it was failing), I decided it would be more fruitful to build it locally.

Unfortunately, upon deleting the original package, I found that it took the forensics-extra package with it. Further, when I built the TestDisk source locally (tl;dr is below), it installed it in /usr/local/bin/ instead of the repository installation target which is /usr/bin/. Because dpkg (and as such, apt) are only aware of packages they install, I could not reinstall forensics-extra without having a duplicate, older version of TestDisk sitting in /usr/bin/. 

Rather than hacking around on blocking the TestDisk repo download, I decided to package my own full Debian package of TestDisk 7.2-WIP for 64-bit Debian-based Linux distros with a working version of QPhotoRec complete with the menu entries (they don't come with a source installation, unfortunately). 

---------------

### Quick install on 64-bit Debian distros:

---------------

1.) Open terminal (ctrl+alt+t)

2.) Paste this junk in and hit enter:

```
cd ~/Downloads && wget https://github.com/deltabravozulu/testdisk-qphotorec/raw/master/testdisk_deb/testdisk_7.2-WIP-deltabravozulu-qphotorec-included_amd64.deb && sudo dpkg --install testdisk_7.2-WIP-deltabravozulu-qphotorec-included_amd64.deb
```

---------------

### Updating the .deb (if new versions emerge):

---------------
When a new version of testdisk emerges (we're at 7.2-WIP right now) basically what you are going to do is as follows:

1.) Clone my repo:
```
# clone the repo
git clone https://github.com/deltabravozulu/testdisk-qphotorec.git

#enter the directory
cd testdisk-qphotorec
```

2.) Follow the instructions at [cgsecurity/testdisk](https://github.com/cgsecurity/testdisk/blob/master/INSTALL) up until the part where it says `make install` (Apologies if they change--here's how it's currently done)
	
```
# uninstall old testdisk
sudo apt remove --purge testdisk

# install dependencies
sudo apt install build-essential e2fslibs-dev libncurses5-dev libncursesw5-dev ntfs-3g-dev libjpeg-dev uuid-dev zlib1g-dev qtbase5-dev qttools5-dev-tools pkg-config dh-autoreconf git

# download testdisk source
git clone https://github.com/cgsecurity/testdisk.git

# enter directory
cd testdisk

# generate config script:
mkdir config
autoreconf --install -W all -I config

# set up the configuration files for your system:
./configure

# compile the package
make

# check the install
make check
```

Now, note we are deviating from the install instructions and stopping here. 

3.) Move files around (note, this only handles updating the binaries--if you care about updated manuals and whatnot, you'll need to grab those from ./testdisk/man after you compile it).
```
# I'm assuming you are still in testdisk-photorec/testdisk/
# cd up a directory 
cd ..
cp testdisk/src/fidentify testdisk_deb_sourcedir/usr/bin/
cp testdisk/src/photorec testdisk_deb_sourcedir/usr/bin/
cp testdisk/src/qphotorec testdisk_deb_sourcedir/usr/bin/
cp testdisk/src/testdisk testdisk_deb_sourcedir/usr/bin/
```

4.) Fix the md5sums in the testdisk_deb_sourcedir/DEBIAN folder:
```
# cd into the testdisk_deb_sourcedir
cd testdisk_deb_sourcedir

# calculate md5sums
md5sum usr/share/applications/qphotorec.desktop > DEBIAN/md5sums
md5sum usr/share/man/man8/fidentify.8 >> DEBIAN/md5sums
md5sum usr/share/man/man8/photorec.8 >> DEBIAN/md5sums
md5sum usr/share/man/man8/testdisk.8 >> DEBIAN/md5sums
md5sum usr/share/man/man8/qphotorec.8 >> DEBIAN/md5sums
md5sum usr/share/menu/qphotorec >> DEBIAN/md5sums
md5sum usr/share/doc/testdisk/changelog.Debian >> DEBIAN/md5sums
md5sum usr/share/doc/testdisk/NEWS >> DEBIAN/md5sums
md5sum usr/share/doc/testdisk/copyright >> DEBIAN/md5sums
md5sum usr/share/doc/testdisk/README.md >> DEBIAN/md5sums
md5sum usr/share/pixmaps/qphotorec.png >> DEBIAN/md5sums
md5sum usr/bin/photorec >> DEBIAN/md5sums
md5sum usr/bin/fidentify >> DEBIAN/md5sums
md5sum usr/bin/qphotorec >> DEBIAN/md5sums
md5sum usr/bin/testdisk >> DEBIAN/md5sums
```

5.) Calculate the total size of the directories for DEBIAN/control:

```
# The man pages for dpkg suggest that one should use the size of the entire directory in bytes/1024 as the file size and round up
echo $(( $( du --summarize --bytes | cut --fields=1 ) / 1024 + 1 ))
	# you can use this if you ant to be more exact:
	# echo $(( $( du --summarize --block-size=1 | cut --fields=1 ) / 1024 + 1 ))
	# you can also use this for the same outcome:
	echo $(( $( du -ks | cut --fields=1 ) + 1 ))
	# You can also just use du -ks 
```

6.) Now, open DEBIAN/control and change the line that says "Installed-Size: 14xxx" (Line 7) to whatever number you decided on above. You should also change the version number (Line 2):
```
# Do it however you want but it is probably easiest to do this:
nano DEBIAN/control

# Then ctrl+o and write the file. Neat. You're done with that. 
```

7.) The fun part begins: let's build that .deb!
```
# cd back to the testdisk-qphotorec folder:
cd ..

# build that baby
dpkg-deb -Z xz -b testdisk_deb_sourcedir/ testdisk_deb/

# nice
```

8.) Now for the good stuff. Let's install it! 

```
sudo dpkg --install testdisk_deb/testdisk_7.2-WIP-deltabravozulu-qphotorec-included_amd64.deb`
# or whatever version you gave it--just look for the deb file.
```

9.) Done! 

