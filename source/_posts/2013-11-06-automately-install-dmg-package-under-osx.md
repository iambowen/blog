---
layout: post
title: "Automately install dmg package under OSX"
description: ""
category: "osx"
tags: ["osx"]
---


The common way install a new app in osx is like:

1. Install from AppStore;
2. Download *.dmg file and open it then click install script;

For the App that can only be installed from *.dmg file, there's a better way to automate the install process.
The second way includes following steps:

1. Download the package-*.dmg file;
2. Mount *.dmg file to file system- e.g /Volumes/app_name
3. Install app with the installer or *.pkg file

Given we want to install [vagrant](www.vagrantup.com):
For the  first step, say `curl -O download_link` for people don't have tools like `wget`. If you have `wget`, say installed through brew `brew install wget`, you can just use `wget`.

For the second step, we can use `hdiutil` command to mount the dmg file to `/Volumes`.

For the last step, the `installer` in osx perfectly solve the issue. Command like:

```bash
sudo installer -store -pkg /Volumes/Vagrant/Vagrant.pkg -target /
```

And it would use the administrator previlidge to install the package.

For the whole install process, the only thing you need to do is type in the password. And [this](https://raw.github.com/iambowen/vagrant_wrapper/master/install.sh) is an example for fulfilling the automation. Surely it can be improved more genericly, for I only use this for vagrant session.^_^
