Initial setup
=============

Do this on the fastest RPi
--------------------------
Regardless of which RPi I ultimately use, I do this setup work on the fastest
RPi I have. The speed difference between the single core and quad core RPi
models is tremendous. When the setup is done, move the microSD card into the
slower target RPi.

Update packages
---------------
Update the existing packages before doing anything else. The RPi is connected to
the internet. If there are any security patches in the package upgrades, I want
those installed as soon as possible.

As with any Linux OS, the image of Pi OS Lite burned on the microSD card is out
of date the moment it is created because packages are updated on a daily basis.
It is up to the user to maintain the OS.

In general, this is done with the single command::

   sudo apt full-upgrade

The same works for the initial update, I show optional additional steps I like
to take. Regardless, the update takes some time because many packages will be
missing updates that were released after the Pi Lite OS image was created.

Check for updates, do not install anything::

   sudo apt update

List which packages are upgradable::

   sudo apt list --upgradable

Perform all upgrades::

   sudo apt upgrade

List again to see that all packages are upgraded::

   sudo apt list --upgradable

Or run ``sudo apt full-upgrade`` to see there is nothing to upgrade.

When package files are no longer needed, ``apt`` outputs a message to remove
unnecessary files with ``sudo apt clean``.

Fix the keyboard (US)
---------------------
I am in the US. The ``|`` and ``~`` keys output different symbols. The RPi
defaults to a UK keyboard. Fix this.

Open the *Raspberry Pi Software Configuration Tool*::

   pi@raspberrypi:~ $ sudo raspi-config

Select::

   4 Localisation Options     Set up language and regional settings to match your location

Select::

   Change keyboard layout

Fix the font
------------
The default font is too small. I cannot read it. Change the font and increase
the font size::

   sudo vi /etc/default/console-setup

Set ``FONTFACE="Terminus"`` and ``FONTSIZE="16x32"``.


Source the console-setup script::

   sudo /etc/init.d/console-setup.sh restart

*The font updates and all sessions use the new settings.*

Install initial critical packages
---------------------------------

I need Git and Vim to do anything useful.

.. code-block:: bash

   sudo apt install git
   sudo apt install vim

To transport my custom Vim setup, I clone my ``vim-dotvim`` repo in my ``$HOME``
folder and rename it ``.vim``::

   cd ~
   git clone https://sustainablelab/vim-dotvim.git
   mv vim-dotvim/ .vim

Attempting to start Vim I get errors because the Vim plugins are Git submodules.
Clone the submodules as well::

   git submodule init
   git submodule update

Now my Vim installation is OK except for color. Not much I can do about color.
But the Vim version is missing features I want. I build Vim from source. To
install with X11, install the xorg-dev. I install checkinstall to make it easy
to remove stuff, but I haven't used it yet.

    sudo apt install xorg-dev
    sudo apt install checkinstall
    sudo apt build-dep vim

Now, clone Vim::

    mkdir ~/repos
    cd ~/repos
    git clone https://github.com/vim/vim.git

Remove the vim package before continuining::

    sudo apt remove git

Build::

    make
    sudo make install

Remove old Vim things hanging around from original package install::

    sudo apt autoremove

Should find Vim now, but I need to type ``/usr/local/bin/vim`` why? I just need
to restart. Then I can start Vim with ``vim``.

Lynx is easy to install but not well-supported by websites.

Install the Chromium browser. This is *not* the same as Google Chrome, but it is
close enough and it has Javascript, so I should be able to run bokeh.

Install Chromium::

   sudo apt install chromium-browser

This installs chromium. List the commands that launch the browser::

   pi@raspberrypi:~ $ ls /usr/bin | grep browser
   chromium-browser
   gnome-www-browser
   sensible-browser
   x-www-browser

But XWindows is required to launch the browser. Install everything needed with::

    sudo apt install xorg

While trying to set this up, I installed xinit::

    sudo apt install xinit

But I think that is included with xorg.

Now start Chromium browser with::

    startx /usr/bin/chromium-browser

This opens the browser window, but I am unable to quickly switch back to the
terminal. For that kind of usage, I should just install the entire desktop. The
only way to get back to the terminal is to quit the browser with Ctrl+Shift+W.

But the browser does open up pretty quick. It might be usable as for bokeh
plotting.

Use Pi OS Lite
==============

Shutdown
--------

sudo shutdown now


Multiple Sessions
-----------------
Alt+F1-F6 switches between terminal sessions. There are six: ``tty1`` through
``tty6``. They are all available when the RPi boots. Log in with username and
password. Logout with ``exit``.

The RPi starts with one session open, the one you get with Alt+F1.
Pressing Alt+F2 opens a new session. Log in again. Press Alt+F1 to
switch back to the original, Alt+F2 to switch back to the new.

As expected, the Alt+F1-F12 keybindings work from *within* applications too.
For example, opening Vim, I can switch to a different terminal while Vim still
runs in the other terminal.

Utilities
=========

Linux version
-------------
Check what version I am running::

   lsb_release -a
   Distributor ID:	Raspbian
   Description:	Raspbian GNU/Linux 10 (buster)
   Release:	10
   Codename:	buster

htop: monitor performance
-------------------------

When the RPi boots, one raspberry means single core, four raspberries means quad
core. Another way to check this is with ``htop``::

   htop

This opens the application. The upper left shows the number of cores.

``htop`` is pre-installed with Pi OS Lite.

df: Disk space
--------------
Check the file system to see how much space is available on the microSD card::

   pi@raspberrypi:~ $ df -h

   Filesystem      Size  Used Avail Use% Mounted on
   /dev/root        15G  2.2G   12G  16% /
   devtmpfs        430M     0  430M   0% /dev
   tmpfs           463M     0  463M   0% /dev/shm
   tmpfs           463M  6.3M  457M   2% /run
   tmpfs           5.0M  4.0K  5.0M   1% /run/lock
   tmpfs           463M     0  463M   0% /sys/fs/cgroup
   /dev/mmcblk0p1  253M   54M  199M  22% /boot
   tmpfs            93M     0   93M   0% /run/user/1000

For example, I am using a 16GB microSD card. User storage is on ``/dev/root``.
This shows 15GB total, 2.2GB used, 12GB available.

Run Python
==========
python is Python2.7, python3 is Python3.7

Simple way to alias is to create a venv with Python3, activate venv, upgrade
pip, good to go.

NumPy installs, but attempting to import yields the error: ``libf77blas.so.3:
cannot open shared obecjt file: No such file or directory``

Fix this by installing the missing libraries::

    sudo apt install libatlas-base-dev
