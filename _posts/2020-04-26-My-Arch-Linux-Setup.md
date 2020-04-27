# My Arch Linux Setup

I've set up my Arch Linux environment for over a year now and I haven't made significant changes to it due to the fact that all of my work is through my company's Macbooks (with SSH into remote compute resources, of course). I figured since the next few C++ blogs I'll be writing are going to be done with `gcc` instead of `clang`, and I will be dissecting the C++ into assembly using `objdump` instead of `otool`, I'll be using my workstation more often.



This blog is meant to illustrate how I set up my Arch Linux environment **after** I've set up the appropriate UEFI, swap, and filesystem partitions. I'm not going to explain how boot loaders work, how to set up Pacman (the package manager for Arch), setting up locale, etc. With that said, let's dive in!

## Preliminary

Throughout this blog, I'll be using `pacman` to install bare necessities, and `yay` otherwise. To install `yay`, simply run the following:

```
archlinux% git clone https://aur.archlinux.org/yay.git
archlinux% cd yay
archlinux% makepkg -si
```

# `i3`

What is `i3`? It's a **tiling window manager** that allows you to place windows efficiently on your screen, allow simple navigation through keystrokes, separating windows into workspaces, and much more. Below is a screenshot of my setup:

![screenshot1]({{ site.url }}/assets/arch_screen.png)

As you can see, I have multiple terminal windows open, as well as Spotify, neatly tiled into bifurcated windows (as I'll explain later, you can adjust the size of the windows beyond the fixed $\frac{1}{2^n}$ sizes as well). In addition, you can see the status bar above, showing the metrics on the top right and workspace on the left. We'll explain how to set this up.

## Installing `i3`

So there are two versions of `i3` as far as I know. The windows manager I'm using has gaps in between the processes and is called `i3-gaps`, and the default one is called `i3-gaps`. Install either of these with `pacman -S <i3 package>`. `i3` prompts you to generate a default config file. If you decide to go this route, it'll ask you to set the modifier key to Alt or the super key (the key itself says "Win", "Command", or some random logo). I set it to Alt. Here are some things you can do with the `i3` config to make it aesthetic but still functional:

You can add gaps to the windows:

```
TODO
```



### Setting Backgrounds

You can set a background image using `feh`. First install an image viewing tool with `sudo pacman -S feh`, and use the below command to set a background for `i3`:

```
TODO
```

I save all of my backgrounds in my github repo [here](https://github.com/OneRaynyDay/wallpapers).

### Using a File Manager

If you want to be a supreme `vim` overlord and have bindings on everything, I suggest using `ranger`, installable via `sudo pacman -S ranger`. If you want to just have a typical gnome-like experience, try `SpaceFM` or `PCManFM`, which are two great GUI file managers. Add this into your config if you want to trigger `ranger` with hotkeys:

```

```

We want to have image preview for `ranger`, since by itself it's fairly lightweight. To do this, `sudo pacman -S w3m terminator` and add the following file into the ranger config file:

```
# i3 ❯ echo ~/.config/ranger/rc.conf
set preview_images true 
```

TODO PICTURE FOR RANGER

### Reading PDF's

In the theme of `vim` overlord, install `zathura` if you want to have a nice pdf viewer with vim-like bindings with `sudo pacman -S zathura`. I like to alias it to `pdf` because who would remember to write `zathura` when they want to read a pdf?

```

```

Some things I found useful: `s` for fitting to the width of the document, `r` for rotating documents, `d` for two-page view.

TODO PICTURE FOR ZATHURA

### Spotify

Everyone listens to music via spotify now, so let's streamline the operation by setting the last workspace to have spotify upon start-up. To do this, you need the spotify client: `sudo yay -S spotify`. This is what we add into the config file:

```

```

 ### Program Launcher (`dmenu` replacement)

`dmenu` is an easy way for users to run applications without having to go into the directory it lives in, and executing it like an uncultured savage. However, it's really ugly. Let's use a `dmenu` replacement like `rofi` by downloading via `sudo pacman -S rofi`, and we set the following config:

```

```

TODO FIGURE FOR ROFI

### Terminal Emulator

We want to install `urxvt`, which is a terminal emulator. It's aesthetic because we can change the opacity of the terminal itself, has nice color support, and has multiple font types it can support. To do this, run `sudo pacman -S rxvt-unicode`. To verify this, simply run:

```
archlinux% echo $TERM
rxvt-unicode-256color
```

You should not yet have a `~/.Xresources` file, and that's fine. Below are some things I added into mine:



Note that if you want to use the same font (powerline family fonts), install it using `sudo yay -S powerline-fonts-git`.

###  Login Manager

We want to have a sexy login manager to greet us for logins. Let's use `lightdm` along with its greeter in the Aether theme by installing `sudo pacman -S lightdm lightdm-webkit2-greeter lightdm-webkit-theme-aether`. No need to change anything in `~/.config/i3/config`.



