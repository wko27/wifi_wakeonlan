# wifi_wakeonlan

Do you want to use wakeonlan but got stuck since your desktop connects via wifi?

Here's a solution which uses a raspberry pi to scan the wifi network for the phone, then sends a wakeonlan signal to your desktop via a wired connection to wake it up.

This is tested on:
* Desktop running Windows 10
* Nexus 5 running Android v6.0.1
* Pi Model B running Raspbian Jessie 4.1

### Overview
1. Pi sends an arp request to the wifi network every few seconds
2. If/when the phone is connected to the wifi, it should respond to the request with its mac address
3. When the pi notices that phone was previously disconnected and is now connected, it sends a wakeonlan signal to the Desktop

### Steps:
1. Connect pi to desktop via ethernet
2. Set up desktop
  1. Enable wakeonlan in BIOS settings
  2. Enable wakeonlan in [Windows 10](http://www.groovypost.com/howto/enable-wake-on-lan-windows-10/)
    1. Device Manager -> Network Adapters section -> ethernet controller -> right click to properties
    2. Power Management tab -> Check "Allow this device to wake the computer"
    3. Advanced tab -> Set "Wake on magic packet" to "Enabled"
  3. Grab mac address of your desktop
    1. Network Connections -> ethernet controller -> right click to status
    2. Click "Details..." and copy down the "Physical Address"
3. Set up phone
  1. Keep wi-fi on when sleeping
    1. Settings -> Wi-Fi -> Advanced -> Set "Keep Wi-Fi on during sleep" to "Always"
  2. Grab the mac address of your phone
    1. Settings -> Wi-Fi -> Advanced -> Should be at the bottom
4. Set up pi
  1. Download arp-scan and wakeonlan
    1. sudo apt-get arp-scan
    2. sudo apt-get wakeonlan
  2. Setup ethernet interface
    1. Append the contents of interfaces.append to /etc/network/interfaces
    2. This registers your pi with ip address 10.0.0.1 on the LAN connection to the desktop
    3. Reboot the pi so the configuration takes effect
    4. Test that your pi can see the desktop by running "sudo arp-scan -I eth0 --localnet"
    5. If all goes well, you should see your desktop's ip and mac address pop up
    6. Specifically, it should say "Ending arp-scan (arp scan version number): 256 hosts scanned ... 1 responded"
    7. If not, check if your desktop's firewall is blocking requests
  3. Download contents of this repository onto the pi
    1. Create a directory (e.g. /home/pi/scripts)
    2. Move the is_phone_connected, phone_wake_desktop, and wake_desktop scripts into the directory
    3. Give execute permissions on these scripts by running "chmod +x /home/pi/scripts/*"
  4. Enable pi to check if phone is connected
    1. Update the phone's mac address
    2. Update wifirange if you have a lot of devices on your wifi (as an optimization, we only send an arp requests to the first few addresses since it's unlikely your router is assigning higher addresses to your phone)
    3. Test if this works by running the script when your phone has wifi enabled/disabled
    4. If this doesn't work, go back to step 3
  5. Enable wake desktop
    1. Update macaddr to your desktop's mac address
    2. Test if this works by running the script to turn on your desktop
    3. If this doesn't work go back to step 2
  6. Set up a service to run phone_wake_desktop continuously
    1. Update the location of the phone_wake_desktop script if needed in wake_up script
    2. Move wake_up script into /etc/init.d
    3. Run "sudo update-rc.d wake_up defaults" to register the script to start automatically
    4. Reboot the pi
    5. Check the log output in /var/log/phone_wake_desktop/out.log
  7. Set up a service to remove old log files
    1. Run "crontab -e"
    2. Add "0 0 * * 1 find /var/log/phone_wake_desktop -mtime +7 -exec rm -rf '{}' \;"
    3. This tells crontab that on the first day of each week, remove any files and directories that are over a week old from that log directory

If you get stuck with any of this, please use Google and don't be afraid to take a peek at the scripts.  They are documented and should be fairly readable :)

### Notes
1. arp-scan requires sudo

Cheers,
Andrew
