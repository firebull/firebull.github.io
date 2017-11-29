---
layout: post
title:  "Клонирование диска таблицей разделов GPT с Windows 8/8.1"
date:   2015-09-10 12:31:58 +0300
categories: windows8 gpt
---

При клонировании диска с GPT-разделами с помощью Clonezilla скорее всего будет нарушена целостность таблицы разделов. Это нестрашно и легко лечится.

Симптом один - с новым диском в упор не видит загрузчик. Загрузившись с диска или флэшки с Windows 8 увидите пустой диск. Утилита diskpart также не видит разделы. Линуксовая утилита GParted выдает ошибку:
> Warning: /dev/sda contains GPT signatures, indicating that it has a GPT table.
However, it does not have a valid fake msdos partition table, as it should.
Perhaps it was corrupted -- possibly by a program that doesn't understand GPT
partition tables. Or perhaps you deleted the GPT table, and are now using an
msdos partition table. Is this a GPT partition table?

Лечится простой перезаписью таблицы разделов с помощью утилиты [gdisk](http://manpages.ubuntu.com/manpages/lucid/man8/gdisk.8.html "gdisk"):

```bash
# gdisk /dev/sda
GPT fdisk (gdisk) version 0.7.2

Warning! Main partition table CRC mismatch! Loaded backup partition table
instead of main partition table!

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

****************************************************************************
Caution: Found valid GPT with corrupt MBR; using GPT and will write new
protective MBR on save.
****************************************************************************

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT).
The operation has completed successfully.
```

Другими словами, запустить gdisk и перезаписать таблицу командой **w**.

Кстати говоря, для восстановления я использую LiveCD [Ultimate Boot CD](http://www.ultimatebootcd.com/ "Ultimate Boot CD"), на котором есть огромное количество утилит как для восстановления данных, так и клонирования дисков и тестирования.
<!--more-->
