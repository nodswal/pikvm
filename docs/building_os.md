# Building the OS

The PiKVM OS is based on Arch Linux ARM and contains all the required packages and configs for it to work.  To build the OS you will need x86_64 Linux machine with:

* kernel >= 5.8
* glibc >= 2.33
* docker >= 19.03.13

Docker must be enabled in privileged mode.

1. When starting with a clean OS you need to install and configure docker (after adding your user to the docker group you must log out and log back in), as well as git and make. An example for Ubuntu:

    ```shell
    [user@localhost ~]$ sudo apt-get install git make curl binutils -y
    [user@localhost ~]$ curl -fsSL https://get.docker.com -o get-docker.sh
    [user@localhost ~]$ sudo sh get-docker.sh
    [user@localhost ~]$ sudo usermod -aG docker $USER
    ```

    Re-login to apply the changes.

2. Git checkout the build toolchain:

    ```shell
    [user@localhost ~]$ git clone --depth=1 https://github.com/pikvm/os
    [user@localhost ~]$ cd os
    ```

3. Determine the target hardware configuration (platform):

    * Choose the board: `BOARD=rpi4` for Raspberry Pi 4 or `BOARD=zerow`, `BOARD=rpi2`, `BOARD=rpi3` for other options.
    * Choose the platform:
        * `PLATFORM=v3-hdmi` for RPi4 and PiKVM v3 HAT.
        * `PLATFORM=v2-hdmi` for RPi4 or ZeroW with HDMI-CSI bridge.
        * `PLATFORM=v0-hdmi` for RPi 2 or 3 with HDMI-CSI bridge and Arduino HID.
        * `PLATFORM=v2-hdmiusb` for RPi4 with HDMI-USB dongle.
        * `PLATFORM=v0-hdmiusb` for RPi 2 or 3 with HDMI-USB dongle and Arduino HID.
        * Other options are for legacy or specialized PiKVM boards (WIP).

4. Create the config file `config.mk` for the target system. You must specify the path to the SD card on your local computer (this will be used to format and install the system) and the version of your Raspberry Pi and platform. You can change other parameters as you wish. Please note: if your password contains the # character, you must escape it using a backslash like `ROOT_PASSWD = pass\#word`.

    ```Makefile
    [user@localhost os]$ cat config.mk
    # rpi3 for Raspberry Pi 3; rpi2 for the version 2, zerow for ZeroW
    BOARD = rpi4
    
    # Hardware configuration
    PLATFORM = v2-hdmi
    
    # Target hostname
    HOSTNAME = pikvm
    
    # ru_RU, etc. UTF-8 only
    LOCALE = en_US
    
    # See /usr/share/zoneinfo
    TIMEZONE = Europe/Moscow
    
    # For SSH root user
    ROOT_PASSWD = root
    
    # Web UI credentials: user=admin, password=<this>
    WEBUI_ADMIN_PASSWD = admin
    
    # IPMI credentials: user=admin, password=<this>
    IPMI_ADMIN_PASSWD = admin
    
    # SD card device
    CARD = /dev/mmcblk0
    ```
    
    If you want to configure wifi (for ZeroW board for example) you must add these lines to `config.mk`:

    ```Makefile
    WIFI_ESSID = "my-network"
    WIFI_PASSWD = "P@$$word"
    ```

4. Build the OS. It may take about one hour depending on your Internet connection:

    ```shell
    [user@localhost os]$ make os
    ```
    
5. One of two actions:
    * Put SD card into card reader and install OS (**you should disable automounting beforehand**: `systemctl stop udisk2` or something like that):

        ```shell
        [user@localhost os]$ make install
        ```

    * Make the image to copy elsewhere and burn on to SD card:

        ```shell
        [user@localhost os]$ make image
        ```

        Image is then available as a bziped file in `images/`.

6. After installation remove the SD card and insert it into your RPi. Turn on the power. The RPi will try to get an IP address using DHCP on your LAN. It will then be available via SSH.

7. If you can't find the device's address, try using the following command:

    ```shell
    [user@localhost os]$ make scan
    ```