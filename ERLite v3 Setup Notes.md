# Ubiquiti EdgeRouter Lite 3 Setup Notes

## Initial Setup
* Select a system to use for setup that has a wired network adapter
* Browse to https://www.ubnt.com/download/edgemax and download the latest firmware for "ERLite-3/ERPoe-5"
* hard-code your setup system's wired IP to 192.168.1.11
* connect your setup system to eth0 on ERLite-3
* Power up the ERLite-3
* Browse to https://192.168.1.1
<br>&nbsp; username: ubnt
<br>&nbsp; password: ubnt
* Click 'System' (at the bottom left), scroll down to find 'update firmware' (lower right), click 'upload a file'
<br>&nbsp; Select the firmware file downloaded previously 
<br>&nbsp; (ER-e100.v1.9.7+hotfix.4.5024004.tar as of this writing)
* Reboot

## Continued Setup
* change password for ubnt
* create user 'admin'
* enable public/private key auth for admin
<br>&nbsp; copy id_rsa.pub from your workstation to /config/pubkeys/\<your-system-name\>_rsa.pub
<br>&nbsp; (replace "\<your-system-name\>_rsa.pub" with something that will remind you where this key came from...)
<br>&nbsp; Load the keys into the router config
```
configure
loadkey admin /config/pubkeys/<your-system-name>_rsa.pub
commit
save
exit
#
# for normal configure commands, "commit" will restart affected services
# the "loadkey" macro does not generate the restart of ssh that is
# required for recognition of the new keys
sudo /etc/init.d/ssh restart
```
* Run the 'LoadBalance' wizard
<br>&nbsp; eth1 ->  (your first ISP)
<br>&nbsp; eth2 ->  (your second ISP)
<br>&nbsp; eth0 ->  (your LAN; enable or disable DHCP as appropriate for your situation)

## Remote Administration
* change the Port for ssh in 'system' (to eg 2202)
* Create "WAN_LOCAL" rule to allow new connections -> 2202 -> 2202 to allow remote ssh connections
