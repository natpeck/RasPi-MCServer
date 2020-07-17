# RasPi-MCServer
Setting up a Raspberry Pi 3 B as a Minecraft Server from scratch

Materials:
	- Raspberry Pi 3 B
	- USB Keyboard
	- USB Mouse
	- Power w/ micro USB cord
	- Ethernet
	- HDMI Display (TV)
	- Display & Case (MHS 3.5in Display)
	- Micro SD card at least 8GB (using 16GB)
	
Setup the Pi
	1)	Flash Raspbian OS image on SD card
        - Go to https://www.raspberrypi.org/downloads/
        - download "Raspberry Pi Imager for Windows" (https://downloads.raspberrypi.org/imager/imager.exe)
        - mount SD card to computer
        - start "imager.exe"
        - choose OS: "Raspian 32-bit"
        - choose SD previously mounted
        - click "write"
        - once finished, remove SD card
	2)	Assemble the Pi
        - mount the SD card
        - plug in all peripherals (usb keyboard, usb mouse, HDMI display, ethernet)
        - plug in power
	3)	First Boot
        - Once booted, go through the setup wizard
            - skip the wifi setup, the ethernet cable allows internet access
            - allow it to check for, download, and install updates
            - reboot
	4)	Setup Display
        - execute the following commands:
            sudo rm -rf LCD-show
            git clone https://github.com/Lcdwiki/LCD-show.git
            chmod -R 755 LCD-show
            cd LCD-show/
            sudo ./MHS35-show
	5)	Setup Access Point
        - install <<hostapd>> software package
            sudo apt install hostapd
        - enable the wireless access point service
            sudo systemctl unmask hostapd
            sudo systemctl enable hostapd
        - install <<dnsmasq>> software package
            sudo apt install dnsmasq
        - install <<netfilter-persistent>> and its plugin <<iptables-persistent>>
            sudo DEBIAN_FRONTEND=noninteractive apt install -y netfilter-persistent iptables-persistent
        - configure the static IP address
            - edit the configuration file for dhcpcd
                sudo nano /etc/dhcpcd.conf
            - Go to the end of the file and add the following:
                interface wlan0
                  static ip_address=192.168.4.1
                  nohook wpa_supplicant
        - configure the DHCP and DNS services
            - Rename the default configuration file
                sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
            - edit a new one
                sudo nano /etc/dnsmasq.conf
            - Add the following to the file and save it:
                interface=wlan0 # Listening interface
                dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
                  # Pool of IP addresses served via DHCP
                domain=wlan     # Local wireless DNS domain
                address=/gw.wlan/192.168.4.1
                  # Alias for this router
        - ensure wireless operation
            sudo rfkill unblock wlan
        - configure the access point software
            - create the <<hostapd>> configuration file
                sudo nano /etc/hostapd/hostapd.conf
            - add the following to the new file:
                country_code=US
                interface=wlan0
                ssid=TheCraftyMouse's_Server
                hw_mode=g
                channel=7
                macaddr_acl=0
                auth_algs=1
                ignore_broadcast_ssid=0
                wpa=2
                wpa_passphrase=MobileMouse
                wpa_key_mgmt=WPA-PSK
                wpa_pairwise=TKIP
                rsn_pairwise=CCMP
        - run the new wireless access point
            sudo systemctl reboot
	6)	Recheck for updates
        sudo apt-get update
        sudo apt-get upgrade
	
				
References:
	- https://www.raspberrypi.org/documentation/configuration/wireless/access-point-routed.md
	- https://thebetterparent.com/2019/07/how-to-set-up-a-minecraft-server-on-raspberry-pi/
