# libinput wheel scroll patch

let `deltaY` x `/tmp/libinput_discrete_deltay_multiplier` to increase **wheel scroll distance**

## Install

aur [libinput-multiplier](https://aur.archlinux.org/packages/libinput-multiplier) or [official build doc](https://wayland.freedesktop.org/libinput/doc/latest/building.html),
and for ubuntu 16.04 especially
```sh
brew install meson ninja # system's meson 0.29.0 fail
sudo apt install -y git gcc g++ pkg-config check libmtdev-dev libudev-dev libevdev-dev libwacom-dev xserver-xorg-input-libinput

v=1.16.5 # if you don't use tablet, higher version is ok by appending "-D libwacom=false" to meson
wget https://www.freedesktop.org/software/libinput/libinput-$v.tar.xz
tar xf libinput-$v.tar.xz
cd libinput-$v
wget https://raw.githubusercontent.com/tkkcc/libinput_patch/ubuntu/multiplier.patch
patch -Np1 -i multiplier.patch
 
ln -s /usr/bin/pkg-config /some/path/prior/to/brew # use system's pkg-config instead of brew's
meson --prefix=/usr build/ \
        -D tests=false \
        -D documentation=false \
        -D debug-gui=false
ninja -C build/
sudo ninja -C build/ install
```

## Usage

### write directly
```sh
echo 6 > /tmp/libinput_discrete_deltay_multiplier
```
> perfect in chromium, too fast in urxvt, jumps in ranger and telegram img gallery

### change with focused window

for i3wm, need python3 and [i3ipc](https://github.com/acrisci/i3ipc-python)
```python
#!/usr/bin/env python
import i3ipc, re, mmap
r = re.compile(r'chrom|telegram|Master PDF Editor|typora',re.I)
fd = open("/tmp/libinput_discrete_deltay_multiplier","r+b")
m = mmap.mmap(fd.fileno(), 0, access=mmap.ACCESS_WRITE)
def on_window_focus(i3, e):
    v = 1
    c = e.container.window_class
    if not c:
        return
    if (r.search(c)):
        v = 6
    m.seek(0)
    m.write(str(v).encode())

i3 = i3ipc.Connection()
i3.on("window::focus", on_window_focus)
i3.main()
```

for sway, you should stick with official libinput, and use builtin `scale_factor` option.

```python
#!/usr/bin/env python
import i3ipc, re
r = re.compile(r'chrom|telegram|Master PDF Editor|typora',re.I)
def on_window_focus(i3, e):
    c = e.container.app_id
    if not c:
        return
    v = 1
    if r.search(c):
        v = 6
    i3.command(f"input type:pointer scroll_factor {v}")

i3 = i3ipc.Connection()
i3.on("window::focus", on_window_focus)
i3.main()
```

## Issues

1. scroll on tensorboard(<1.15.0) graph not well, try [roughscroll](https://greasyfork.org/en/scripts/36257-roughscroll)
2. need to manually remove `/tmp/libinput_discrete_deltay_multiplier` when switching X graphics driver from intel to nvidia

##  Workaround before

see [Chromium_low_scroll_speed](https://wiki.archlinux.org/index.php/Chromium#Chromium_low_scroll_speed)
