Unifi (as of this writing) still uses old/outdated versions of mongodb and java jdk.  To get the older mongodb, install on Ubuntu v16.04.
https://github.com/mmccarn/Notes

# Create Container

* connect to your proxmox host and use these commands to download the ubuntu 16.04 container image

  ```
  pveam update
  pveam available -section system
  pveam download local $(pveam available -section system |grep ubuntu.*16.*04 |sed 's/.*ubuntu/ubuntu/')
  ```
  
* Create new container in proxmox ui
  * Name: unifi
  * Template:
    ```ubuntu-16.04-standard_16.04.5-1_amd64.tar.gz```
  * Root Disk
    ```32GB```
  * Memory
    ```1536 Mib```
  * Network
    ```Use values appropriate for your network```
  * DNS
    ```Use values appropriate for your network```
    
# Update Ubunti
* Start and Update the container
   * Start from Proxmox UI
   * Use the proxmox _Console to login as 'root' using the password selected during container creation
   
   ```
   apt update && apt upgrade -y
   reboot
   ```
   
# Install prerequisites
* Setup repositories and install prerequisites
 
   ```
   apt-get install gnupg2
   echo 'deb https://www.ui.com/downloads/unifi/debian stable ubiquiti' | sudo tee /etc/apt/sources.list.d/100-ubnt-unifi.list
   wget -qO - https://www.mongodb.org/static/pgp/server-3.4.asc | apt-key add -
   wget -O /etc/apt/trusted.gpg.d/unifi-repo.gpg https://dl.ui.com/unifi/unifi-repo.gpg
   echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-3.4.list
   apt-get update
   ```
   
* Install openjdk-8 to avoid installing the openjdk-9 in the next step

  ```
  apt install openjdk-8-jre-headless
  ```

# Install Unifi
* Install unifi

  ```
  apt install unifi -y
  ```

Done.  

You can now browse to your installed unifi controller by IP address on port 8443 and either set it up from scatch or restore a backup.
