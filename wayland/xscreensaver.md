Xscreensaver is a fun screensaver application for linux made by "by [Jamie Zawinski](https://www.jwz.org/) and many others."
On X11, it runs fine, but in order to have access to all the screensavers, you have to run the following/equivalent:
```
sudo apt-get install xscreensaver xscreensaver-gl-extra xscreensaver-data-extra
```
xscreensaver runs fine on X11. However, on Wayland, it does not.\
According to [a link in the FAQ]([https://www.jwz.org/xscreensaver/faq.html#wayland](https://www.jwz.org/xscreensaver/man1.html#17)):

>THE WAYLAND PROBLEM
>Wayland is a completely different window system that is intended to replace X11. After 14+ years of trying, some Linux distros have finally begun enabling it by default. Most deployments of it also include XWayland, which is a compatibility layer that allows some X11 programs to continue to work within a Wayland environment.
>
>Unfortunately, XScreenSaver is not one of those programs.
>
>If your system is running XWayland, XScreenSaver will malfunction in two ways:
>
>1: It will be unable to detect user activity in non-X11 programs.
>
>This means that while a native Wayland program is selected, XScreenSaver will think that you are idle, and may blank the screen prematurely.
>
>2: It will be unable to lock the screen.
>
>This is because X11 grabs don't work properly under XWayland, so there is no way for XScreenSaver to prevent the user from switching away from the screen locker to another application.
>
>In short, for XScreenSaver to work properly, you will need to switch off Wayland and use the X Window System like in the "good old days".

However, as also mentioned, this will only work for so long, once distros stop allowing te use of X11, xscreensaver won't be able to be used anymore.

According to the [FAQ](https://www.jwz.org/xscreensaver/faq.html#wayland):
> Fixing this requires changes to Wayland itself, not merely to XScreenSaver.
>
>I have been warning about this problem since 2021, and wrote in some detail about the technical issues [on my blog in 2023](https://www.jwz.org/blog/2023/09/wayland-and-screen-savers/). Wayland could be improved to allow screen savers to continue to exist, but it appears that the Wayland developers have no interest in fixing this regression.

Wayland does not support screensavers. "[It does not have any provision that allows screen savers to even exist in any meaningful way.](https://www.jwz.org/blog/2023/09/wayland-and-screen-savers/)
