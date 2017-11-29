---
layout: post
title:  "Удалить старые ядра на Ubuntu"
date:   2014-11-27 10:45:16 +0300
categories: ubuntu
---

На Ubuntu, если /boot на отдельном разделе, периодически необходимо удалять старые ядра. Делать это можно одной командой:

```bash

sudo apt-get purge $(dpkg -l linux-{image,headers}-"[0-9]*" | awk '/ii/{print $2}' | grep -ve "$(uname -r | sed -r 's/-[a-z]+//')")

```

Естественно, после установки нового ядра и перезагрузки. [Источник](http://askubuntu.com/questions/89710/how-do-i-free-up-more-space-in-boot "Источник")
<!--more-->
