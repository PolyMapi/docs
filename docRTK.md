# RTK documentation

 This doc describes the hardware we have, and how to setup the connections

## Hardware
We have several hardware available to us :

-  a [ZED-F9P](https://www.mikroe.com/gnss-rtk-click) module receives the GPS positions from the GNSS antena. It also receives the correction data from centipede, through a NTRIP client (which must be connected to the Internet). It returns accurate geolocalisation positions (about 1 centimeter accuracy).

<img src="images/GNSS_RTK_click.jpg" alt="gnss rtk click with F9P module" width="400">

- a GNSS antena that allows to receive geolocalisation signals (GPS, BeiDou, Galileo, GLONASS)

- a bluetooth antena that allows to communicate with a smartphone using bluetooth. This smartphone is the NTRIP client which recieves data from centipede.

- an [ESP32](https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32d_esp32-wroom-32u_datasheet_en.pdf) that allows WIFI and Bluetooth connection with an other board, the ZED-F9P in our case.

<img src="images/ESP32.png" alt="ESP32 board" width=200>

- a TinyGS card that links up the ESP32 with other devices.

## Connections
The grove connector must be pluged as the following : on the ESP32 it must be pluged on the I2C port, and on the F9P the white cable must be connected to SDA, the yellow one to SDL, the red one to the current (5V) and the black one to the ground (GND)

The GNSS antena must be connected to the GNSS Antena connector on the ZED-F9P

The bluetooth antena must be pluged on the ESP32

The micro USB cable links up the ESP32 with a computer for developing and debuging.

<img src="images/connections.jpg" alt="connections" width=400>