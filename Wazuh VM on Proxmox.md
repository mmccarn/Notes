Wazuh VM on Proxmox

From the [Wazuh VM Install Guide](https://documentation.wazuh.com/current/installation-guide/virtual-machine.html):

> We provide a pre-built virtual machine image (OVA), you can directly import using VirtualBox (where has been built it) and other OVA compatible virtualization system as well.  
> 
> Note  
> 
> This VM only run on 64-bit systems and is not recommended to use in production environments. Instead, it can be a useful tool for proof of concepts and labs. Distributed architectures and multi-node Elastic Stack clusters are usually a better fit for production environments, where higher performance is required.

These instructions describe how to install the provided Virtualbox VM on Proxmox.

At the time of writing, the VM version is wazuh3.1.0_6.1.1 - you will need to adjust the instructions as versions change.

1. Get the latest VM download URL
  - Open the Wazuh VM Install Guide linked above
  - Place the VM download URL on your clipboard
  
2. Open a shell on your proxmox server  
   Use ssh (linux / mac os X), putty (windows), or the ">_ Shell" button in the Proxmox GUI.

3. Download and extract the Wazuh VM

  ```
  mkdir ~/wazuh
  cd ~/wazuh
  # wget <paste the vm url from step 1 here>
  wget https://packages.wazuh.com/vm/wazuh3.1.0_6.1.1.ova
  #
  # verify that the md5 checksum of the downloaded file matches the value 
  # shown on the wazuh download page.
  # Be sure to use the correct checksum for the file you have just downloaded!
  OVA=wazuh3.1.0_6.1.1.ova
  MD5=3be23b186b414d95ebc94caa3a72cb0c
  [[ $(md5sum $OVA |awk '{print $1}') == "$MD5" ]] && echo Checksum OK || echo Checksum DOES NOT MATCH
  #
  
  tar xvf $OVA
 ```

4. Convert the vmdk disk image to qcow2 format  
   All the online Proxmox howtos recommend converting vmdk disks to qcow2 format.  
   This command will convert all vmdk files in the current directory to qcow2 files in the same folder.  
  ```
  for f in $(find . -name "*.vmdk"); do VMDK=$(basename $f); qemu-img convert -f vmdk $VMDK -O qcow2 ${VMDK/\.vmdk/\.qcow2}; done
  ```

5. Create the Wazuh VM using the proxmox GUI
  - RAM: 2048MB <sup>[*](#ramsize)</sup>
  - DISK: 20GB <sup>[**](#disksize)</sup>   
     **IMPORTANT Select "IDE" under Bus/Device**  
     (When I selected 'SCSI' I was consistently dropped to the dracut emergency shell...)

6. Note the new VM number, and use it to locate the qcow2 disk file created in step 4  
   ("109" in this example)

  ```
  VM=109
  updatedb
  locate vm-$VM-disk
  ```

7. Copy the qcow2 file created in step 4 on top of the qcow2 file created in step 5  
   (Feel free to replace $(locate vm-109-disk) with the actual path to the vm disk file)

  ```
  cp ${OVA/.ova}-*.qcow2 $(locate vm-$VM-disk)
  ```

8. 'Start' the VM, install updates, and reboot
  - default root password *wazuh*
  - update using

  ```
  # update free geop data
  geoipupdate
  cd /etc/logstash
  # create symlinks to geoip data in /etc/logstash
  # (I don't know if this works...)
  ln -s /usr/share/GeoIP/GeoLite* .
  # schedule weekly geoip updates
  cd /etc/cron.weekly
  ln -s /usr/bin/geoipupdate .
  # install any pending Centos or Wazuh updates
  yum update -y
  shutdown -r now
  ```


10. Determine the IP address and connect to Kibana  
   
   ```
   # display the IP Address
   hostname -I
   ```
   
   http://IP_ADDRESS:5601
   
---

<a name="ramsize">*</a>: Memory Size for Wazuh VM  
   The Virtualbox description of the allocated amount of memory can be found as follows:  
   ```
   grep -H "of memory" *.ovf
   #
   # result:
   # wazuh3.1.0_6.1.1.ovf:        <rasd:ElementName>2048MB of memory</rasd:ElementName>
   ```

<a name="disksize">**</a>: Disk size for Wazuh VM  
   Disk Size in a VirtualBox ovf file is a function of two settings:  
1. **ovf:capicity**: the number of allocation units to use  
2. **ovf:capacityAllocationUnits**: the size of each allocation unit  
   "KB", "KILOBYTE" or "byte * 2^10"  
   "MB", "MEGABYTE" or "byte * 2^20"  
   "GB", "GIGABYTE" or "byte * 2^30"  
   "TB", "TERABYTE" or "byte * 2^40"  
     
   ```
   # Get the disk size from the wazuh VirtualBox ovf file
   cd ~/wazuh
   grep -H ovf:capacity *.ovf
   #
   # result:
   # wazuh3.1.0_6.1.1.ovf:    <Disk ovf:capacity="20" ovf:capacityAllocationUnits="byte * 2^30" ... (you can ignore the rest)
   ```
