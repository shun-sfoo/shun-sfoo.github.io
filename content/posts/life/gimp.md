---
title: 'Gimp'
date: 2022-06-17T14:50:02+08:00
draft: false
---

## build gimp

run `./autogen.sh` show the dependencies

```bash
make
# ERROR: if it error python module giscanner not found
# https://archlinux.org/packages/extra/x86_64/gobject-introspection/files/
# the python file in /usr/bin
# reboot could save the problem
# if not `export PATH="/usr/bin:$PATH"`
sudo make install
```

## remove

```bash
make clean
make distclean
```

## README

more information read the INSTALL.in in source
