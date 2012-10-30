##########################################
Multiseating with KDE and XBMC Like a boss
##########################################

:date: 2012-05-27 21:03
:author: Dejan Noveski
:tags: kde, xbmc, desktop, linux, miscellaneous, multiseat, xorg
:category: Miscellaneous


`UPDATED with udev, modular configuration and less headache here. <http://brainacle.com/multiseating-with-kde-and-xbmc-like-a-boss-update.html>`_
#################################################################################################################################################


Linux multi-seat setup is a setup where multiple users can use the same PC simultaneously.
You have a monitor and input devices for each of those users, and a separate X session for all.
It's a PITA to set up if you don't know what to do, but the end result is legendary.
Trust me, I spent countless hours setting up mine, but it was worth it.


Why?
####

- You have a spare hardware, and you want to utilize it
- You have 1 PC, but you want your sibling/SO/family member to have it's own independent environment
- You want to add a TV or multimedia session monitor to your setup that will be independent of your current desktop session (we'll focus on this)
- It's awesome!


What do I need?
###############

First off, you need a PC with at least 2 graphic cards. Any combination is good. N discrete GPUs or 1 integrated + N discrete.
If you mix graphic card vendors, try to use the FOSS drivers. I haven't tested multiseating with mixed binary drivers or blobs and FOSS drivers. I'll update the post once I test this.

You need at least 1 monitor per seat, or you can switch one monitor back and forward. In the guide (my setup), I use 3 monitors.
One dual-head seat and one single-monitor seat. At least one monitor per GPU.

You'll need separate inputs for each seat. They are optional, but how would that work? Try to, at least, get another mouse.

You'll need advanced knowledge on Linux. Sorry, but this is an advanced topic, so If you don't know how to set up xorg.conf, you are
in for a lot of frustrations.

Iron will and nerves of steel: It might be a bumpy ride, so brace yourself.


My scenario
###########

I recently purchased a TV and wanted to hook it up to the PC, pop XBMC on it, and, if possible, make it work with separate user/input/session. A perfect use-case for a multi-seat. I already had a dual-head setup with an ATI Radeon HD6770. My PC has
an integrated HD2000 GPU (i3-2120) which I could use for the second seat. Pics:

.. container:: center-align

    .. image:: ../static/uploads/ms1.jpg
    
    2 monitors on the Ati, TV tied to the onboard graphic card via HDMI
    
    .. image:: ../static/uploads/ms2.jpg
    
    Main seat - dual-head, KDE session
    
    .. image:: ../static/uploads/ms3.jpg
    
    XBMC running on the TV in separate session/seat
    
    .. image:: ../static/uploads/ms4.jpg
    
    KDE running on the TV in separate session/seat


I could have gone with the triple-head setup, but:

1. It puts a lot of strain to the main GPU
2. Vsync is out of whack
3. It bugs out often
4. All my monitors are different sizes/resolutions.
5. Running XBMC on fullscreen on the third monitor is a pain.

With multiseat, I run xbmc automatically, fullscreen, in a separate session and a separate user.

My distro is Archlinux. The configurations or config file paths might differ in other distros, but it shouldn't be a problem.

**Backup all the files you'll be changing before you start changing them.**
###########################################################################

Lets start configuring!


Xorg Configuration
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


Finding out Device Path of the Input devices
============================================

Plug in all the inputs you'll be using. Use **ls /dev/input/by-id**. The output should look like this:

    usb-Logitech_Logitech_Illuminated_Keyboard-event-if01
    usb-Logitech_Logitech_Illuminated_Keyboard-event-kbd
    usb-Logitech_USB_RECEIVER-event-mouse
    usb-Razer_Razer_DeathAdder-event-mouse

From this list, you can identify your inputs. If you have same vendor/model inputs, you can reference them by
bus and device ID, and even by the simplest form **/dev/input/eventX**. I suggest you use some specific referencing - by id or by device id.

So, my device paths are:

- /dev/input/by-id/usb-Logitech_Logitech_Illuminated_Keyboard-event-kbd
- /dev/input/by-id/usb-Logitech_USB_RECEIVER-event-mouse
- /dev/input/by-id/usb-Razer_Razer_DeathAdder-event-mouse

That's all we need for now. Let's pull all of this in xorg.conf:

Configuring X
=============

This is the most important step in making a multiseat. We work solely on xorg.conf.

Open /etc/X11/xorg.conf for editing, as root user. If you don't have that file, just create it. **Note:**
Different distros might have different xorg.conf path. Most of the distros use /etc/X11/.

We'll use a simple, barebone xorg configuration, and let X autoconfigure everything else:

.. code-block:: bash

    Section "ServerFlags"
        # Xorg will otherwise not start if it can't find a mouse to use. Better safe than sorry.
        Option  "AllowMouseOpenFail"    "true"
    EndSection

    Section "InputDevice"
            #Configuring the keyboard for the main seat
            Identifier "keyboard0"
            Driver "evdev"
            Option "Device" "/dev/input/by-id/usb-Logitech_Logitech_Illuminated_Keyboard-event-kbd"
    EndSection

    Section "InputDevice"
            #Configuring the mouse for the main seat
            Identifier "mouse0"
            Driver "evdev"
            Option "Protocol" "Auto"
            Option "Device" "/dev/input/by-id/usb-Razer_Razer_DeathAdder-event-mouse"
    EndSection

    Section "InputDevice"
            #Configuring the mouse for the second seat
            Identifier "mouse1"
            Driver "evdev"
            Option "Protocol" "Auto"
            Option "Device" "/dev/input/by-id/usb-Logitech_USB_RECEIVER-event-mouse"
    EndSection

    Section "Device"
            #Configuring the discrete GPU
            Identifier "radeon"
            Driver "radeon" #Or a driver specific to your GPU - "nvidia"
            BusId "PCI:1:0:0" #From the BusID section above
    EndSection

    Section "Device"
            Identifier "intel"
            Driver "intel" #Or a driver specific to your GPU - "nvidia", "radeon"
            BusId "PCI:0:2:0" #From the BusID section above
    EndSection

    Section "Screen"
            # Configuring the screen for the main seat
            Identifier "screen0"
            Device "radeon"
    EndSection

    Section "Screen"
            # Configuring the screen for the second seat
            Identifier "screen1"
            Device "intel"
    EndSection

    Section "ServerLayout"
            #Configuring the Server layout for the main seat
            Identifier      "main"
            Screen          "screen0"       0                   0
            InputDevice     "mouse0"        "CorePointer"
            InputDevice     "keyboard0"     "CoreKeyboard"
            #This is a must - otherwise, it will add all the inputs to this seat
            Option          "AutoAddDevices"        "off"
    EndSection

    Section "ServerLayout"
            #Configuring the Server layout for the second seat
            Identifier      "tv"
            Screen          "screen1"       0                   0
            InputDevice     "mouse1"        "CorePointer"
            
            #If you have a spare keyboard, make another InputDevice entry for it and add it here
            #InputDevice     "keyboard1"     "CoreKeyboard"

            #This is a must - otherwise, it will add all the inputs to this seat
            Option          "AutoAddDevices"        "off"
    EndSection

Next, we configure the session manager

Configuring KDM
===============

Since I use KDE, I'll show you how to configure KDM. If you use other DE, please google around, it's mostly similar to this.

Open **/usr/share/config/kdm/kdmrc** for editing, as root.

In the [General] section, change the **ReserveServers** and **StaticServers** as follows:

.. code-block:: bash

    ReserveServers=:2,:3
    StaticServers=:0,:1

At the end, add this snippet:

.. code-block:: bash

    [X-:0-Core]
    ServerArgsLocal=-nolisten tcp -layout main

    [X-:1-Core]
    ServerArgsLocal=-nolisten tcp -layout tv -sharevts -novtswitch

If you want a user to auto-login on the second seat (in my case, I want the xbmc user to autologin),  add this to the file:

.. code-block:: bash

    [X-:1-Core]
    AutoLoginEnable=true
    AutoLoginLocked=false
    AutoLoginUser=xbmc
    ClientLogFile=.xsession-errors

And, if you want a custom session to be executed, not the default - KDE, you'll need 2 things:

1. A shell script that would execute the session (exec xbmc-standalone), marked executable
2. Add a line in **[X-:1-Core]** - **Session=/path/to/that/shell/script**

That's it for KDM. If you followed and saved everything, you can restart X now. You don't need to reboot (but do it anyway).
Next login, you should see different sessions on different GPUs

**If you have any problems starting the X server, just move or remove xorg.conf, and revert kdmrc. Keep backups of the old files, so you can revert fast**

Fine tuning
###########

Now, I have a KDE session on my main seat, and xbmc session on my second seat. But no sound...
Making the sound work on a multi-seat system is quite difficult and it differs from the type of sound architecture.

In my scenario, I feed my audio through the HDMI cable to the TV, and it's quite easy to do so. I have pulseaudio on my system.

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

The "howto" is quite big for me to follow. If you feel I've missed anything, notify me in comments and I'll update the post. Thanks for understanding.

I want to thank the Archlinux and Gentoo communities for their effort on the wikis. They helped me a lot in making this.
Also, kudos to the whole Linux community for doing things this awesome.

And, XBMC devs, hats off to you guys!



