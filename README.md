# How to Get Camera via Ethernet from Raspberry Pi to Linux

**Requirements:**
- Raspberry Pi with Pi camera attached using ribbon cable and connecting it to the connector clamp labeled camera, near the USB-A ports
- A Linux device connectable via ethernet
- Ethernet Cable

**Steps:**

1. **(Re)/Install Raspberry Pi OS:**

    - Download the latest version of Raspberry Pi Imager.
    - Prepare the necessary materials: Raspberry Pi board, MicroSD card (at least 8GB recommended), Computer with an SD card slot or USB adapter, Power supply for the Raspberry Pi, HDMI cable (optional, for connecting to a display).
    - Download, Install, and Configure Raspberry Pi OS:
        - Download Raspberry Pi Imager from the official Raspberry Pi website at [https://www.raspberrypi.com/software/](https://www.raspberrypi.com/software/).
        - Insert the MicroSD card into your computer using the card reader or adapter.
        - Select the latest version of Raspberry Pi OS and the storage (SD card) to install it on.
        - Press the Write button and follow the prompted instructions to download onto the SD card.
        - Once the writing process is complete, remove the MicroSD card from your computer and insert it into the Raspberry Pi's MicroSD card slot.
        - Connect the Raspberry Pi to a display using an HDMI cable, and connect a keyboard and mouse to the Raspberry Pi's USB ports.
        - Plug in the power supply to turn on the Raspberry Pi.
        - Follow the on-screen instructions to configure your Raspberry Pi, including selecting language, keyboard layout, and password.
        - Make sure to connect to the Wi-Fi network during the setup process.
        - Once the setup is complete, the Raspberry Pi OS desktop will load.

2. **Enable Raspberry Pi Camera:**

    - Open a terminal by clicking on the Raspberry icon in the upper left corner, click Accessories, then click Terminal, or press "Ctrl+Alt+T" on your keyboard.
    - Type the following command and press Enter:
        
        ```shell
        sudo raspi-config
        ```

    - In the Raspberry Pi Software Configuration Tool menu, highlight "Interfacing Options" and press Enter.
    - In the next menu, select "Camera" and press Enter.
    - Enable the camera by selecting "Yes" and press Enter.
    - Return to the main menu, select "Finish," and press Enter.
    - If prompted to reboot your Raspberry Pi, select "Yes." Otherwise, manually reboot your Raspberry Pi later.
    - Once your Raspberry Pi reboots, the camera should be enabled and ready for use.

# Configure the network on the Raspberry Pi and Linux Computer

1. Connect the Raspberry Pi and the Linux computer using an Ethernet cable. Ensure that both devices are powered on and connected to a Wi-Fi network.

2. Get current IP addresses:

    **On Linux Computer:**
    
    - Open a new terminal by pressing "Ctrl+Alt+T" on your keyboard.
    - Type in the following then press Enter:
    
        ```shell
        hostname -I
        ```
    
    - Write down the returned IP address (you will need this later).
    - This will be considered "Linux IP address" for the rest of the instructions.

    **On Raspberry Pi:**
    
    - Open a new terminal by pressing "Ctrl+Alt+T" on your keyboard.
    - Type in the following then press Enter:
    
        ```shell
        hostname -I
        ```
    
    - Write down the returned IP address (you will need this later).
    - This will be considered "Raspberry Pi original IP address" for the rest of the instructions.

3. Configure the network on the Raspberry Pi:

    - Open a new terminal by pressing "Ctrl+Alt+T" on your keyboard.
    - Edit the network configuration file by running the following command:
    
        ```shell
        sudo nano /etc/dhcpcd.conf
        ```
    
    - Scroll to the bottom of the file and add the following lines at the end of the file:
    
        ```shell
        interface eth0
        static ip_address=<raspberry_pi_ip>/24
        static routers=<router_ip>
        static domain_name_servers=<router_ip>
        ```
    
    - Replace `<raspberry_pi_ip>` with an IP address that is in the same subnet as the Linux computer.
    - Make sure the new IP address you select has the same network address as the existing device. For example, let's assume the Linux computer has an IP address of 192.168.1.100, you can choose an IP address like 192.168.1.200. The network portion (first three octets) should match the network address of the Linux computer and the last octet should be any number between 1 and 256, excluding any current last octet numbers.
    - Write down this new IP address.
    - This will be considered "Raspberry Pi new IP address" for the rest of the instructions.
    - Replace `<router_ip>` with an IP address that has the same first three octets as the Linux IP address and the Raspberry Pi new IP address. Then for the last octet select number one. Write down this IP, you will need it later.

4. Once you are done, save it by pressing "Ctrl+o" then press Enter.
5. Once saved, close the window by pressing "Ctrl+x".
6. Restart the Raspberry Pi networking service with the following command:
    
    ```shell
    sudo service dhcpcd restart
    ```

7. Configure the network on the Linux computer:

    - Open a terminal on the Linux computer.
    - Edit the network configuration file by running the following command:
    
        ```shell
        sudo nano /etc/network/interfaces
        ```
    
    - If prompted, enter your password and press Enter.
    - Add the following lines at the end of the file:
    
        ```shell
        iface eth0 inet static
        address <linux_computer_ip>
        netmask 255.255.255.0
        ```
    
    - Replace `<linux_computer_ip>` with the Linux IP Address you saved from above.
    - Once you are done, save it by pressing "Ctrl+o" then press Enter.
    - Once saved, close the window by pressing "Ctrl+x".
    - Restart the Linux networking service with the following command:
    
        ```shell
        sudo systemctl restart NetworkManager
        ```

8. Next, we will need to find out the name of our Ethernet connection by typing into the terminal:
    
    ```shell
    ip addr show
    ```
    
    - Look through the returned messages for the name of the Ethernet interface and write it down (Eg. eth0, enccw…, or for me, it's enp0…).

9. Next, we will need to open up the network manager through our terminal by using the following command:
    
    ```shell
    sudo nano /etc/netplan/01-network-manager-all.yaml
    ```
    
    - Enter your password and press Enter.
    - Modify the file to include the following configuration for the Ethernet interface:
    
        ```yaml
        network:
          version: 2
          renderer: NetworkManager
          ethernets:
            <Ethernet_interface_name>:
              dhcp4: no
              addresses: [<unique_ip_address>/24]
              gateway4: <router_ip>
        ```
    
    - Replace `<Ethernet_interface_name>` with the name you wrote down a step above.
    - Replace `<unique_ip_address>` with a unique IP address that is within the same subnet as the Raspberry Pi and Linux Computer.
    - Replace `<router_ip>` with the same router IP you used when setting up the Raspberry Pi.
    - Once you are done, save it by pressing "Ctrl+o" then press Enter.
    - Once saved, close the window by pressing "Ctrl+x".
    - Apply the network configuration changes by running the following command in the terminal:
    
        ```shell
        sudo netplan apply
        ```

## Install Motion on Raspberry Pi:

1. Unplug the Ethernet cable from the Raspberry Pi.

2. Update and Upgrade:

    - Open the terminal on your Raspberry Pi.
    - Run the following commands to update the package lists and upgrade existing packages:
    
        ```shell
        sudo apt-get update
        sudo apt-get upgrade
        ```
    
    - If prompted, enter your password and press Enter.
    - Wait for the update to complete.

3. Install Motion:

    - Still in the terminal, use the following command to install Motion:
    
        ```shell
        sudo apt-get install motion
        ```
    
    - When prompted, select "y" and press Enter to continue.
    - If prompted, enter your password and press Enter.
    - Wait for the installation to complete.

4. Configure Motion:

    - Open the configuration file using the terminal:
    
        ```shell
        sudo nano /etc/motion/motion.conf
        ```
    
    - Change `framerate` from `15` to `30`.
    - Change `picture_output` from `off` to `on`.
    - Change `movie_quality` from `45` to `60`.
    - Change `webcontrol_localhost` from `on` to `off`.
    - Change `stream_localhost` from `on` to `off`.
    - After the `stream_localhost` line, add two more lines:
    
        ```
        stream_quality 75
        stream_maxrate 30
        ```
    
    - Once you are done, save it by pressing "Ctrl+o" then press Enter.
    - Once saved, close the window by pressing "Ctrl+x".

5. Restart Motion by entering the following line into the terminal:
    
    ```shell
    sudo pkill motion
    ```

## Install VLC on Linux Computer:

1. Open the terminal on your Linux computer.

2. Run the following commands to update the package lists and upgrade existing packages:
    
    ```shell
    sudo apt-get update
    sudo apt-get upgrade
    ```
    
    - If prompted, enter your password and press Enter.
    - Wait for the update to complete.

3. Install VLC by entering the following line into the terminal:
    
    ```shell
    sudo apt install vlc
    ```
    
    - If prompted, select "y" and press Enter to continue.
    - If prompted, enter your password and press Enter.
    - Wait for the installation to complete.

## Verify It Is Working:

1. Connect your Raspberry Pi and your Linux computer together directly via an Ethernet cable.

2. Disable the wireless network on both your Raspberry Pi and Linux computer.

3. Next, we will ping each device in their terminals to verify there is a two-way connection.

    - Open a terminal on your Linux computer and enter the following line:
    
        ```shell
        ping <Raspberry_Pi_new_ip_address>
        ```
    
        - Replace `<Raspberry_Pi_new_ip_address>` with the new IP address noted for the Raspberry Pi in one of the first steps.
        - If your ping worked, you will receive an incoming repeating message.
        - If your ping failed, you will receive a one-time message indicating "Destination Host Unreachable."
        - If your ping is successful, you will consistently be receiving messages in the terminal. To stop it, press "Ctrl+c".

4. If your ping failed, go back and redo the network configuration of both devices.

## Last Steps:

1. Verify both devices are connected via Ethernet and that the wireless networks are disabled.

2. Enter the following command into your Raspberry Pi terminal to begin the stream:
    
    ```shell
    sudo motion -c /etc/motion/motion.conf
    ```

3. Now open VLC media player on your Linux computer and press "Ctrl+n".

4. When prompted to enter a URL, enter the following:
    
    ```
    http://<Raspberry_Pi_new_ip_address>:8081/
    ```

    - Replace `<Raspberry_Pi_new_ip_address>` with the Raspberry Pi's new IP address noted in one of the first steps.

5. Press play, and a live video stream will appear.

6. To stop recording at any time, press "Ctrl+c" on the Raspberry Pi.

## Make the Raspberry Pi Begin Recording Upon Boot (optional):

1. Open the terminal on your Raspberry Pi.

2. Edit the `rc.local` file by running the following command:
    
    ```shell
    sudo nano /etc/rc.local
    ```

3. In the `rc.local` file, just before the `exit 0` line, add the following line:
    
    ```shell
    sudo motion -c /etc/motion/motion.conf &
    ```
    
    - The `&` symbol at the end of the line is important as it runs the command in the background, allowing the boot process to continue.

4. Save the file and exit Nano by pressing "Ctrl+x", then "y" to confirm saving, and finally Enter to exit.

5. Reboot your Raspberry Pi by running the following command:
    
    ```shell
    sudo reboot
    ```
