# Linux

## General Desktop

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

### Default page size for printing

Enter the name of the page size you want in the file `/etc/papersize`.

For a list of page sizes and other info view `man papersize`.

### Pacman "uknown key trust" or something like that
When trying to update or download a package, one of the gpg keys might not be trusted. (i.e. you don't have it in youir bank of keys)
Firstly, try `pacman-key --refresh`. This probably won't work, complaining about not finding the keyserver or something.

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

## AoE III (original) using wine
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
