---
layout: post
title:  "Ошибка запуска npm в Ubuntu"
date:   2015-04-20 14:30:10 +0300
categories: nodejs npm ubuntu
---

Часто при запуске npm в Ubuntu возникает ошибка:

```bash
/usr/bin/env: node: Нет такого файла или каталога
```

Лечится созданием линка:

```bash
ln -s /usr/bin/nodejs /usr/bin/node
```
<!--more-->
