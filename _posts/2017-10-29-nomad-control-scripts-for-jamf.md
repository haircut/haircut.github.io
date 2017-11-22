---
layout: post
title: NoMAD Control Scripts for Jamf
tags: [jamf, scripts, nomad, python]
feature-img: "assets/img/feature-nomad-control-scripts.png"
---
[NoMAD](https://nomad.menu/) is a _fantastic_ utility to bridge the gap between your enterprise directory accounts and endpoint devices. It allows your Mac to behave like a directory-bound client without all the headache.

I've written a handful of Python scripts to help manage the application on your clients.

<!--more-->

- `nomad-add-launchagent.py`: creates the NoMAD LaunchAgent
- `nomad-load-launchagent.py`: loads an existing NoMAD LaunchAgent
- `nomad-pre-update.py`: unloads NoMAD LaunchAgent and quits NoMAD prior to installing an updated version

These scripts are designed to be used in Jamf Pro policies. I've separated the functionality for different use cases
and flexibility. The `...add...` and `...load...` file naming convention ensures the scripts will run in the correct
order since Jamf Pro runs scripts alphabetically.

## Scripts

### nomad-add-launchagent.py

<script src="https://gist.github.com/haircut/108d3b8bd402cc56aa1f65df54f3b7eb.js?file=nomad-add-launchagent.py"></script>

## Example Jamf Pro policies

### Install Policy

- Package: `NoMAD.pkg`
- Script: `nomad-add-launchagent.py` **After**
- Script: `nomad-load-launchagent.py` **After**

### Update Policy

- Script: `nomad-pre-update.py` **Before**
- Package: `NoMAD.pkg`
- Script: `nomad-load-launchagent.py` **After**