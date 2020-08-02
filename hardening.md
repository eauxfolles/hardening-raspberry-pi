Hardening Raspberry Pi 4
========================

v4 @ 30.05.2020


Content
=======

1. Users & Passwords
2. Disable WiFi & BT
3. Remote Access via SSH
4. Auto-Update
5. iptables (Firewall)


**1) Users & Passwords** 
====================

Goal is to change standard credentials (pi:raspberry) and "deactive" default user. New user will be created with admin-rights. To increase security, sudo-command will be set to require password.

* Change password for user "pi": ```passwd``` (or during installation)
* Create new user and set password: ```sudo adduser < user >```
* Set admin rights to new user: ```sudo usermod -aG adm,sudo < user >```
* Change to new user: ```su < user >```
* Check for sudo-privileges: ```sudo su root```
* Deactive password for user "pi": ```sudo passwd --lock pi```
* Make sudo require password: ```sudo visudo /etc/sudoers.d/010_pi-nopasswd``` and change/add:
> pi ALL=(ALL) PASSWD: ALL

> < user > ALL=(ALL) PASSWD: ALL

Source:
https://www.raspberrypi.org/documentation/configuration/security.md


**2) Disable WiFi & BT**
====================

Access to Raspberry Pi is planned via Ethernet. Wireless and Bluetooth will be not required and therefore deactivated. 

Edit file **/boot/config.txt** and add: 
> dtoverlay=disable-wifi

> dtoverlay=disable-bt

Source:
https://github.com/raspberrypi/firmware/blob/master/boot/overlays/README


**3) Remote Access via SSH**
========================

Raspberry Pi should be used in headless mode. Connection will be allowed via SSH. To increase security, SSH will be only possible via new user and key pairs, no password-authentication. 

* Change "Port 22" to <random number> (49152–65535): ```nano /etc/ssh/sshd_config```
* Activate SSH: ```sudo raspi-config``` (select "Interfacing Options", "SSH" and enable it)
* Restart SSH: ```sudo service ssh restart```
* Generate key-pair: ```ssh-keygen``` (keep suggested directory)
* Authorize key in folder ~/.ssh: ```less id_rsa.pub >> authorized_keys```
* Login (via SSH and credentials, not yet keys) from main system to Raspberry Pi and copy id_rsa-file
* Delete private key on Raspberry Pi: ```rm id_rsa``` (while in folder ~/.ssh)
* Import key into e.g. PuTTY and configure for usage of private key
* Disable password authentication: ```sudo nano /etc/ssh/sshd_config``` (enable/change "PasswordAuthentication" to "no")
* Restart SSH: ```sudo service ssh restart```

Source:
https://www.raspberrypi.org/documentation/configuration/security.md


**4) Auto-Update**
==============

Cron-job will be created to automatically update and upgrade Raspberry Pi (including SSH). This will ensure patches are installed. Log will be created (overwritten) after each run. 

Create bash script-file < file >.sh in folder /home/< user > and add: 
* ```sudo apt-get update -y```
* ```sudo apt-get dist-upgrade -y```
* ```sudo apt-get autoremove -y```
* ```sudo apt-get autoclean -y```
* ```sudo apt install openssh-server -y```
* ```sudo reboot```

Render < file >.sh executable: 
* ```sudo chmod +x < file >.sh```

Create folder to save logs (in folder /home/< user >): 
* ```mkdir log```

Type ```sudo crontab -e``` into terminal, select preferred editor and add following for daily execution (at midnight): 
* 0 0 * * * /home/< user >/< file >.sh > /home/< user >/log/< file >.log 2>&1

Source: 
https://www.instructables.com/id/Raspberry-Pi-Auto-Update/


**5) iptables (Firewall)**
======================

Setup persistent rules for Firewall to block all incomming traffic apart from defined system. 

Install persistent iptables: 
* ```sudo apt-get install iptables-persistent```

Add following rules: 
* ```sudo iptables -P INPUT ACCEPT```
* ```sudo iptables -A INPUT -i lo -m comment --comment "Traffic related to localhost" -j ACCEPT```
* ```sudo iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -m comment --comment "Traffic related to established connection" -j ACCEPT```
* ```sudo iptables -A INPUT -s aaa.bbb.ccc.ddd -j ACCEPT```
* ```sudo iptables -A INPUT -m comment --comment "Traffic related to other requests" -j DROP```

Save rules to persistant iptables: 
* ```sudo iptables-save > /etc/iptables/rules.v4```

Source: 
https://askubuntu.com/questions/780575/use-iptables-to-block-all-incoming-ips
