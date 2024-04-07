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

5. then i could start the service

    ```sudo systemctl start mgetty.service```

7. to make it autostart (not sure if it works at this time)
    ```
    sudo systemctl enable mgetty.service
    ```

8. Turn On PPP Dial In Service
    Mgetty by default will not invoke PPP, it is commented out in the /etc/mgetty+sendfax/login.config file. We need this service so IP packets can flow across the dial-in connection. 
    
    ```sudo vi /etc/mgetty/login.config```
    
    Look for a line:
    
    ```#/AutoPPP/ - a_ppp /usr/sbin/pppd auth -chap +pap login debug```
    
    Change to
    
    ```/AutoPPP/ - a_ppp /usr/sbin/pppd auth -chap +pap login debug ```
    
    and remove the first character, the # and save the file. Notice the "debug" option on that line. This logs useful information in /var/log/messages that we will look at later. Also, the "login" option means to authenticate with the /etc/passwd file after "pap" authentication (described below). 

9. Setup PPP Options
    When PPP starts up, it reads options from the command line from /etc/mgetty+sendfax/login.config. Then it read more options from the /etc/ppp directory. We will create a new file called options.server where we will put generic options for all modems that dial in. Then we will have an options file for each modem where we can put the IP address we will assign anyone on that modem. That file will be named options.ttyS0 or options.ttyS1. 
    
    ```vi /etc/ppp/options.server```
    ```
    -detach
    asyncmap 0
    modem
    crtscts
    lock
    proxyarp                                                     
    ms-dns 1.2.3.4           #replace 1.2.3.4 with DNS address Primary                    
    ms-dns 3.4.5.6           #replace 3.4.5.6 with DNS address Slave
    ```
    
    ```vi /etc/ppp/options.ttyS0``` 
    ```
    192.168.0.12:192.168.0.100            #serverAddress:clientAdress
    netmask 255.255.255.0                    #The netmask of the LAN the server is connected to
    ```

7. Add Users To pap-secrets
    In order to allow dial in, you will have to define users and passwords to authenticate them with. PPP will authenticate them. First, we must add users and passwords to the /etc/ppp/pap-secrets file.
    ```vi /etc/ppp/pap-secrets```
    ```
    Client (User)      Server       Secret (password)         IP
    sohail               *               boby                  *
    zain                 *               zain123               *
    ```
8. Create Linux Users
    Now, create some regular linux users that correspond to the /etc/ppp/pap-secrets file. Use the same password that has been entered in that file. If you do not want to do this step then you must remove the "login" option from the command line of ppp kept in /etc/mgetty+sendfax/login.config. 

9. Turn On Routing
    We now want Linux to be a router and allow packets to flow through it. This is called packet forwarding.
    
    ```vi /etc/sysctl.conf```
    ```
    net.ipv4.ip_forward  = 1 
    sysctl -e -p /etc/sysctl.conf
    ```
10. Start Mgetty
    Tell the init to re-read its config file (/etc/inittab) and start up mgetty on the modems. 
    
    ```/sbin/telinit q```
11 Test Dial In and View Logs
    Have someone try dialing in on Phone number attached to modem and you can watch the logs live by typing this: 
    
    ```tail -f /var/log/messages```
    You will see the connection attempts and some useful debugging info.

# References:
https://howtoforge.com/linux_dialin_server

https://forums.raspberrypi.com/viewtopic.php?t=132542

https://hereirestinremorse.wordpress.com/tag/ubuntu-10-04-varlogmessages/
