# Pihole-and-docker-on-Google-cloud-free-tier

# Set up firewall
Go to VCP Network - Firewall
  
Create a rule "local-security-group" - This will only allow browsers from your local LAN access to the services
  - Targets - Apply to all
  - Source - [LOCAL WAN IP]/32
  - Protocol  - TCP:53,80,443,9000,2375
              - UDP:53

Delete rule - default-allow-rdp

# Create free tier VM
Go to Compute Engine - VM instances
  - Click Create Instance
  - Give it a name
  - Choose a US region for free tier - Will state "Your first 720 hours are free"
  - Change Machine Series to N1 and Type to F1
  - Change the boot disk to 30GB
  - Expand " Management, security, disks, networking, sole tenancy"
      - Click Networking
      - Click pen of "Network Interface"
      - Change Exterenal IP to "Create IP address" and use Standard pricing
  - Click Create to build the vm
  
# SSH to the VM
  In VM instances, click the SSH button
  
# Update VM
  sudo apt-get update && sudo apt-get upgrade -y && sudo reboot
  
# Set the VM to automatically update - https://libre-software.net/ubuntu-automatic-updates/
  - sudo apt-get update
  - sudo nano /etc/apt/apt.conf.d/50unattended-upgrades - uncomment the “updates” line by deleting the two slashes at the beginning of it:
    - "origin=Debian,codename=${distro_codename}-updates";
    
    - uncommenting and adapt the following lines:
      - Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
      - Unattended-Upgrade::Remove-Unused-Dependencies "true"; 
      - Unattended-Upgrade::Automatic-Reboot "true";
      - Unattended-Upgrade::Automatic-Reboot-Time "02:38";
  - <ctrl>+o <enter> <ctrl>+x

  - sudo nano /etc/apt/apt.conf.d/20auto-upgrades - In most cases, the file will be empty. Copy and paste the following lines:
     - APT::Periodic::Update-Package-Lists "1";
     - APT::Periodic::Download-Upgradeable-Packages "1";
     - APT::Periodic::AutocleanInterval "7";
     - APT::Periodic::Unattended-Upgrade "1";
  - <ctrl>+o <enter> <ctrl>+x

  You can see if the auto-upgrades work by launching a dry run:
    sudo unattended-upgrades --dry-run --debug

# Install Docker - https://tomroth.com.au/gcp-docker/
  - sudo apt update
  - sudo apt install --yes apt-transport-https ca-certificates curl gnupg2 software-properties-common
  - curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
  - sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
  - sudo apt update
  - sudo apt install --yes docker-ce
  - sudo usermod -aG docker $USER
  - logout

# Install Watchtower - Will keep your docker images up to date - https://github.com/containrrr/watchtower
  - docker run -d \
    --name watchtower \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower


# Install Portainer - https://medium.com/google-cloud/google-container-registry-and-portainer-57198bdae070
    - docker volume create portainer_data
    - docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
  
You can now access Portainer from your LAN using http://[VM external IP]:9000
  
# Install pihole into Docker using Portainer - https://codeopolis.com/posts/running-pi-hole-in-docker-is-remarkably-easy/

  - docker volume create pihole
  - docker volume create dnsmasq
  
  - docker run \
--name=pihole \
-e WEBPASSWORD=YOURPASS \
-e SERVERIP=YOUR.SERVER.IP \
-e DNS1=DNS1IP \
-e DNS2=DNS2IP \
-v pihole:/etc/pihole \
-v dnsmasq:/etc/dnsmasq.d \
-p 80:80 \
-p 53:53/tcp \
-p 53:53/udp \
--restart=unless-stopped \
pihole/pihole

You can now admin your pihole using http://[VM external IP]/admin

# Use Pihole on your LAN
  Change the DNS on your router to be your Goolge Cloud VM IP
  
# Notes
- This is not behind a VPN, so if someone did spoof your WAN IP, they could then access the pihole, if they knew the password
