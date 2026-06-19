+++
date = '2026-06-19'
title = 'One Distro to Rule them all'
tags = ["NixOS", "kubernetes", "IaC"]
+++

# The Problem

I currently have my K3S cluster running on NixOS. This has been invaluable to me for many reasons, but one of them is because I can review my configuration at any time from a Github Repository. No SSH or dnf list needed.

*However*

This is a problem for my laptop. Currently, I have Bazzite installed because I though that someday, maybe I would play some videogames on it. That was a lie. So now I have a 'gaming' laptop with no games and a lot of software I have no need of. This is my plan to fix that.

## Planning the Migration

While it would be nice to reboot and install immediately, I do have necessary files on this machine. So I will take this in steps:

1. Backup my /home folder to another pc
2. Ensure my dotfiles are safely in github
3. Install NixOS and enable flakes
4. Pull my [Homelab Repo](https://github.com/That1LinuxGuy/Homelab) and add my laptop configuration
5. Commit, Modify the update command, and finish

## Potential Issues

While 5 steps is not complex, per-se, I assume I will run into issues along the way. Some potential problems I might encouter are:

- Desktop Environment | my servers have no DE or and GUI apps installed. will KDE work as I expect it to with /nix/store builds?
- Build Times | my servers rebuild every night while I sleep because they run 24/7. My Laptop does not and has a much older CPU. Will rebuilding steal too much of my productive time?
- Different Configuration | I built my common.nix based on server needs. Will I have to refactor my code to account for a Desktop use case as a "common" device?

## Post Mortem

TBD :construction: :construction: :construction:
