# The Grand Manual (Alpha)

A big list of computer/tech problems that I've faced and possible solutions.
A constant work in progress

--------

## Hardware
### VR
#### Headset keeps dropping in and out
- Plug headset into USB 3.0
- Bases up high, facing each other, diagonal

### Storage
#### Diagnosing a HDD failure as having bad sectors
##### Symptoms
- Programs using that HDD will fuck up out of the blue while you're using them
- Might log you out
- System may fail to start a few times
- Eventually it will let you start it, but it might take a while and the HDD might make scratching noises.
##### Steps
- Do a full backup of the drive using CloneZilla
    - Make sure you pick the "Expert" option and specify `--rescue` in the options. (This will ignore bad blocks)
- I'm pretty sure you shouldn't do the following steps if you're rescuing an SSD.
- On a Linux live CD or another Linux machine, run 
- **Windows/NTFS partitions:** `chkdsk`
- **Linux/ext partitions:**
    - `sudo badblocks -v /dev/sdXY > badsectors.txt` to get a list of bad sectors
    - `sudo e2fsck -l badsectors.txt /dev/sdXY` to fix/remap those sectors from that list generated in the previous step 
        - Replace `e2fsck` with `fsck` for non ext* formats 
    - Source: https://www.tecmint.com/check-linux-hard-disk-bad-sectors-bad-blocks/
        - This is kind of an old method and I think there's a better way with `smartmontools` (`smartctl`)

## Linux
### Server
#### Gitea HTTPS setup w/ nginx

Gitea is like GitLab except it doesn't use 4GB+ of fucking RAM.
Only needs 2 cores and 1GB of RAM. Works on Rasp Pi 3 apparently.
[Specs](https://docs.gitea.io/en-us/#system-requirements)

##### Install
You'll probably have to download the binary yourself and install it from there. Aren't any good packages on Debian, but on Arch there are some (I think)

Follow [this](https://www.vultr.com/docs/how-to-install-gitea-on-ubuntu-18-04)
or [this](https://docs.gitea.io/en-us/install-from-binary/).

*Ignore that site's reverse proxy config*

##### Config

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

##### Reverse Proxy
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

##### HTTPS
Once everything is working via normal `HTTP` (including reverse proxy), you can start HTTPS.

**Ignore the bullshit about setting `PROTOCOL = https` or using Let's Encrypt integration in `app.ini`**
Nginx handles all HTTPS stuff.

- Using certbot, run `certbot --nginx` as the user that installed nginx (the normal user, not nessesarily root). Will get an error about not finding nginx executable or some shit.
- Choose the option for `git.example.com`
- Choose auto-redirect (option 2)

This will alter the `git.example.com` nginx site file.

**Should work?**

#### Updating Gitea
To check your gitea version: `gitea --version`

You simply can just replace the binary at `/usr/local/bin/gitea` with a new one.

Follow [this](https://docs.gitea.io/en-us/install-from-binary/#updating-to-a-new-version).

You can get the binary from [here](https://docs.gitea.io/en-us/install-from-binary/#installation-from-binary)

### Steam

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


#### Won't start, seems a few things are missing. Seg fault.
```
libGL error: No matching fbConfigs or visuals found
libGL error: failed to load driver: swrast
A bunch of X related errors...
...
```
Install `lib32-nvidia-utils`, choose the version appropriate to your kernel.

#### Getting BeatSaber to work
- Run steam from the command line to see errors
- Install SteamVR and set it to the Linux Beta in properties
- Launch SteamVR first
- Launch BeatSaber within SteamVR
- Change the graphics settings to the lowest (I don't know which ones specifically)

##### Mods
- Install QBeat using the settings on the Github page
    - SongCore for custom songs
    - For Twitch chat intergration, find the latest Enhanced Stream Chat you can find on Google (There are a ton of forks)
    - In BeatSaber's specific Wine prefix, you need to add fonts to its fonts folder to display text

### Capture Card
- Install the V4L2 plugin for OBS (to record from /dev/video* sources)
- Add a Video Caputre Device
    - Use *anything other than* YUYV
    - Make sure frame rate is 30
- Add an Input Capture
    - Capture the input for the capture card

### NVidia
#### NVENC on OBS
- Install ffmpeg
- Install OBS
- Update NVidia proprietary drivers
- Set NVENC to be enabled in the Output settings
### Documents
#### Pandoc
##### md->pdf Table of Contents
`--toc`
##### Linebreaks being ignored in Markdown
Use `-f markdown+hard_line_breaks` argument
##### Missing character: There is no ○○○ in font...
Use `--pdf-engine=xelatex` to force it to use xelatex.
You will need to specify another font for it to use such as`-V mainfont="Noto Serif" -V mathfont="XITS Math"`
For certain annoying symbols you may need to enclose them in dollar signs and provided a supported font for the math font.
##### Export KaTeX equations enclosed with $$ in Markdown
Use `--katex` argument
##### (Xe)LaTeX is being a pain in the ass whjen converting MD to PDF
You can use other go-betweens such as via HTML. This stops some annoying font/unicode issues.
Add `-t html`, but make sure the file extention output is still `*.pdf`.
The package on Arch/Manjaro to install a HTML --> PDF converter is `wkhtmltopdf`. You need this for this to work.
### VMs
#### VMware/VirtualBox vmmon issues
_[26/3/20] Manjaro 19_
- Make sure VT-x (aka SVM) is enabled in BIOS.
- Disable Secure Boot
- Follow the [Arch Wiki guide] (https://wiki.archlinux.org/index.php/VMware#Installation) for AUR installation
  - vmware-workstation AUR package
- This AUR pacakge is pre-patched. No need for the vmware-patch AUR pacakge unless installing from website.
- The important line is: `modprobe -a vmw_vmci vmmon` after vmware is installed for the first time.
Should be a similar story for VirtualBox...
##### Useful services:
  - `vmware-networks.service` for guest network access
  - `vmware-usbarbitrator.service` for connecting USB devices to guest
  - `vmware-hostd.service` for sharing virtual machines
### Server related
#### PHP Imagick not found
Install imagick via:
`sudo apt install libmagickwand-dev imagemagick`
`sudo apt install php-pear`
`sudo pecl install imagick`

Make sure all permissions are set properly
#### SFTP seup
With a working ssh, add the follwoing line to `/etc/ssh/sshd_config`
`Subsystem	sftp	/usr/lib/openssh/sftp-server`

## Windows 10
### Python
#### pip `[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1108)` error when installing
_[26/3/20]_

Make sure `requests` is installed first via `pip install requests`.

## Cross-platform

