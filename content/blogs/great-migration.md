+++
date = '2026-06-19'
title = 'One Distro to Rule them all'
tags = ["NixOS", "kubernetes", "IaC"]
+++

# The Problem

I currently have my K3S cluster running on NixOS. This has been invaluable to me for many reasons, but one of them is because I can review my configuration at any time from a Github Repository. No SSH or dnf list needed.

*However*

This is a problem for my laptop. Currently, I have Bazzite installed because I thought that someday, maybe I would play some videogames on it. That was a lie. So now I have a 'gaming' laptop with no games and a lot of software I have no need of. This is my plan to fix that.

## Planning the Migration

While it would be nice to reboot and install immediately, I do have necessary files on this machine. So I will take this in steps:

1. Backup my /home folder to another pc
2. Ensure my dotfiles are safely in github
3. Install NixOS and enable flakes
4. Pull my [Homelab Repo](https://github.com/That1LinuxGuy/Homelab) and add my laptop configuration
5. Commit, Modify the nixos-rebuild command to pull from my repo, and finish

## Potential Issues

While 5 steps is not complex, per-se, I assume I will run into issues along the way. Some potential problems I might encouter are:

- Desktop Environment | my servers have no DE or and GUI apps installed. will KDE work as I expect it to with /nix/store builds?
- Build Times | my servers rebuild every night while I sleep because they run 24/7. My Laptop does not and has a much older CPU. Will rebuilding steal too much of my productive time?
- Different Configuration | I built my common.nix based on server needs. Will I have to refactor my code to account for a Desktop use case as a "common" device?

## Post Mortem

I learned a lot from this migration. Almost everything went perfectly smooth, but I did have some hiccups which I will share.

*but first*

### Positives

Installing Desktop NixOS taught me some things I never would have learned otherwise:

**Don't Repeat Yourself (DRY)**
Reconciling default configurations for a server vs. the condiguration I needed for a desktop required carefully comparing WHAT I had and WHY it was there. For example, there is absolutely no need for X11 keybinds in a server that only uses the linux shell

**Configuring apps with nix**

On other distros, I have had to install CAC card services like ccid, pcsc, and opensc then open firefox to configure the browser to actually use those tools seperately. With Nixos I can use built in tools to do this whole process from the command line:

```nix
  programs.firefox = {
    enable = true;
    policies = {
      SecurityDevices = {
        Add = {"p11-kit-proxy" = "${pkgs.p11-kit}/lib/p11-kit-proxy.so"; };
      };
    };
  };
```

Once I set this up, all I had to do was access a DOD website and it worked -- no firefox settings needed.

**flake evaluations are relative**

I have my entire stack configured in /homelab and I decided to start using qtile so I can experiment with python code. 

Becuase I'm new to qtile, I placed qtile-config.py in my /code repository and symlinked it to /etc/nixos which is the default evaluation path.

*However*

because my flake.nix is lcoated in /homelab, I needed to add it to /homelab/hosts/pikachu instead.

This will be very good knowledge to have as I experiment with qtile in NixOS

### Negatives

**Syntax**

I'm not sure if I will learn to love semicolons or just learn to deal with them, but I must have forgotten to add some necessary syntax more than 20 times JUST trying to get my laptop running the way I wanted. 

**Build Times**

As I expected, rebuilding my system every time I want to install a package is a little slow.

What makes it even worse is that every time I build, NixOS reminds me if I have not committed to git...

Sure, it helps keep my honest and makes my repo stay up to date. But do I really need to update my entire Homelab just to add kubernetes commands?

**Artificial Intelligence**

When it comes to reading docs, AI is often better than I am, but in very niche environments it fails miserably.

I wanted qtile on NixOS running Wayland. Simple, but not for AI

![A gemini chat](./images/chat.png)

which led me to this configuration:
```Nix
programs.qtile = {
  enable = true;
  backend = "wayland"; 
  configFile = ./qtile-config.py;
```

Then this:

```Nix
windowManager.qtile = {
  enable = true;
  configFile = ./qtile-config.py; # Points to your symlink
```

Then finally:
```Nix
services.xserver = {
  enable = true; # Keeps the display architecture active
  
  windowManager.qtile = {
    enable = true;
    configFile = ./qtile-config.py; # Points to your symlink
```
