###################################################
UPDATED: Multiseating with KDE and XBMC Like a boss
###################################################

:date: 2012-10-30 23:03
:author: Dejan Noveski
:tags: kde, xbmc, desktop, linux, miscellaneous, multiseat, xorg
:category: Miscellaneous



A while ago, I wrote a how-to concerning a multiseat setup with xbmc and KDE. Since then, I've done a lot of tweaks to
the system, optimized the configurations, made them more modular and DE independent. Also, UDEV RULES!

Make sure to read the old article `here <http://brainacle.com/multiseating-with-kde-and-xbmc-like-a-boss.html>`_ 

You can skip the part with the configurations, but if you have time, see how the old system was and how it is now.

Again, It's best if I cover my behind so:

**Backup all the files you'll be changing before you start changing them. I don't take any responsibility for any damage you might cause.**
###########################################################################################################################################

Lets start configuring!


Configuration
##################

Before you start, make sure you have a fresh distro. 3.2+ kernel, xorg 1.10+, evdev drivers for mouse and keyboard.

Finding out BusId of the GPUs
===============================

We use BusId of the GPU to configure the "Device" parts in xorg.conf. The BusID is a string that looks like "PCI:1:0:0".
To find out the right BusID, use **lspci | egrep "VGA|Display"**. The output should look something like this:

    00:02.0 Display controller: Intel Corporation 2nd Generation Core Processor Family Integrated Graphics Controller (rev 09)
    01:00.0 VGA compatible controller: Advanced Micro Devices [AMD] nee ATI Juniper XT [AMD Radeon HD 6000 Series]

The output is pretty much straightforward. You can see which Bus is which GPU. So, based on this, your BusIDs would be
**PCI:0:2:0** and **PCI:1:0:0**. Save those strings somewhere, you'll need them later.


Configuring Udev
================

Here's the fun part :). We write udev rules to **tag** the inputs, so we can assign them to specific seats.

First, we have to distinguish the different inputs for the seats by some identifier. If your input are different models and/or vendors
it's very easy. If not, comment me, and if there's enough comments, I'll try to write up a good Udev tutorial on how to find
sys paths, query by advanced parameters etc.

So now, open a terminal and write 'lsusb'. The output should look like this:

.. code-block:: bash

    Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
    Bus 003 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
    Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
    Bus 001 Device 003: ID 046d:c50e Logitech, Inc. Cordless Mouse Receiver
    Bus 003 Device 003: ID 1058:1021 Western Digital Technologies, Inc. Elements 2TB
    Bus 003 Device 004: ID 046d:c318 Logitech, Inc. Illuminated Keyboard
    Bus 003 Device 005: ID 1532:0016 Razer USA, Ltd DeathAdder Mouse

Notice the ID. Its meaning is VENDOR_ID:PRODUCT_ID. My inputs are 046d:c50e, 046d:c318 and 1532:0016.

Save your IDs.

Next, go to /etc/udev/rules.d and create a script called *99-multiseat.rules*.

Next few lines should go in it:

.. code-block:: bash

    SUBSYSTEM=="input", ENV{ID_INPUT.tags}="input_default"
    SUBSYSTEM=="input", ENV{ID_VENDOR_ID}=="046d", ENV{ID_MODEL_ID}=="c50e", ENV{ID_INPUT.tags}="input_right"

The first line tags all inputs with tag "input_default". The second line tags the specific input with Vendor ID 046d and model ID c50e with the tag "input right".
Change the values to your vendor and model ID.

Save the script, mark it +x and reboot.


Configuring X
=============

Last how-to used one xorg.conf. Now, we'll use main seat and off-seat configuration files for xorg.conf.

Open /etc/X11/xorg.conf for editing, as root user. If you don't have that file, just create it. **Note:**
Different distros might have different xorg.conf path. Most of the distros use /etc/X11/.

We'll use a simple, barebone xorg configuration, and let X autoconfigure everything else:

.. code-block:: bash

    Section "ServerFlags"
        # Xorg will otherwise not start if it can't find a mouse to use. Better safe than sorry.
        Option  "AllowMouseOpenFail"    "true"
    EndSection

    Section "Device"
        Identifier "gfx_left"
        Driver "radeon"
        BusID       "PCI:1:0:0"
        EndSection

        Section "InputClass"
        Identifier "ignore_other_seats"
        MatchTag "input_right"
        Option "Ignore" "yes"
    EndSection

**That's ALL!** By using MatchTag, we match the inputs tagged as "input_right" and we "Ignore" them.
Note the BussID in the "Device" section. It should match your primary GPU.

Save the script. This should work independently of the next configuration file.

Open /etc/X11/xorg-xbmc.conf for editing, as root user.

.. code-block:: bash

    Section "Device"
        Identifier "gfx_right"
        Driver "intel"
        BusID       "PCI:0:2:0"
    EndSection

    Section "InputClass"
        Identifier "ignore_other_seats"
        Option "Ignore" "yes"
    EndSection

    Section "InputClass"
        Identifier "input_right"
        MatchTag "input_right"
        Option "Ignore" "no"
    EndSection

This config will be used by the off-seat xserver. We point to the graphic card, we tell it to ignore all the inputs except 
those tagged as "input_right" and we're set here.


Testing the configuration
=========================

First, let the main xserver load the xorg.conf. Switch to runlevel 3 and then 5 again (telinit [runlevel]).

When the main seat is up, open up a terminal, and execute this:

.. code-block:: bash

    /bin/su -- - [your off-seat user] -c "xinit -- /usr/bin/X :2 vt9 -nolisten tcp -config xorg-xbmc.conf -sharevts -novtswitch" </dev/tty9 >/dev/tty9 2>/dev/tty9

It should get the server up.

If you want to daemonize this, open up a init script (either /etc/init.d/xorg-offseat or /etc/rc.d/xorg-offseat depending on distro) and inser the following:

.. code-block:: bash

    #!/bin/sh

    set -e

    PATH=/usr/local/bin:/usr/local/sbin:/bin:/usr/bin:/sbin:/usr/sbin
    DAEMON=/usr/bin/startx
    PIDFILE=/var/run/xorg-offseat.pid
    USER="[your offseat user here]"
    HOME=~[your offseat user home dir here]
    CONFIG="xorg-xmbc.conf"

    case "$1" in
      start)
        sleep 5s
        echo -n "Starting X session: startx"
        cd $HOME
        /bin/su -- - $USER -c "xinit -- /usr/bin/X :2 vt9 -nolisten tcp -config $CONFIG -sharevts -novtswitch" </dev/tty9 >/dev/tty9 2>/dev/tty9 & echo $! >$PIDFILE
        #/usr/bin/startx -- :2 vt9 -nolisten tcp -config $CONFIG -sharevts -novtswitch"
        echo "."
      ;;

      restart)
        /etc/rc.d/xorg-offseat stop
        /etc/rc.d/xorg-offseat start
      ;;

      stop)
        echo -n "Stopping X session: startx"
        kill `cat $PIDFILE` || echo -n " not running"
        echo "."
      ;;

      *)
        echo "Usage: /etc/init.d/xorg-offseat {start|stop|restart}"
        exit 1
        ;;
    esac

You can start this as you start any other daemons. And stop it as well.

If you don't want to use daemons, you can always rely on your display manager.
I used KDM, trieg gdm but a lot has changed and the docs aren't much updated, so I can't be sure if it'll work.

Configuring KDM
===============

Open **/usr/share/config/kdm/kdmrc** for editing, as root.

In the [General] section, change the **ReserveServers** and **StaticServers** as follows:

.. code-block:: bash

    ReserveServers=:2,:3
    StaticServers=:0,:1

At the end, add this snippet:

.. code-block:: bash

    [X-:0-Core]
    ServerArgsLocal=-nolisten tcp -config xorg.conf

    [X-:1-Core]
    ServerArgsLocal=-nolisten tcp -config xorg-xbmc.conf -sharevts -novtswitch

If you want a user to auto-login on the second seat (in my case, I want the xbmc user to autologin),  add this to the file:

.. code-block:: bash

    [X-:1-Core]
    AutoLoginEnable=true
    AutoLoginLocked=false
    AutoLoginUser=xbmc
    ClientLogFile=.xsession-errors

**Don't execute custom sessions!**. It's bugged. Try the default one and autostart xbmc from there.

Things from the old How-To
##########################

Tweaking HDMI audio for XBMC
============================

You need to find the alsa sinks of your system. To do that, use **aplay -l**. The output should be something like this:

.. code-block:: bash

    **** List of PLAYBACK Hardware Devices ****
    card 0: PCH [HDA Intel PCH], device 0: ALC887-VD Analog [ALC887-VD Analog]
      Subdevices: 1/1
      Subdevice #0: subdevice #0
    card 0: PCH [HDA Intel PCH], device 1: ALC887-VD Digital [ALC887-VD Digital]
      Subdevices: 1/1
      Subdevice #0: subdevice #0
    card 0: PCH [HDA Intel PCH], device 3: HDMI 0 [HDMI 0]
      Subdevices: 1/1
      Subdevice #0: subdevice #0
    card 0: PCH [HDA Intel PCH], device 7: HDMI 1 [HDMI 1]
      Subdevices: 1/1
      Subdevice #0: subdevice #0
    card 1: Generic [HD-Audio Generic], device 3: HDMI 0 [HDMI 0]
      Subdevices: 1/1
      Subdevice #0: subdevice #0

Next, find out which sink goes to the right HDMI. Use aplay for that:

    aplay -D plughw:[cardId],[deviceId] /usr/share/sounds/alsa/Front_Center.wav

Replace cardId and deviceId from the list of playback hardware devices. When you hear a sound on your TV,
you've hit the right device. Just remember the card id and the device id.

Go to XBMC System Settings->Audio output. Choose Audio output device - Custom. Insert plughw:[cardId],[deviceId](e.g. plughw:0,7) in Custom audio device. You're done. XBMC should route audio thru your HDMI.


Tweaking policykit
==================

If the second seat is unable to use removable drives, bluetooth dongles, policykit is to blame.

Add/Edit **/etc/polkit-1/localauthority/50-local.d/mseat.pkla**:

.. code-block:: bash

    [allow operations]
    Identity=unix-group:plugdev
    Action=org.freedesktop.udisks.*;org.blueman.*;org.freedesktop.pulseaudio
    ResultAny=yes
    ResultActive=yes
    ResultInactive=yes


That should be it.

If this setup doesn't work for you, don't give up easily. Ask around forums and irc. These setups can differ largely, based on the hardware and the setup. You can ask in comments as well. I'll try to help out as much as I can.

Post Scriptum
#############

Comment me with your issues, I'll try to help as much as I can. Try to leave verbose outputs please. Best of luck with your multi-seat setups!