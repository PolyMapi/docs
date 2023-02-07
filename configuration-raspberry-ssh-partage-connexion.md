Prerequisites:

 - Software: VSCode with the Microsoft "Remote SSH" extension, Raspberry Pi Imager (tested on version 1.7.3)
 - Hardware: A Raspberry, a phone allowing connection sharing

Generate an SSH key on the computer with the command `ssh-keygen -t ed25519`

Insert the micro-SD of the Raspberry in the drive of the computer

Start the connection share. Connect the computer to the share. Note the SSID and password. Make sure it is active throughout the process.

Open Raspberry PI Manager
In Operating System/Choose OS, select "Raspberry PI OS (32-bit)"
In the menu that just appeared, fill in the following fields:

 - Set hostname -> Leave disabled
 - Enable SSH -> Allow public authentication only -> Enable, and enter the content of the public key
 - Set username and password -> Enable, and enter the credentials of your choice
 - Configure wireless LAN -> Enable, and enter the SSID and password of the connection share
 - Wireless LAN country -> Country in which you realize it (we put FR)
 - Set locale settings -> Leave disabled
 - Persistent settings -> Enable/Disable according to your choice. (Note: Telemetry involves sending data, which can be an additional cost when sharing a connection)
In Storage, choose the Micro SD
Click on Write, and confirm the overwriting of data

Once the procedure is complete, eject the micro-SD and reinsert it into the Raspberry, then turn it on

Launch VSCode

Click on the small green button at the bottom left of the screen

In the list of actions displayed at the top of the screen, connect via SSH

Indicate the connection to chosen user name@raspberrypi

Once connected, the green button at the bottom left will indicate the SSH connection

A window will open, allowing you to work on the Raspberry and access its files.
