# VNC-remote-screen

Making a raspberrypi + a TV into remote display for a VNC server. Based on the [minimal-wallboard](https://github.com/samuelb/minimal-wallboard).

1. Flash the image

- Download the [Raspbian Lite image](https://www.raspberrypi.org/downloads/raspbian/).
- Flash it onto the SD card.

2. Activate SSH

- place an empty file into the boot partition that is named `ssh`.

3. Configure Raspbian

- Start the Pi up and log in using SSH.
- Use `sudo raspi-config` to change settings
  - Change the default password
  - Set Locale, Timezone and keyboard-layout
  
reboot after changes are made.

4. Update

Update with `sudo apt-get update` and `sudo apt-get upgrade`.

5. Install packages

```
$ sudo apt-get install xtightvncviewer vim cec-utils xinit xserver-xorg-legacy x11-xserver-utils xfonts-scalable xfonts-100dpi xfonts-75dpi xfonts-base matchbox-window-manager
```

6. Create Xwrapper.config

`sudo vim /etc/X11/Xwrapper.config` with just a single line in it:

```
allowed_users=anybody
```

7. Create .xinitrc

`sudo vim /home/pi/.xinitrc` with following content:

```
# don't turn off display
xset dpms 0 0 0
xset -dpms
xset s noblank
xset s off

# simple fullscreen wm
matchbox-window-manager -use_cursor no -use_titlebar no &

# start VNC client
exec echo "MySecurePassword" | xtightvncviewer -viewonly -fullscreen -autopass 192.168.1.100:0
```

Adjust the password (MySecurePassword) to the one you've set in the VNC server as the View-Only password and change the IP to the one of your VNC server.

8. Create xinit-login.service

`sudo vim /etc/systemd/system/xinit-login.service` with following content:

```
[Unit]
After=systemd-user-sessions.service

[Service]
ExecStart=/bin/su pi -l -c /usr/bin/xinit -- VT08
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

`sudo systemctl daemon-reload`
`sudo systemctl enable xinit-login`
`sudo systemctl start xinit-login`


9. Disable lightdm

`sudo systemctl disable lightdm.service` 

10. Reboot

`sudo reboot` and login again.

11. Adjust config.txt

`sudo vim /boot/config.txt`, following values worked for my Samsung TV's:

```
# uncomment this if your display has a black border of unused pixels visible
# and your display can output without overscan
disable_overscan=1

# uncomment the following to adjust overscan. Use positive numbers if console
# goes off screen, and negative if there is too much border
overscan_left=24
overscan_right=24
overscan_top=24
overscan_bottom=24

# uncomment to force a console size. By default it will be display's size minus
# overscan.
framebuffer_width=1920
framebuffer_height=1080
```

Set the Screen resolution using the `framebuffer_*` values, I don' think a Rpi can handle much higher resolutions.
Adjust the borders with the `overscan_*` values.

