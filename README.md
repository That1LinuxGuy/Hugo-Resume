# My Resume Website
---
This is the complete setup for my hugo-profile resume - all bells and whistles included

## What does it do?
This repository will feed into [Resume](https://github.com/That1LinuxGuy/Resume) using Github Actions,
built into a docker image using, and then get pulled into my local K3S cluster homelab with FluxCD (hopefully)

## Checklist
- [x] Build the base repository for my Resume (this one)
- [x] Setup Github Action to build, test, and deploy to [Resume](https://github.com/That1LinuxGuy/Resume)
- [x] Setup FluxCD locally and connect to my Github
- [ ] Build CI pipeline to automatically create and test a docker image of my website
- [ ] Self Host my resume and run a fallback on Github

### Thanks to:
[Hugo](https://gohugo.io/) again for allowing me to very quickly build and deploy a respectable website \
[Hugo-profile](https://github.com/gurusabarish/hugo-profile) for a very simple-to-edit, but professional looking template. Bonus points for the great documentation
