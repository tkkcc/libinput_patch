# libinput wheel scroll patch
This patch multiple `discrete deltaY` with `/tmp/libinput_discrete_deltay_multiplier`.
The goal is to increase **wheel scroll deltaY** perfectly in apps like chromium

## Install

aur [libinput-multiplier](https://aur.archlinux.org/packages/libinput-multiplier) or [official build doc](https://wayland.freedesktop.org/libinput/doc/latest/building.html)

## Usage

### set permission
```sh
chmod 666 /tmp/libinput_discrete_deltay_multiplier
stat -c '%a' /tmp/libinput_discrete_deltay_multiplier
```
without right permission, multiplier is always 1  
### write directly
```sh
echo 6 > /tmp/libinput_discrete_deltay_multiplier
```
> perfect in chromium, too fast in urxvt, jumps in ranger and telegram img gallery

### change with focused window

for i3wm, need python3 and [i3ipc](https://github.com/acrisci/i3ipc-python)
```python
#!/bin/python
import i3ipc, re, mmap
r = re.compile(r'chrom|telegram|Master PDF Editor|typora',re.I)
fd = open("/tmp/libinput_discrete_deltay_multiplier","r+b")
m = mmap.mmap(fd.fileno(), 0, access=mmap.ACCESS_WRITE)
def on_window_focus(i3, e):
    v = 1
    c = e.container.window_class
    if (r.search(c)):
        v = 6
    m.seek(0)
    m.write(str(v).encode())
    # print(c, v)
i3 = i3ipc.Connection()
i3.on("window::focus", on_window_focus)
i3.main()
```

##  Workaround before

[SmoothScroll](https://chrome.google.com/webstore/detail/smoothscroll/nbokbjkabcmbfdlbddjidfmibcpneigj), find and cache scrollable container, not work in some areas

[imwheel](http://imwheel.sourceforge.net/), one scroll jumps twice in page, forth in tab