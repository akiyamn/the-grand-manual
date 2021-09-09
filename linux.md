# Linux

## General Desktop

### Auto detect and mount USBs (in Arch)

Surprisingly in a lot of more stripped down GNU/Linux distros USB drives aren't automatically mounted.

`udisks2` is a program which manages USB drives and `udiskie` uses that program to automatically mount and label removable drives.

```bash
sudo pacman -Ss udisks2 udiskie
```

#### On startup

You need to enable `udisks2` and run `udiskie` when your xsession (WM, DE etc...) starts.
A place to put this could be `~/.xinitrc`, the application startup menu on your DE or the run-on-start script for your standalone WM.

`sudo systemctl enable --now udisks2`
then put the sh line `udiskie` in your run-on-startup script/application.

### Alacritty colour emoji
Alacritty by default only allows you to have one monospace font.
To get colour emoji, you would need to have a backup font provided such as `Noto Emoji Color`. Alacritty does not have this functionality.

First, make sure you install the `Noto Color Emoji` font (or something else that contains colour emoji). On Arch, install: `noto-fonts-emoji`

In your Alacritty config (`~/.config/alacritty/alacritty.yml` for me) change:
```yaml
font:
  normal:
    family: monospace
```
`monospace` just means "use the default font (which ever is the best match for a given character.)"

To see what your system interprets `monospace` as, run:
```bash
fc-match -c monospace
```
This list is sorted in order of priority. It starts from the top and works its way down.
You may notice a colour emoji font is at the bottom of the list.
This can be changed by editing: `~/.config/fontconfig/fonts.conf` and adding the following between `<fontconfig>` tags.

```xml
<alias>
  <family>monospace</family>
    <prefer>
      <family>Mononoki</family> <!-- your favourite font here -->
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
    </prefer>
</alias>
```

Run `fc-cache` to reset the system fonts and restart Alacritty.

Try `echo 💩`.
   
### Qtile

#### Qtile groups not in same order as screens.
Set the monitor you want ot be the first group as the primary monitor.
Rearrange the HDMI/DP cables to order the rest of the screens.


### SDDM autologin
Config is in `/etc/sddm.conf`
Could also be in `/etc/sddm.conf.d/autologin.conf`

Change or add the following lines:
```
User=usernamehere
Session=something.desktop
```

The list of WM/DE names is in `/usr/share/xsession` as filenames.


### Emacs

#### Emacs is almost completely transparent for no reason with picom
For some reason, when only using picom on some computers emacs is like, 10% opacity for some reason.
None of the config options in `~/.config/picom.conf` seemed to work for me.

Add the following snippet into `~/.emacs.d/init.el`:

```elisp
 (set-frame-parameter (selected-frame) 'alpha '(97 . 97))
 (add-to-list 'default-frame-alist '(alpha . (97 . 97)))
```

Each of the numbers represent a percentage of opacity. The first represents the level when focused, the latter when unfocused.
I've set both to 97%.

### Printing

#### Printing a bunch of blank pages or other printing issue
CUPS (a universal printing service for Linux) need to be installed and enabled to print properly.

`sudo pacman -S cups`

The Arch Wiki has more information about installing cups.

CUPS needs to also be running and enabled otherwise wacky shit may happen, such as constantly printing blank pages etc...
When using just a WM (as opposed to a desktop environment (DE) such as XFCE or KDE), certain things such as CUPS drivers may not be enabled by default.

**To enable:**
```bash
sudo systemctl enable --now cups.service
```
This should also enable `cups.socket` and `cups.path`.
In case it doesn't repeat the above step two separate times with both of those each.

This should make CUPS run now on startup.

### Japanese (Or other non Latin) IME Input

#### Setup
Generally you need an IME (such as IBus or Fcitx) to switch between keyboard layouts and input methods.
The actual input methods for a given language are separate from the IME. You need to install both in order to get the full functionality.
On desktop environments (KDE, XFCE, etc...) sometimes it is already included and can be fully setup via GUI. Gnome ships IBus by default.
When using only a WM, things can be more difficult.

Make sure a font for your target language is installed in the system. `noto-fonts` contains characters for heaps of languages.
More fonts at: https://wiki.archlinux.org/title/Localization

Pick a combination on the table located at: https://wiki.archlinux.org/title/Input_method#IBus .
A common IME is IBus and IBus+Anthy (via the `ibus-anthy` package) is a common pairing for Japanese input.

#### Environment Variables

Once that is installed, make sure the following environment variables are set on start up:
```
GTK_IM_MODULE=ibus
QT_IM_MODULE=ibus
XMODIFIERS=@im=ibus
```
The Arch wiki recommends `/etc/environment` but that didn't work for me.

You can export the variables using bash as follows:
```bash
export GTK_IM_MODULE='ibus'
export QT_IM_MODULE='ibus'
export XMODIFIERS=@im='ibus'
```

This can go in the following locations.
`~/.xinitrc` should work for starting with startx
`~/.xprofile` should work for starting with a login manager (e.g. LightDM)
`~/.bashrc` works as a last resort. Only works if bash is your user's prompt.
There are probably better places to put these declarations

Logout and back in to check if these changes took affect.
You can check to see if it loaded the variables using the `env` command. They should be listed.

#### Adding Anthy to IBus

Ibus can be started as a daeman as follows:
`ibus-daemon -drxR`
It should appear in your system tray if you have one. Right click it and go to "Preferences".
*(If not, try running `ibus-setup`.)*

Under "Input Method" click "Add" go into your desired language (for me, Japanese) and click on your IME (for me, Anthy).
Then close the settings menu.

`CTRL+Space` changes the input method. It should all work.
*If Anthy looks like it's just typing latin letters, make sure "Latin" isn't selected in the "Input Method"*

##### IBus only types into *some* programs.
Make sure that the environment variables are set properly in the above section.
For me, I had to put them in my `bashrc`.


### Default page size for printing

Enter the name of the page size you want in the file `/etc/papersize`.

For a list of page sizes and other info view `man papersize`.

### Pacman "unknown key trust" or something like that
When trying to update or download a package, one of the gpg keys might not be trusted. (i.e. you don't have it in youir bank of keys)
Firstly, try `pacman-key --refresh`. This probably won't work, complaining about not finding the keyserver or something.

- `pacman -Sy archlinux-keyring` *might* work.

**The best way is to manually add the key you need.**

#### Finding the public key
Take note of the name of the key holder which isn't trusted when you try to use pacman. E.g. `Yui Hirasawa <yhirasawa@archlinux.org>`

You can either:

- Search for this guy's public key using a normal search engine
- OR `pacman-key -l hirasawa` on a different machine and take note of the long hex string "pub" line.
  
Arch Linux lists all of thier trusted key staff on https://archlinux.org/people/trusted-users/. If they're on this list, they are legit and thier public key is listed under "PGP Key"
Copy at least the first 8 characters (excluding the `0x`). More chars the better though.

#### Finding a working key server
Arch/Manjaro needs to download the key from somewhere. By default, it should choose a working server for you. But this never seems to work for some reason and no-one on the Internet seems to acknowledge this.

**Here's a big list of key servers: https://sks-keyservers.net/overview-of-pools.php**

The best one for Australians is: `oc.pool.sks-keyservers.net` but apparently it isn't "stable" (of course not)

Another one that worked for me is: `na.pool.sks-keyservers.net`. `eu.` exists too.

#### Adding the key
Once you have the public key, run the following command with one of the servers:
`sudo pacman-key --keyserver <SERVER URL> --recv-keys <PUB KEY>`

E.g.: `sudo pacman-key --keyserver oc.pool.sks-keyservers.net --recv-keys 19AF4A9B`

If it looks like there where no errors, try to update or install that packagae that you were trying to do earlier.

### Pacman is properly messed up

A lot of good troubleshooting tips are on the Arch wiki. [](https://wiki.archlinux.org/title/pacman#Troubleshooting)

Sometimes package maintainers change the names/versions/dependencies of packages. This causes your database to become out of date and fail to update.

Often `pacman -Sy` solves a lot of headaches.

If no mirror services are found, run: `pacman-mirrors -c YOUR_COUNTRY_NAME` to generate new mirrors. You will need to run `pacman -Syu` afterwards.

#### "Failed to commit transaction (conflicting files)" error
This means that pacman detects a file which is in conflict between multiple packages or versions of a package.

Test what package owns it with: `pacman -Qo /path/to/file`

If only one package owns it (the one you're trying to upgrade) then you can delete those files or rename them. Then try to update again.

## Server

### Static IP in command line

At the bottom of `/etc/dhcpcd.conf`:

```bash
interface <interface name>
inform <static ip>
static routers=<default gateway>
static domain_name_servers=<DNS IP>
static domain_search=<Alt DNS IP>
```

Pretty sure `dhcpcd` needs to be enabled and running


### PHP Imagick not found
Install imagick via:
`sudo apt install libmagickwand-dev imagemagick`
`sudo apt install php-pear`
`sudo pecl install imagick`

Make sure all permissions are set properly

### SFTP seup
With a working ssh, add the follwoing line to `/etc/ssh/sshd_config`

### Gitea HTTPS setup w/ nginx

Gitea is like GitLab except it doesn't use 4GB+ of fucking RAM.
Only needs 2 cores and 1GB of RAM. Works on Rasp Pi 3 apparently.
[Specs](https://docs.gitea.io/en-us/#system-requirements)

#### Install
You'll probably have to download the binary yourself and install it from there. Aren't any good packages on Debian, but on Arch there are some (I think)

Follow [this](https://www.vultr.com/docs/how-to-install-gitea-on-ubuntu-18-04)
or [this](https://docs.gitea.io/en-us/install-from-binary/).

*Ignore that site's reverse proxy config*

#### Config

Gitea's config is in `/etc/gitea/app.ini`. It gets you to autoconfigure it on the first time you load up the Gitea server in a browser. You can just delete this config to access that autoconfig again. (Vise-versa to avoid it)

Important lines in `app.ini`:
Under `[server]`
```ini
DOMAIN           = git.example.moe
HTTP_PORT        = 3000
ROOT_URL         = https://git.example.moe:3000
```

Add this to the config for dark mode:
```ini
[ui]
DEFAULT_THEME = arc-green
```

The [Arch Wiki](https://wiki.archlinux.org/index.php/Gitea) has a lot of good info on Gitea as well.

Also here's a [cheatsheet](https://docs.gitea.io/en-us/config-cheat-sheet/).

We're going to have the reverse proxy redirect the plain URL to the service running on port 3000 (i.e. the Gitea server).

#### Reverse Proxy
Use this config in `/etc/nginx/sites-available/git.example.com`:

```nginx
server {
    listen 80;
    server_name git.example.com;

    location / { # Note: Trailing slash
        proxy_pass http://localhost:3000/; # Note: Trailing slash
    }
}
```

Enable the config by linking the file in `sites-enabled` using `ln -s` (Maybe `-s` isn't needed? Use it to be safe.)
Make a backup of this config if it works because Certbot will overwrite it when HTTPS is set up.

#### HTTPS
Once everything is working via normal `HTTP` (including reverse proxy), you can start HTTPS.

**Ignore the bullshit about setting `PROTOCOL = https` or using Let's Encrypt integration in `app.ini`**
Nginx handles all HTTPS stuff.

- Using certbot, run `certbot --nginx` as the user that installed nginx (the normal user, not nessesarily root). Will get an error about not finding nginx executable or some shit.
- Choose the option for `git.example.com`
- Choose auto-redirect (option 2)

This will alter the `git.example.com` nginx site file.

**Should work?**

### Updating Gitea
To check your gitea version: `gitea --version`

You simply can just replace the binary at `/usr/local/bin/gitea` with a new one.

Follow [this](https://docs.gitea.io/en-us/install-from-binary/#updating-to-a-new-version).

You can get the binary from [here](https://docs.gitea.io/en-us/install-from-binary/#installation-from-binary)

## Steam

### AoE III (original) using wine
Working with wine 5.19

It should run out of the box, except with some sounds missing. AoE III is a 32-bit game and requires 32 bit drivers/libraries to function.

It should tell you what libraries are missing, which should have an analogue on Linux systems as a 32 bit package.
I don't remember exactly what they were but some that may be involved are:
```
lib32-nvidia-450xx-utils
lib32-libvdpau
lib32-libva-mesa-driver
lib32-libva-intel-driver
lib32-gst-plugins-base, base-libs and good
```


### Won't start, seems a few things are missing. Seg fault.
```
libGL error: No matching fbConfigs or visuals found
libGL error: failed to load driver: swrast
A bunch of X related errors...
...
```
Install `lib32-nvidia-utils`, choose the version appropriate to your kernel.

### Getting BeatSaber to work
- Run steam from the command line to see errors
- Install SteamVR and set it to the Linux Beta in properties
- Launch SteamVR first
- Launch BeatSaber within SteamVR
- Change the graphics settings to the lowest (I don't know which ones specifically)

#### Mods
- Install QBeat using the settings on the Github page
    - SongCore for custom songs
    - For Twitch chat intergration, find the latest Enhanced Stream Chat you can find on Google (There are a ton of forks)
    - In BeatSaber's specific Wine prefix, you need to add fonts to its fonts folder to display text

## Capture Card
- Install the V4L2 plugin for OBS (to record from /dev/video\* sources)
- Add a Video Caputre Device
    - Use *anything other than* YUYV
    - Make sure frame rate is 30
- Add an Input Capture
    - Capture the input for the capture card

## NVidia
### NVENC on OBS
- Install ffmpeg
- Install OBS
- Update NVidia proprietary drivers
- Set NVENC to be enabled in the Output settings

### Save screen settings on proprietary NVidia drivers
Xorg only looks at one file for the screen configuration.
This is not nessesarily `/etc/X11/xorg.conf` as it normally would be for some reason with these proprietary drivers.

Where it's looking is shown by typing:
```bash
sudo mhwd-gpu
```
For my system it was `/etc/X11/mhwd.d/nvidia.conf`.

Use `sudo nvidia-settings` (need root privileges to save to the right location) to change the display settings.
Save the output file to the path found before.


## Documents
### Pandoc
#### md->pdf Table of Contents
`--toc`
#### Linebreaks being ignored in Markdown
Use `-f markdown+hard_line_breaks` argument
#### Missing character: There is no ○○○ in font...
Use `--pdf-engine=xelatex` to force it to use xelatex.
You will need to specify another font for it to use such as`-V mainfont="Noto Serif" -V mathfont="XITS Math"`
For certain annoying symbols you may need to enclose them in dollar signs and provided a supported font for the math font.
#### Export KaTeX equations enclosed with $$ in Markdown
Use `--katex` argument
#### (Xe)LaTeX is being a pain in the ass whjen converting MD to PDF
You can use other go-betweens such as via HTML. This stops some annoying font/unicode issues.
Add `-t html`, but make sure the file extention output is still `*.pdf`.
The package on Arch/Manjaro to install a HTML --> PDF converter is `wkhtmltopdf`. You need this for this to work.
## VMs
### VMware/VirtualBox vmmon issues
_[26/3/20] Manjaro 19_
- Make sure VT-x (aka SVM) is enabled in BIOS.
- Disable Secure Boot
- Follow the [Arch Wiki guide] (https://wiki.archlinux.org/index.php/VMware#Installation) for AUR installation
  - vmware-workstation AUR package
- This AUR pacakge is pre-patched. No need for the vmware-patch AUR pacakge unless installing from website.
- The important line is: `modprobe -a vmw_vmci vmmon` after vmware is installed for the first time.
Should be a similar story for VirtualBox...
#### Useful services:
  - `vmware-networks.service` for guest network access
  - `vmware-usbarbitrator.service` for connecting USB devices to guest
  - `vmware-hostd.service` for sharing virtual machines
