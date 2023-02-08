# RTK documentation

 This doc describes the hardware we have, and how to setup the connections

## hardware
We have several hardware available to us :
-  a ZED-F9P module receives the GPS positions from the GNSS antena. It also receives the correction data from centipede, through a NTRIP client (which must be connected to the Internet). It returns more accurate geolocalisation positions (accurate to about 1 centimeter)
![GNSSRKTCLIC](https://github.com/PolyMapi/docs/blob/main/images/F9P_RTK.jpg)
- a GNSS antena which allows to receive geolocalisation signals (GPS, BeiDou, Galileo, GLONASS)
- a bluetooth antena that permits to communicate with a smartphone using bluetooth. Especially, this smartphone is the NTRIP client which recieves data from centipede.
- an ESP32 
- a TinyGS card that links up the ESP32 with other devices

## connections
The grove connector must be pluged as the following : on the ESP32 it must be pluged on the I2C port, and on the F9P the white cable must be connected to SDA, the yellow one to SDL, the red one to the current (5V) and the black one to the ground (GND)

The GNSS antena must be connected to the GNSS Antena connector

The bluetooth antena must be pluged on the ESP32

The micro USB cable links up the ESP32 with a computer.

