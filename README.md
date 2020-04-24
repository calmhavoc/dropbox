# Configure dropbox for Ubuntu 16.10. 
# Use case: plug into target network switch via ethernet, connect to target via wifi or onion hidden service
# Tested on OrangePi Zero; should work on anything

# The usual
apt update && apt upgrade -y
reboot

# Install tor
echo < _EOF >> /etc/apt/sources.list
deb https://deb.torproject.org/torproject.org xenial main
deb-src https://deb.torproject.org/torproject.org xenial main
_EOF

apt install apt-transport-https deb.torproject.org-keyring tor torsocks\
systemctl stop tor

# Modify Tor
## Add hidden service directives to torrc file
echo -e "HiddenServiceDir /var/lib/tor/hiddenSSHservice/\nHiddenSSHServicePort 22 127.0.0.1:22" >> /etc/tor/torrc  
mkdir /var/lib/tor/hiddenSSHservice  
chown -R debian-tor:debian-tor /var/lib/tor/hiddenSSHservice/  
chmod 2700 /var/lib/tor/hiddenSSHservice/  

# Fix tor service and start it
mv /lib/systemd/system/tor.service /lib/systemd/system/bak_tor.service  
systemctl restart tor  
systemctl enable tor  

# Get our onion address
cat /var/lib/tor/hiddenSSHservice/hostname # provides our onion address  


# CONFIGURE AS HOTSPOT 
nmcli con add type wifi ifname wlan0 con-name hotspot autoconnect yes ssid xfinitywifi-2  
nmcli con modify hotspot 802-11-wireless.mode ap 802-11-wireless.band bg ipv4.method shared  
nmcli con modify hotspot wifi-sec.key-mgmt wpa-psk  
nmcli con modify hotspot wifi-sec.psk "secretpassword"  
nmcli con up hotspot  
nmcli con modify hotspot connection.autoconnect true  


# change hostname
hostname=cisco360  
hostnamectl set-hostname $hostname  

# add user
username=james  
adduser --disabled-password --gecos "" $username  
usermod -aG sudo $username  
passwd james  

# Add ssh key
Modify ssh-config to only allow key
disable user: chsh -s /bin/false ubuntu
