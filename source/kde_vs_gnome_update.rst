##########################################
KDE vs. Gnome - A Subjective View : UPDATE
##########################################

:date: 2012-04-12 00:03
:author: Dejan Noveski
:tags: kde, gnome, desktop, linux, miscellaneous
:category: Miscellaneous


Few months ago, I `wrote an article </kde-vs-gnome-a-subjective-view.html>`_ on my view on the KDE vs. Gnome - Pros, Cons, stuff that irritates me and stuff
that's good. At that time, I was still a Gnome user. But after a lot of frustrations with the Gnome desktop,
I switched to KDE and never went back to this date. After KDE SC 4.8 dropped in. Archlinux immediately picked it up, as
always, so I had a lot of time to test drive it and see how it preforms.

This update is about my view on the updated KDE 4.8 and everything I did to make my KDE experience better (things that were on the cons list last time).
As a bonus, a bit of ranting about the current state of Linux on the Desktop.



Testing Ground
==============

Another thing that changed in the meantime, was my workstation:

    i3-2120 3.3GHz

    Asus P8Z68-V LX

    8GB ddr3 ram

    Ati Radeon HD6770 1GB DDR5 with fglrx (binary driver AND radeon) + NVidia GT440 + HD2000 - tested on all cards

    1x22"@1680x1050 + 1x24"@1920x1200 monitors

    Archlinux, latest updates and some Sabayon KDE tests



Enter KDE 4.8
################

On first looks, KDE 4.8 is not too much different from 4.7. The panel is reworked a bit, so it looks slicker, dolphin does tricks when resizing,
tweak here and there. The real update KDE 4.8 brings is increased performance. It's truly faster. For the first time, OpenGL renderer is faster than
XRender, even with VSync on. Woot! Now, I can enable desktop effects and don't worry about performance drop. On all cards and all drivers!

This might not strike you as an update, but at this moment, it's joy to me - KDE still keeps the original desktop metaphor as primary, and adds on that
an even better **optional** interface, optimized for small/touch displays. I bolded **optional** because everybody seems trigger happy when dealing with touch displays,
destroying the good ole' desktop in the process (yes. I'm looking at you Gnome. You too Ubuntu!). Anyway, for the interested, check out `Plasma Active <http://community.kde.org/Plasma/Active>`_.


Dolphin
-------

Still no mouse back-button. A bit bummer, but i learned to live with it. At least, dolphin has better UI than any GTK based file manager, so
going back and forward, up and down, is easy with mouse/keyboard. You could try the hack with xbindkeys, but apparently, it hates my Razer Deathadder.


KIO
---

Nothing has changed here. And I doubt anything will, so, I did a few tweaks - I'm using smb4k for smb hosts, and VLC has implemented KIO slaves!
I can live with this and so can you.


KDE-Wallet
----------

I was under the impression that KSecretService will change things, but it wasn't what I was hoping for. So, I made a bit of tweaking here too.
I use keychain and add **eval `keychain --eval --agents ssh -Q --quiet id_dsa`** to my .zshrc/.bashrc. Then, if you use openssh ask pass, KDE wallet can
save the password. Works like a charm. Eat that gnome keyring!


General / Performance bugs
--------------------------

A lot has changed here. There are no corruptions (to date) in the plasmoids, panels etc. All works good. Resizing dolphin is fast (except with some NVidias+blobs).
There was a significant lag when one starts dragging window that's not focused. It works great now.

The default application bundle is still better than Gnome's (seriously, compare Evince to Okular, Brasero to K3b, Gwenview to any gnome image viewer).

It still looks frickin' GOOD! It's so good looking and configurable, Gnome looks like an OS intended for some child-centric laptop. You can tweak every little bit of it.
And they should start selling this point! Average new user won't be aware of this and might revert him from KDE. Seriously guys, Moar marketing nao!


.. container:: center-align

    .. image:: ../static/uploads/beauty.png


There were some screenshots of KDE made to look like Unity/Gnome3. If you want that look, you can `have it <http://maketecheasier.com/give-kde-desktop-ubuntu-makeover/2011/11/23>`_ .

And the most important, it didn't betray the users that respect and like the desktop metaphor, just so it can be in the
competition for the best UI for the tablets/phones/TV. It still is - with Plasma Active, and by the looks of it, it already scores big time,
but they stayed loyal to the old ways. The way a desktop is meant to be.



Gnome 3 / Unity
###############

Sadly, nothing has changed here. They really stuck with it. Good for you guys! One thing tho. When a user needs 30 different
apps to change the UI, you aren't doing a good work. I've been a PC and Linux user for many years, I have friends with the same/similar
experience, and neither one of them has said good things about G3/Unity. Few friends were so appalled by the current state,
they switched to Mac after all these years (KDE was not an option because of the issues mentioned in the previous post).

The only "serious" DE that promises, is Cinamon by Linux Mint dudes, but:

    1: It's a long shot. The community working on it is small and poorly financed
    
    2: It's new. XFCE has been on the scene longer, yet it can't come even remotely close to the usage of Gnome/KDE
    
    3: It's Gnome3 at it's core!

And mate...


Bonus: My view on the whole Desktop thing
#########################################

After a long time of me ranting about the slug that was KDE compared to Gnome, I caved in and moved (for good i hope). I owe some people apologies. KDE community is focused on
**building** and **improving** stuff. Not tearing the old things and starting from scratch. I know, I know. KDE 3 to 4. Now, think about how big of a change was that, and how much
Gnome 2 changed when it hit 3. We all owe KDE apologies!

As a bonus, here are some of my thoughts on Linux on the Desktop:

No, it's not the year of Linux for the Desktop. It's (and will be) the year of Linux for the smartphone. If we want to live to see Linux on the desktop,
**PEOPLE SHOULD USE IT!**. A lot of people say they use Linux, but they don't. Either they use it in VM or dual boot it with Windows as a secondary - testing machine.

In order for people to use Linux on the desktop, a lot of things have to change.

    First and foremost, stop the effin' fragmentation! 2 DEs were enough. 4 are a lot. 6 are shaving yaks. KDE is doing the right thing at the moment. Switch to it. You'll thank me later. That will make some of the popular - entry-level distros to support their KDE flavors more and we'll stop bitching about the damned tablet mentality. 

    Second: Great Unified Package manager. See Mac's bundles. People have big HDDs now, you can ship the libs in the same folder even if the lib is present in the system.

    Third: Stop focusing on minor things and fix the Elephant. Ubuntu folk were so excited about the new test printer template, but I can't see it because I can't
    make my printer work on Linux!

    Fourth: For Buddha's sake, stop the support for antics. There's enough backlog of every distribution in history to power anything from Eniac to a super modern workstation.
    Slim down on support for old hardware.

    And most important: Stop being purists and elitists. So what if a distro ships with non-free drivers? Boo-hoo!
    Are you trying to chase even those that stick to Linux? Personally, I've been FOSS supporter for a lot of time, but
    even I don't want to limit myself because there is no (truly) free support for my hardware. NVidia/Ati blobs are free of charge - aren't they?
    So what if a user uses DE instead of Awesome/ratpoison/dwm/xmonad? Not all Linux users are sysadmins or Haskell hackers, you know. Don't call them newbs.
    Focus a bit more to the user. Ubuntu did that, and was reign of the Linux on the desktop until the Great Tabletation. They had the chance, and they blew it.

Believe me, I want to see Linux in the top percentages of usage as much as the next guy, but in order for that to happen, I have to make my wife, my brother, my aunt, my cousins to use Linux.
It's a tough task. They can't install apps, they can't install Linux on their cheap laptops, they can't make their wi-fi/printer work, **they can't find the start button!** And they make for the majority
of users! Tech savvy people are small percentage, and they tend to focus on the work instead hacking config files, compiling mesa stacks, chasing windows and hot spots for activating the launchers.
Make it work OOB. Let's copy Windows for a change :) (just kidding... or am I).

P.S. -again- For the purists s/Linux/Gnu\\/Linux/