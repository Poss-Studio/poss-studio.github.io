# 这里是你们的Rexim喵~!🐱
# --发行版 -- 
Archlinux 1台, NixOS一台,FreeBSD一台
# 关于我的linux内核
linux-zen linux-cachyos,早些时候intel还没有放弃clear linux的时候还用过linux-clear(真的挺喜欢的)
# 关于窗口管理器
其实我个人并不需要桌面环境,对于我一个重度编程爱好者来说,拥有一个稳定的工作区更重要喵~!
我在这方面折腾过很长时间:
i3wm --> swaywm(i3wm的wayland实现,开发者比较保守) --> river-classic(现在正在使用喵~!)  相当好用
我私下的探索dwm,好像还有一个深度定制的cwm也挺好,dwm的wayland实现dwl,中期在codeberg上短暂出现开发者缺口
Niri 和hyprland都试过,动画很好看,niri的操作逻辑我个人不是很习惯,所以我还是用回了river-classic
现在river已经变成了一个纯粹的wayland合成器了,不提供窗口管理,river-classic相对于i3wm,sway等并不开箱即用
# 关于窗口管理器的微调和依赖软件包
mako -> 用来提示通知的
wlroots0.20 ,pixman,tllist-> doc相关基础依赖
i3status | dam -> 适用于river-classic的状态栏(很Suckless)
swaybg -> sway继承下来的用来设置壁纸
gammastep ->用来设置夜间模式
fcitx5,fcitx5-im,fcitx5-chinese-addons -> linux下除了ibus的中文输入法解决方案
wmenu -> 符合Unix管道哲学的应用程序启动器(过滤器) -< 万能的模糊搜索
foot -> 利用率相当高的终端,用cpu渲染,极致的算法,常采用C\S架构
imv -> Unix下标准图片查看器
mpv -> 媒体播放器（Unix标准播放器）
cava -> 用于显示音符变化
pipewire -> plauseaudio自从2000年以来真的太古老了,pipewire是一个很好的替代方案
waybar,quickshell,noctalia-shell,dms-shell,dam+i3status(现在在用)
关于river-classic以及dwl,dwm下调整分辨率问题:
dwm是基于x11的,距今约10年左右没有更新了,还是用老旧的xorg-xrandr解决
dwl是dwm的wayland实现,river-classic也是在wayland下实现的