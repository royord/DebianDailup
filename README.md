# DebianDailup

Seems as though it has been a while since someone has endevoured to setup a dialup server in linux. There have been many changes in the underpenning in the linux subsystem since the last complete example that I can find. There are at times still need for a dial-up internet connection for specific uses and maybe this document will help someone else, once I'm able to prove that it works myself.

# Installs
```
sudo apt install mgetty mgetty-fix mgetty-voice wvdial
```

1. ```sudo apt wvdial``` will find the modem
2. ```cat /etc/wfdial.conf``` will now show your device

# References:
https://howtoforge.com/linux_dialin_server
