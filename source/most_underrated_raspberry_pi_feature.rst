The Most Underrated Raspberry Pi feature
========================================

:date: 2014-11-04 16:07
:author: Dejan Noveski
:tags: linux, raspberry-pi, hdmi, cec
:category: Miscellaneous


I absolutely love my Raspberry Pi. It's awesome and there's no point for me to start writing why is that.

There is one feature tho, that's been very underrated, yet it's one of the best selling points
of the RPi:

HDMI-CEC
++++++++

"HDMI-CEC (Consumer Electronics Control) is an HDMI feature designed to allow the user to command and control
up-to 15 CEC-enabled devices, that are connected through HDMI, by using only one of their remote controls
(for example by controlling a television set, set-top box, and DVD player using only the remote control of the TV).
CEC also allows for individual CEC-enabled devices to command and control each other without user intervention." (`From Wikipedia <http://en.wikipedia.org/wiki/HDMI#CEC/>`_)

What this means is, you can control your RPi with the TV's remote! And the best part is, **it works out of the box** with your favorite xbmc flavor!
I've had my RPi as a media center since they started shipping them and I've yet to put a keyboard or a mouse on it. If you have a TV with support for CEC, you're in luck.

As I understand, Banana Pi and the likes have the hardware, but the linux cec driver doesn't work for them so they are trying to get that sorted out.
Until then, RPi is the way to go if you plan on making a media center.