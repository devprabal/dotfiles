*Although I am using Void Linux, this guide should also help (as is, or, in a minimum-modified form) other Linux-based OS users to understand or perform common tweaks/tasks.*

---


## Getting the iso (Void Linux)

The default mirror is too slow for me, so I used the "New York" Tier 1 mirror listed in their official documentation.

Verified the iso (steps mentioned in their docs).

`dd` command (example explained in their docs) is good to creating the live-usb, don't forget to `sync`.

## Windows

Deleted the linux (ubuntu) partition using `diskmgmt.msc` (Ctrl-R, Run). Left the deleted partition as free-space, not created a new simple volume.

Restored windows boot mgr (and removed grub) using the method demonstrated in [this video](https://youtu.be/ZTMCKOx5Jz0).

## Installing 

`F9` is the key on HP (Pavilion) to select boot device.

Void Linux presents itself with a TUI to aid in the installation process which is pretty simple and straight-forward and is explained well in their docs. (RTFM)

The only thing to keep in check is that, since I am already dual-booting with Windows, so there already is a partition of type **EFI System** (about 200MB-1GB).

Void provides `cfdisk` to create and modify partitions. Write the changes before exiting.

I created a main partition (type linux filesystem) - 200G and swap partition (type linux swap) - 16G (for my 8GB RAM).

The next step is to modify the filesystem-type of the created partition. I just modified them appropriately (using type as ext4 and linux-swap respectively) for the two that I created - 200G and 16G. It also asks the mount point for the 200G, since it is my primary partition so I mounted it at `/`. Also, tell the mount point for the *EFI System*-type partition (206M, in my case) to be `/boot/efi`. 

It will ask whether to create a new filesystem for 

a) 200G (ext4, primary partition, linux filesystem mounted at `/`) - say yes.

b) 16G (swap, linux swap) - say yes.

c) 206M (EFI System type, mounted at `/boot/efi`) - say NO.

It separates root account (asking for a root password) and a first user account (which is already added to the `wheel` group for `sudo` powers) (asks a password).

## Enter the void

The grub, on first start of the system, didn't show an entry for Windows, however, on next system reboot, it did show (or maybe because the system was updated and so the grub entry modification was triggered by some package that got installed?) Anyways, it shows up now. 

The tty fonts are too small to be legible (HiDPI screen problems), which can be changed in `/etc/rc.conf`, though I have to still do that.

Updated the system.

Changed the mirror to the one from which the iso was downloaded (since it is fast for me). The process is explained in their docs.

Installed alacritty, bspwm, sxhkd, xorg, vim, firefox, and other necessary packages (from their repos, using `xbps`). - Pretty good and no hassle.

Created files - `~/.xinitrc`, `.Xresources`, `~/.config/bspwm/bspwmrc`, `~/.config/sxhkd/sxhkdrc` and copied their default configs. Remember to check if these files have the proper permissions or not, and the ones which should be executable are executable or not. Also, set the terminal emulator in `sxhkdrc` to `alacriity` (default config has `urxvt`).

Add `exec /usr/bin/bspwm` to the end of `~/.xinitrc` and comment out the x-programs (like xclock, twm, etc, etc) which is not needed.

## Starting the WM

`startx` from tty (This starts the X server and the client programs). Use `bspc quit` to return to this tty closing the X server and the client programs. Read more about it in the [arch wiki](https://wiki.archlinux.org/index.php/Xinit).

## Others

### Mounting an external hard drive

Follow the steps explained [here](https://linuxnewbieguide.org/how-to-mount-a-usb-stick-as-a-non-root-user-with-write-permission/).

### Date and Time

Installed `chrony` (mentioned as an alternative to `ntp` in their docs), and enabled the service (explained in the docs). The time is set correctly (as if by magic, since I had to issue no other command).

Do I need to disable the already installed `ntp` service (I don't see it listed in the running services)? Should I remove `ntp` package?

`chrony` had the following messages during installation - 

```text
chrony-4.0_2: unpacking ...
chrony-4.0_2: registered 'ntpd' alternatives group
Creating 'ntpd' alternatives group symlink: ntpd -> /usr/bin/chronyd
Creating 'ntpd' alternatives group symlink: ntpd.8 -> /usr/share/man/man8/chronyd.8
Creating 'ntpd' alternatives group symlink: ntpd -> /etc/sv/chronyd
```

Note: I also made Windows to use the UTC time (instead of localtime) (as linux uses UTC as default) by following the arch wiki for hardware clock.

### Compression utilities

`tar` will exit on a child process like `xz` since it is not already installed. Installed that and also `unzip`.

### NetworkManager

I have had no experience with `dhcpcd` which is the default for networking in void, so I installed `NetworkManager`.

For this, I read about enabling/disabling services with `runit` from their docs. (RTFM).

Then followed the instructions given in their docs for installing `NetworkManager`, disabling `dhcpcd`, enabling `dbus`, enabling `NetworkManager` service, and adding user to `network` group.

Logout and login for reflecting changes (using `groups` command) of being added to the `network` group.

Use `nmtui` or `nmcli` as usual to connect wirelessly, or via eth or usb.

### Brightness

I had used `brightnessctl` in previous Linux (ubuntu) and it worked well enough (`xbacklight` didn't work in ubuntu), so I installed it.

`brightnessctl` already packages a udev rule and installs it by default (in `/lib/udev/rules.d`), so the below steps are not required - 

~~And followed the instruction in [arch wiki](https://wiki.archlinux.org/index.php/Backlight) to create a udev rule in `/etc/udev/rules.d/backlight.rules` (I had to create `rules.d` dir there, I checked the man page of `udev` in void linux too, it said that it looks for a rule there, so it was safe to create a dir. Also the `udevd` service was running as found out by `sudo sv status /var/service/*`, so `udev` was already installed.)~~

```
ACTION=="add", SUBSYSTEM=="backlight", KERNEL=="intel_backlight", GROUP="video", MODE="0664"
```

Note that I (user **devpogi**) am already a part of the video group ( I configured it to be so during the install of void itself). 

### Sound

Installed `alsa-utils` package as instructed in the void docs. 

```bash
cat /proc/asound/modules
```

This gives me - 

```
 0 snd_hda_intel
 1 snd_hda_intel
```

I have checked that multiple applications can output the sound simultaneously, so I don't have to do any `dmix` thing.

Also, I didn't set up any configuration. The sound was ready to output as soon as the package was installed. The only thing to do was to unmute **Master** channel by `alsamixer` (which can be also done using cli-only command `amixer`, see arch wiki for details).

Then I set up keybindings for controlling (increasing, decreasing and un/mute) volume using `amixer`. [arch wiki](https://wiki.archlinux.org/index.php/Advanced_Linux_Sound_Architecture#Keyboard_volume_control) and [stackexchange](https://unix.stackexchange.com/questions/21089/how-to-use-command-line-to-change-volume) can help.

To get the volume, (to use in notification scripts, etc.), use this command, shamelessly stolen from [stackoverflow](https://unix.stackexchange.com/questions/89571/how-to-get-volume-level-from-the-command-line)

```bash
awk -F"[][]" '/dB/ { print $2 }' <(amixer sget Master)
```

### Fonts

Most of the websites use Helvetica as default font (still, apparently from MacOs). Something like ` helvR12.pcf.gz` (or `helv*.pcf.gz`) turns up on command `fc-match 'Helvetica'`. I think it is installed by default in void, however, it is not that great font. So, we can set up a rule in `~/.config/fontconfig/fonts.conf` to replace this font with another font like - 

```xml
<match target="pattern">
	 <test qual="any" name="family"><string>Helvetica</string></test>
	 <edit name="family" mode="assign" binding="same"><string>Roboto</string></edit>
</match>
```

If there are other such rules then enclose the entire `<match>` block under a `<fontconfig>` block (xml).

To test if that got changed or not run `fc-match 'Helvetica'` or `fc-match -a 'Helvetica'`.

We can also set default fonts to use for **serif**, **sans-serif**, **monospace** by adding a rule in `~/.config/fontconfig/fonts.conf` - 

```xml
<alias>
	<family>monospace</family>
	<prefer>
		<family>Hack Nerd Font</family>
		<family>FiraCode Nerd Font</family>
	</prefer>
 </alias>
 ```

To test, run `fc-match -a monospace | less` or `fc-match -a serif | less`

To see which all configs are being used run `fc-conflist`. You should see an entry like `+ ~/.config/fontconfig/fonts.conf` among others.

It appears that *Roboto* doesn't have CJK support. However, *Hack Nerd Font* and *FiraCode Nerd Font* have CJK and other unicode support but they are monospace fonts. So I needed to download a sans-serif font for unicode and CJK support. 

After installing *Noto* fonts from void repos, check if it displays it correctly with `fc-match 'Noto Sans'`. `fc-match 'Noto Serif`.

These are the fonts that I have additionally installed (these don't include any fonts that might have been downloaded as a dependency for another application) - 

- **Monospace:** `Hack Nerd Font` and `FiraCode Nerd Font` from [nerdfonts.com/](nerdfonts.com/) and placed under `~/.local/share/fonts`
- **Sans-serif:** `Roboto` and **Serif:** `Merriweather` from [google fonts](fonts.google.com) and placed under `~/.local/share/fonts`. (I found that google fonts and roboto are also available in void repos).
- **Serif, Sans-serif, monospace:** `noto-fonts-cjk noto-fonts-emoji noto-fonts-ttf noto-fonts-ttf-extra` from void repos. (You can search for them on [void list of packages page](https://voidlinux.org/packages/))
- **Serif, Sans-serif, monospace:** `liberation-fonts-ttf` from void repos.

**Note:** You can disable the `~/.config/fontconfig/fonts.conf` (by renaming it) (check `fc-conflist` after renaming to be sure) after installing *Noto* fonts from the void repos. All fonts will still be rendered properly. However, in order to set a peference for which fonts to use, you can enable `fonts.conf`.

My `~/.config/fontconfig/fonts.conf` looks like - 

```xml
<?xml version='1.0'?>
<!DOCTYPE fontconfig SYSTEM 'fonts.dtd'>
<fontconfig>
    <match target="pattern">
         <test qual="any" name="family"><string>Helvetica</string></test>
        <edit name="family" mode="assign" binding="same"><string>Noto Sans</string></edit>
    </match>
    <alias>
        <family>serif</family>
        <prefer>
            <family>Merriweather</family>
            <family>Noto Serif</family>
        </prefer>
    </alias>
    <alias>
         <family>sans-serif</family>
         <prefer>
             <family>Noto Sans</family>
             <family>Roboto</family>
         </prefer>
    </alias>
    <alias>
         <family>monospace</family>
         <prefer>
             <family>Hack Nerd Font</family>
             <family>FiraCode Nerd Font</family>
         </prefer>
    </alias>
</fontconfig>

```

**Readings:** for understanding fontconfig - 

- [lfs wiki](http://www.linuxfromscratch.org/blfs/view/svn/x/tuning-fontconfig.html)
- [arch wiki](https://wiki.archlinux.org/index.php/Font_configuration#Replace_or_set_default_fonts)
- [arch wiki again](https://wiki.archlinux.org/index.php/Fonts)
- [stackexchange](https://unix.stackexchange.com/questions/106070/changing-monospace-fonts-system-wide)
- [blogpost](https://nolanlawson.com/2020/05/02/customizing-fonts-in-firefox-on-linux/)

### Wallpaper

Use `feh` to set the wallpaper for the first time as `feh --bg-scale /path/to/file`. It will create a file `~/.fehbg`.
Edit that file for changing wallpaper and include this file (which actually is a script) in the `~/.xinitrc` (or in `~/.config/bspwm/bspwmrc, which is also a script file) as -

```bash
~/.fehbg &
```

[See arch wiki for details](https://wiki.archlinux.org/index.php/feh)

### Icons

Some good icon packs (with ongoing development) (in order of liking) - 

- [Papirus](https://www.gnome-look.org/p/1166289/) -- installed from the release on gnome-look.org.
- [Luv, from Nitrux](https://github.com/Nitrux/luv-icon-theme) -- installed from the release on their github.

Notes:

1. Use `lxappearance` (installable without any lxde desktop dependencies from void repos) to change icons. This creates and overrides the files `~/gtkrc-2.0` and `~/.config/gtk-3.0/settings.ini`. This also creates and overrides the file `~/.icons/default/index.theme` for default cursor icon theme.

2. Use `~/.local/share/icons/` to extract icons into.

### Themes

Some good themes (gtk2/3) (with ongoing development) (in order of liking) - 

- [Snow](https://www.gnome-look.org/p/1214421/) -- installed from gnome-look.org (hackable src code available on github).
- [Matcha](https://www.gnome-look.org/p/1187179/) -- installed from gnome-look.org, though it is not the latest (src code available on github) release. -- The gtk2 variant requires `gtk2-engines` and `gtk-engine-murrine`, installed them from the void repos.
- [Zuki](https://github.com/lassekongo83/zuki-themes) -- installed from github release, the one on gnome-look.org is way-too-much outdated. The *dark* themes are not working idk why. 
- [Vimix](https://www.gnome-look.org/p/1013698/) -- installed from gnome-look.org, though it is not the latest (src code availabe on github) release. 
- [Arc](https://github.com/jnsh/arc-theme) -- installed from void repos (`xbps-install arc-theme`).
- [Numix](https://github.com/numixproject/numix-gtk-theme) -- copied the release from their github. idk why Not working.
- [Materia](https://github.com/nana-4/materia-theme) -- not installed.
- ~~[wpgtk](https://github.com/deviantfero/wpgtk) -- I need to look into it.~~

**These do not have source code (sass files)** - 
- [Obsidian 2](https://www.gnome-look.org/p/1173113/)
- [Jade 1](https://www.gnome-look.org/p/1167214)
- [Prof Gnome theme](https://www.gnome-look.org/p/1334194/)
- [Nextwaita](https://www.gnome-look.org/p/1289376/)
- [Desert](https://www.gnome-look.org/p/1449286/)

Notes:

1. Use `lxappearance` (installable without any lxde desktop dependencies from void repos) to change themes. This creates and overrides the files `~/gtkrc-2.0` and `~/.config/gtk-3.0/settings.ini`.

2. See [arck wiki](https://wiki.archlinux.org/index.php/GTK) for other options that can be set in these files.

3. Qt themes are different and don't go well with the gtk themes, even icons.

4. Use `~/.local/share/themes/` to extract themes into.

### Colorschemes for vim and alacritty

Some good colorschemes for vim (but can be extended to terminal and other programs) and having both light/dark variants (mostly)

- [forest-night](https://github.com/sainnhe/forest-night), for alacritty [this](https://gist.github.com/sainnhe/6432f83181c4520ea87b5211fed27950)
- [gruvbox-material](https://github.com/sainnhe/gruvbox-material), for alacritty [this](https://gist.github.com/sainnhe/ad5cbc4f05c4ced83f80e54d9a75d22f)
- [gruvbox](https://github.com/morhetz/gruvbox)
- [iceberg](https://github.com/cocopon/iceberg.vim)
- [base16-ocean](https://github.com/chriskempson/base16-vim/blob/master/colors/base16-ocean.vim), for alacritty [this](https://github.com/aaron-williamson/base16-alacritty/tree/master/colors)
- [oceanic-next](https://github.com/voronianski/oceanic-next-color-scheme) --> The official light theme still has some issues and is not usable at the moment, need to check again.
- [two-firewatch](https://github.com/rakr/vim-two-firewatch)
- [Base2Tone-vim](https://github.com/atelierbram/Base2Tone-vim) based on [DuoTone for Atom](https://simurai.com/projects/2016/01/01/duotone-themes)
- [one-half](https://github.com/sonph/onehalf)
- [vim-one](https://github.com/rakr/vim-one)
- [solarized8](https://github.com/lifepillar/vim-solarized8)
- [papercolor](https://github.com/NLKNguyen/papercolor-theme)
- [ayu](https://github.com/ayu-theme/ayu-vim)

*I got tired of searching for good schemes and so I decided to make some, see [here](mycolorschemes.yml)*
*But tbh, selecting good, eye-pleasing colors is a pain, and I don't even know how hard creating a vimscript is for the same.*

> For more good color schemes (vim) look at 'trending' and 'top' sections of [vimcolorschemes.com](vimcolorschemes.com/)
> Nord and dracula are intentionally left out.

### File Manager GUI

Installed `pcmanfm` from void repos and set its preferences (like which terminal to use, etc.) from inside the gui app itself.

Also installed `pcmanfm-qt` from void repos (though I will uninstall it).

### Neovim

I am moving away from vim and switching to `nvim` because -

- vim by default, doesn't have support for x11 clipboard (need to install and use `vim-x11` instead of `vim` to fix it).
[read this wiki](https://vim.fandom.com/wiki/Accessing_the_system_clipboard)

[Also watch this video by Luke Smith to understand vim's registers and system registers for copy-pasting](https://www.youtube.com/watch?v=E_rbfQqrm7g&feature=youtu.be)

- vim does some very bad indentation shit when pasting a text from another application into it. A part of the solution was that I used default vim settings (by removing `.vimrc` file) but that still strips-off some of the starting characters from the pasted text. [this stackexchange answer is fine, but too much work to do a simple paste](https://unix.stackexchange.com/questions/199203/why-does-vim-indent-pasted-code-incorrectly). (Also, it too didn't solve the stripping of starting characters.

- `nvim` has some good features by default, like it can access the system clipboard (Ctrl-Shift-c to copy from nvim to system clipboard and paste by Ctrl-v in another application) (Ctrl-Shift-v to paste in nvim from another application where we copied into the system clipboard by Ctrl-c).

- `nvim` doesn't do the indentation shit and doesn't strip-off start characters from the pasted text. It also has beam cursor for insert mode and block cursor for normal mode (nice!).

I will be exploring more of neovim's features.

- Trick to comment a block of code. `Ctrl+v`, then select, then `Shift+i`, then `#`. [stackoverflow](https://stackoverflow.com/questions/1676632/whats-a-quick-way-to-comment-uncomment-lines-in-vim)

## Miscellaneous

- Use `pkill` instead of `killall` (not present by default in void).

- I installed `bc` from their repos because it wasn't there already, however, I got the following messages during install, - 

```text
... unpacking ...
bc-1.07.1_5: registered 'bc' alternatives group
Creating 'bc' alternatives group symlink: bc -> /usr/bin/gnu-bc
Creating 'bc' alternatives group symlink: bc.1 -> /usr/share/man/man1/gnu-bc.1
bc-1.07.1_5: registered 'dc' alternatives group
Creating 'dc' alternatives group symlink: dc -> /usr/bin/gnu-dc
Creating 'dc' alternatives group symlink: dc.1 -> /usr/share/man/man1/gnu-dc.1
```

What is `dc`, was it already installed? Was `gnu-bc` already installed? (I guess not. I guess `bc` provides `dc` and also sets the symlinks so that `gnu-bc` and `bc` refer to the same program, also similarly for `dc`.)

- It surprised me a lot to see that `xbps` repos already have `pfetch`, `picom`, `rofi`, `starship` and other popular programs with their latest versions. I no longer have to install them from source (like I used to do in Ubuntu - to get the latest versions).

- Installed `make` and `gcc` (which installs other important packages like binutils).

- `git` is to be installed for `vim-plug` in vim .

- firefox may say **No compatible source found for this video** for videos on some sites (like 4anime.to) and may give an error code of 102630 (video file cannot be played). Install `ffmpeg` and `libavcodec` from void repos. [source, mozilla support](https://support.mozilla.org/en-US/questions/1290087)

- [ Search void packages on web page](https://voidlinux.org/packages/) 

- `masterpdfeditor5` is available as an executable script from their [website](code-industry.net/). So downloaded that and running. But it needs `qt5` to be installed (which installs other `qt` dependencies), along with `qt5-svg`(install from void repos). It is also available as a restricted package from void repos (which needs to be built using `xbps-src`. I haven't tried that). Also it is available as a deb package from their site and you can use `xdeb` to extract from deb into xbps format for installing in void (but haven't tried that either). 

- Installed `featherpad` for a gui text editor.

- Installed `vscode` from void repos. This also installs `xdg-utils`, ~~so I guess the problems of `XDG_RUNTIME_DIR` being not set~~ (DID NOT SOLVE) and `xdg-open` (not found) will get solved. Also, note that this is `code-oss` (program name). Since I had already synced my setup on [settings sync extension](https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync), I just needed to install this extension and sign-in with my github and download my settings.

- Colors in `man` - put the below in `~/.bashrc`.

```bash
man() {
    LESS_TERMCAP_md=$'\e[01;31m' \
    LESS_TERMCAP_me=$'\e[0m' \
    LESS_TERMCAP_se=$'\e[0m' \
    LESS_TERMCAP_so=$'\e[01;44;37m' \
    LESS_TERMCAP_ue=$'\e[0m' \
    LESS_TERMCAP_us=$'\e[01;32m' \
    command man "$@"
}
```

Other programs like `ip`, `ls`, `grep`, `diff` already have a switch `--color=auto`. Use these as aliases in `~/.bash_aliases` (and then source this file in `~/.bashrc`) - 

```bash
alias ls='ls --color=auto'
```

[source arch wiki](https://wiki.archlinux.org/index.php/Color_output_in_console)

- `xtitle` should be installed for solving the floating image previews problem in telegram on bspwm. Or you will have to adapt something similar to `xtitle` with `xprop` (which must be already installed).

- Quick scratchpad in firefox - 

```html
data:text/html, <style>html,body{margin: 0; padding: 0;}</style><textarea style="font-size: 1.1em; font-family: 'FiraCode Nerd Font', monospace; line-height: 1.2em; background: rgb(40,44,52); color: rgb(220,223,228); width: 100%; height: 100%; border: none; outline: none; margin: 0; padding: 90px;" autofocus placeholder="write..." />
```

- Using `audacious` for a GUI music player. Need to install both `audacious` and `audacious-plugins`.

- Using `mpv` for video player. Additionally, installed `youtube-dl` from void repos for watching `URL` (see `man mpv`) videos in `mpv`. 

- For touchpad "tap to click" and "Natural Scrolling", make a dir `xorg.conf.d` in `/etc/X11` and inside it, make a file named `40-libinput.conf` with the following contents - 

```text
Section "InputClass"
    Identifier "Touchpad"
    MatchIsTouchpad "on"
    Driver "libinput"
    Option "Tapping" "on"
EndSection

Section "InputClass"
    Identifier "Touchpad"
    MatchIsTouchpad "on"
    Driver "libinput"
    Option "NaturalScrolling" "on"
EndSection
```
[arch wiki](https://bbs.archlinux.org/viewtopic.php?id=232862). Also see man pages for `xorg.conf`, `xorg.conf.d`, `libinput`.

- **Other programs installed** : 

	- [`bat`](https://github.com/sharkdp/bat) (modern cat clone, available in void repos)
	- 

