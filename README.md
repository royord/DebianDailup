# DebianDailup

Seems as though it has been a while since someone has endevoured to setup a dialup server in linux. There have been many changes in the underpenning in the linux subsystem since the last complete example that I can find. There are at times still need for a dial-up internet connection for specific uses and maybe this document will help someone else, once I'm able to prove that it works myself.

# Installs
```
sudo apt install mgetty mgetty-fix mgetty-voice wvdial
```

1. ```sudo apt wvdial``` will find the modem
2. ```cat /etc/wfdial.conf``` will now show your device on the line that says ```Modem = ```
3. make this file: /lib/systemd/system/mgetty.service

```[Unit]
Description=Smart Modem Getty(mgetty)
Documentation=man:mgetty(8)
Requires=systemd-udev-settle.service
After=systemd-udev-settle.service

[Service]
Type=simple
ExecStart=/sbin/mgetty -x 0 /dev/ttyACM0
Restart=always
PIDFile=/var/run/mgetty.pid.ttyACM0

[Install]
WantedBy=multi-user.target
```
4. then i needed a link:
```
sudo ln -s /lib/systemd/system/mgetty.service /etc/systemd/system/mgetty.service
```

5. then i could start the service:
```sudo systemctl start mgetty.service```

6. to make it autostart (not sure if it works at this time)
```
sudo systemctl enable mgetty.service
```

# References:
https://howtoforge.com/linux_dialin_server
https://forums.raspberrypi.com/viewtopic.php?t=132542
