# How to get camera via ethernet from Raspberry Pi to Linux

## Requirements
- Raspberry Pi with Pi camera attached using ribbon cable and connecting it to the connector clamp labeled camera, near the USB-A ports
- A Linux device connectable via ethernet
- Ethernet Cable

## Steps
1. **(Re)/Install Raspberry Pi OS**
   - Download the latest version of Raspberry Pi Imager.
   - Prepare the necessary materials:
     - Raspberry Pi board
     - MicroSD card (at least 8GB recommended)
     - Computer with an SD card slot or USB adapter
     - Power supply for the Raspberry Pi
     - HDMI cable (optional, for connecting to a display)
   - Download, Install, and Configure Raspberry Pi OS:
     - Download Raspberry Pi Imager from the official Raspberry Pi website at [https://www.raspberrypi.com/software/](https://www.raspberrypi.com/software/).
     - Insert the MicroSD card into your computer using the card reader or adapter.
     - Select the latest version of Raspberry Pi OS and the storage (SD card) to install it on.
     - Press the Write button and follow the prompted instructions to download it onto the SD card.
     - Once the writing process is complete, remove the MicroSD card from your computer.

2. **Configure Raspberry Pi OS**
   - Insert the MicroSD card into the Raspberry Pi's MicroSD card slot.
   - Connect the Raspberry Pi to a display using an HDMI cable.
   - Connect a keyboard and mouse to the Raspberry Pi's USB ports.
   - Plug in the power supply to turn on the Raspberry Pi.
   - Follow the on-screen instructions to configure your Raspberry Pi, including selecting language, keyboard layout, and password.
   - Make sure to connect to the Wi-Fi network during the setup process.
   - Once the setup is complete, the Raspberry Pi OS desktop will load.

3. **Enable Raspberry Pi Camera**
   - Open a terminal by clicking on the Raspberry icon in the upper left corner, click Accessories, then click Terminal, or press "Ctrl+Alt+T" on your keyboard.
   - Type the following command and press Enter:
     ```
     sudo raspi-config
     ```
   - In the Raspberry Pi Software Configuration Tool, navigate to "Interfacing Options" and select "Camera" to enable it.
   - After enabling the camera, select "Finish" to exit the configuration tool.
   - Reboot your Raspberry Pi if prompted.

4. **Install Motion on Raspberry Pi**
   - Connect the Raspberry Pi to a Wi-Fi network.
   - Open the terminal on your Raspberry Pi.
   - Run the following commands to update the package lists and upgrade existing packages:
     ```
     sudo apt-get update
     sudo apt-get upgrade
     ```
   - If prompted, enter your password.
   - Install Motion by running the following command:
     ```
     sudo apt-get install motion
     ```
   - When prompted, select "y" to continue.
   - Configure Motion:
     - Open the configuration file using the terminal:
       ```
       sudo nano /etc/motion/motion.conf
       ```
     - Update the following parameters:
       - `width` from 620 to 1024
       - `height` from 480 to 720
       - `framerate` from 15 to 30
       - `picture_output` from off to on
       - `movie_quality` from 45 to 60
       - `webcontrol_localhost` from on to off
       - `stream_localhost` from on to off
       - Add the following lines after the `stream_localhost` line:
         ```
         stream_quality 75
         stream_maxrate 30
         ```
     - Save the file by pressing "Ctrl+o" and exit Nano by pressing "Ctrl+x".
   - Restart Motion by entering the following command in the terminal:
     ```
     sudo pkill motion
     ```

5. **Configure the network on the Raspberry Pi**
   - Connect the Raspberry Pi and ensure it is powered.
   - Open a new terminal by pressing "Ctrl+Alt+T".
   - Edit the network configuration file by running the following command:
     ```
     sudo nano /etc/dhcpcd.conf
     ```
   - Scroll to the bottom of the file and add the following lines:
     ```
     interface eth0
     static ip_address=<raspberry_pi_ip>/24
     static routers=<router_ip>
     static domain_name_servers=<router_ip>
     ```
   - Replace `<raspberry_pi_ip>` with the desired IP address for the Raspberry Pi.
   - Replace `<router_ip>` with the IP address of the Linux computer.
   - Save the file by pressing "Ctrl+o" and exit Nano by pressing "Ctrl+x".
   - Restart the Raspberry Pi networking service with the following command:
     ```
     sudo service dhcpcd restart
     ```

6. **Verify it is all working**
   - Connect your Raspberry Pi and Linux computer directly via an ethernet cable.
   - Disable the wireless network on both devices.
   - Open a terminal on your Linux computer and enter the following command:
     ```
     ping <Raspberry_Pi_new_ip_address>
     ```
   - Replace `<Raspberry_Pi_new_ip_address>` with the new IP address noted for the Raspberry Pi.
   - If the ping is successful, you will receive incoming repeating messages. Press "Ctrl+c" to stop.
   - If the ping failed, redo the network configuration on both devices.

7. **Make the Raspberry Pi begin recording upon boot (optional)**
   - Open the terminal on your Raspberry Pi.
   - Edit the `rc.local` file by running the following command:
     ```
     sudo nano /etc/rc.local
     ```
   - Add the following line just before the `exit 0` line:
     ```
     sudo motion -c /etc/motion/motion.conf &
     ```
   - Save the file and exit Nano.
   - Reboot your Raspberry Pi by running:
     ```
     sudo reboot
     ```

8. **To view video**
   - Open any video streaming source or browser.
   - Type in the following address:
     ```
     http://<raspberry_pi_ip_address>:8081/
     ```
   - Replace `<raspberry_pi_ip_address>` with the Raspberry Pi's IP address.
