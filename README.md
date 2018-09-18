# libinput wheel scroll patch
This patch multiple `discrete deltaY` with `/tmp/libinput_discrete_deltay_multiplier`

The goal is to increase **wheel scroll deltaY** perfectly in apps like chrome

## Usage
### write directly
```sh
echo 6 > /tmp/libinput_discrete_deltay_multiplier
```
> Everything ok in chrome
>
> Scroll too fast in urxvt
>
> Scroll jumps in ranger, telegram img gallery

### change with focused window

for i3, need python3, [i3ipc](https://github.com/acrisci/i3ipc-python)
```python
#!/usr/bin/env python
import i3ipc, re, mmap
r = re.compile(r'chrom|telegram|Master PDF Editor|^code$|typora',re.I)
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

## Install

aur `yay -S libinput-multiplier`

or [official build doc](https://wayland.freedesktop.org/libinput/doc/latest/building.html)

##  Workaround before

[SmoothScroll](https://chrome.google.com/webstore/detail/smoothscroll/nbokbjkabcmbfdlbddjidfmibcpneigj)

[imwheel](http://imwheel.sourceforge.net/)