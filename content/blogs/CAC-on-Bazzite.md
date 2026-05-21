+++
date = '2026-05-15'
title = 'DOD CAC card on Bazzite'
tags = ["linux", "atomic"]
+++

# What's the issue?

In [Bazzite](https://bazzite.gg/) (and probbaly other Fedora Atomic distros), firefox is pre-installed as a flatpak application. However, because of the Sandboxing, getting firefox to recognize the CAC card can be a challenge. There is an answer in the Bazzite Discord that I found, but I made my own solution that I believe is simpler.

## How do we fix it?

It's quite simple really. Don't use firefox flatpak... Instead, we will use a browser specifically for accessing DOD websites. Personally, I prefer [librewolf](https://librewolf.net/).

## How did I do it?

Install the appimage from their website [here](https://librewolf.net/installation/linux/). Below the section for "Flatpak" there is an option to install the AppImage. Run this per instructions on their page.

After installing and running, open settings and search "certificates" and click on Manage Security Devices\
Then you can Load a new driver, and search for /usr/lib64/pkcs11/opensc-pkcs11.so

Once this is added, you should be able to plug in your CAC card and connect to DOD sites!

## Why is this better than using Flatpak?

In my opinion, there are two main reasons to use this method:
1. A dedicated browser for DOD activities. I think this is a good practice for security.
2. This removes any possibilty that a flatpak rule would get reset during an update. When Bazzite 45 is released, any appimages will not be removed or modified.
