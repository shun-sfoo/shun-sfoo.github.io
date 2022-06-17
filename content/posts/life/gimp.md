---
title: 'Gimp'
date: 2022-06-17T14:50:02+08:00
draft: false
---

## build gimp

### install dependencies

`git clone https://aur.archlinux.org/gimp-develop-git.git`

to see the dependencies

```bash
sudo pacman -S lcms2 libwmf icu enchant libgexiv2 librsvg desktop-file-utils libexif libart-lgpl dbus-glib gtk-doc poppler-glib  \
poppler-data openexr mypaint-brushes1 babl gegl cairo appstream-glib gobject-introspection intltool alsa-lib libxslt  \
glib-networking ghostscript libxpm webkit2gtk libheif libwebp libmng iso-codes aalib zlib gjs python-gobject \
luajit meson xorg-server-xvfb
```

### TODO

learn the meson build process
