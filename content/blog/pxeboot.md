---
Title: Network booting
Description: An introduction to PXE
Date: 2014/05/03 12:40:49
Template: blog-post
---

##Network booting

I recently learned how useful this is. It's another way of getting data on devices that is more flexible than physical storage. It consists of three parts
DHCP server
File server
Executable file

On x86 based computers, the PXE client will request an IP address from the DHCP server.
The DHCP server will return an IP address and if configured the name of a file and the IP address of the TFTP server to download it from.
The PXE client will then download and execute the file.

This post assumes that the pc you are running this on has the IP address 192.168.0.1 please make adjustments as necessary. We need to set up a DHCP server to provide instructions to the PXE client. On debian I will be installing the Internet Systems Consortiums dhcp server. 
```
sudo apt-get install isc-dhcp-server
```

Configure the DHCP leases.
```
#nano /etc/dhcp/dhcpd.conf
cat > /etc/dhcp/dhcpd.conf << EOF
#Lease renewal times
default-lease-time 600;
max-lease-time 7200;

#Provide boot information to clients
allow booting;

#If enabled you have to specify each computer you want #to serve an IP address to 
#ignore unknown-clients;

#DHCP leases
subnet 192.168.0.0 netmask 255.255.255.0 {
    range 192.168.0.16 192.168.0.239;
    option broadcast-address 192.168.0.255;
    option routers 192.168.0.254;
    option domain-name-servers 192.168.0.254;
#This is what to run and from where
    next-server 192.168.0.1;
    filename "pxelinux.0";
}
EOF
```
 
Restart the server to load the new configuration
```
/etc/init.d/isc-dhcp-server restart
```
 
At this point in time a PXE client would try to download the specified files and fail.
#My next post will explain how to set up a tftp server and start getting into payloads

So now we install the trivial file transfer protocol server so our clients can actually download the files they're told to.
```
apt-get install tftpd-hpa
```

 
By default it shares files located in /srv/tftp/ Next we'll just create the basic structure to store our files.
```
mkdir /srv/tftp/
mkdir /srv/tftp/images
mkdir /srv/tftp/pxelinux.cfg
```

 
The images directory will end up containing the various things you want to run.
The pxelinux.cfg directory is where pxelinux will automatically look for a configuration files.
We will only be using the generic file "default".
For detailed information check out http://www.syslinux.org/wiki/index.php/PXELINUX#How_do_I_Configure_PXELINUX.3F
Next we will install pxelinux which is a boot loader made to grab configuration off the network. 
```
apt-get install syslinux
```
Now copy the syslinux files to the TFTP servers directory
```
cp /usr/lib/syslinux/pxelinux.0 /srv/tftp/
cp /usr/lib/syslinux/gpxelinux.0 /srv/tftp/
cp /usr/lib/syslinux/menu.c32 /srv/tftp/
cp /usr/lib/syslinux/vesamenu.c32 /srv/tftp/
cp /usr/lib/syslinux/reboot.c32 /srv/tftp/
cp /usr/lib/syslinux/chain.c32 /srv/tftp/
cp /usr/lib/syslinux/memdisk /srv/tftp/
```

 
After this a PXE client would be able to download and run pxelinux.0 from the server. However with no configuration files it won't do anything.
