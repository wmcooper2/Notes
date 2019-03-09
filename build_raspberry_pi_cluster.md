# Build a Raspberry Pi Cluster
I followed an online tutorial about how to setup parallel processing for the Bramble.
I finished this project in August of 2017, but just got around to typing everything up now (March 2019).

OS:         Raspbian Jessie
Language:   Python 3 (using MPI4PY and MPICH3 libraries)

## Parts

Qty     Part                            Yen per unit      Total Yen     Location
---     ----                            ------------      ---------     --------
4       Raspberry Pi 3 Model B              5,258           21,030      Amazon
4       Micro SD card                       1,674            6,696      Yamada Denki
4       USB to micro USB cable (0.5m)         378            1,512      Yamada Denki
1       4-Pi Bramble case                   4,220            4,220      C4 Labs (online)
1       Ethernet switch                     1,280            1,280      Yamada Denki
4       Ethernet cables (0.15m)               335            1,339      Yamada Denki
1       USB hub                             3,413            3,413      Yamada Denki
                                                          =========
                                                            39,490

1       6-port power strip (3.0m)


### Component Specifications
USB hub
    * 7 ports
    * AC adapter
    * 2500 mA

16 GB micro SD cards
    * Elecom brand
    * Class 10
    * 30 Mb/sec

Ethernet switch
    * 5 ports
    * self-powered
    * 10/100

## Steps
1. Download the Raspbian Jessie image.
    * I used the torrent file because I had too much difficulty with teh zip file, just like the author did.
2. Format a micro SD card.
    * Read "Script 1".
    * Write all zeros to it using:
        ```bash
        dd status=progress if=/dev/zero of=/dev/sdc
        ```
    * change "sdc" to the actual device being formatted.
3. Partition the micro SD card
    * I used `gparted` in linux on my laptop. I have Ubuntu installed.
    * Partitioned with a "gpt" file structure.
4. Install MPICH3.
    * This step takes a long time, and it has many substeps.
    * Read "Script 2".
5. Install MPI4PY.
    * This step is also pretty long, and it has several substeps.
    * Many things could go wrong in steps 4 and 5.
6. Clone the micro SD card.
    * If you've successfully completed the steps up to this point, then you should make copies of the micro SD card. I used the built-in program "SD card copier" that comes with Raspbian. I copied the cards using my Pi-top laptop.
7. Set up each pi (sudo raspi-config).
    * Access the pi's one at a time.
    * Get the IP address of the device.
    * Change the hostname.
    * Exapnd the filesystem.
    * Set the memory split to 16.
    * Enable SSH.
    * Finish and reboot.
8. Store each IP address into a host file.
    * Enter the pi that you will designate as your controlling pi.
    * In $HOME/ type:
        ```bash
        vim machinefile
        ```
    * In newly created and opened "machinefile", type all of the IP addresses, one on each line.
    * Example;
        169.254.115.156
        169.254.115.157
        169.254.115.158
        169.254.115.159
    * Save and exit.
9. Configure the SSH on each device.
    * Read "Script 3".
10. Add the authorized keys to all the nodes.
    * Read "Script 4".
11. Test the cluster from the controlling pi with;
    ```bash
    mpiexec -f machinefile -n 4 hostname
    mpiexec -f machinefile -n 4 python /home/pi/mpi4py-2.0.0/demo/helloworld.py
    ```
    * You may have to change the Python version and the mpi4py version in the second command.
    * If you get no errors and a response from each pi, then it was installed successfully.

## Script 1
1. Insert the micro SD card and locate it in the system using `dmesg`.
2. Open the "gparted" program using `gparted`.
3. Unmount the card.
4. Create a partition table (gpt).
5. Create filesystem (ext4).
6. Close "gparted".
7. Format the card using;
    ```bash
    dd status=progress if=/dev/zero of=/dev/<device location>
    ```
    * Device location found in step 1.
8. After formatting, eject with `sudo eject <device location>`
9. Put the card in the node you will use.
10. On each Pi;
    * Update
    * Upgrade
    * Expand filesystem
    * Enable ssh
    * Change hostname
    * Disable wifi
    * Set memory split to 16
    * Check that hostnames match in "/etc/hosts" and in "/etc/hostname" 

## Script 2
1. Update the system.
    ```bash
    sudo apt-get update
    ```
2. Update the packages.
    ```bash
    sudo apt-get dist-upgrade
    ```
    * This took about 1 hour.
3. Create the folder for MPICH3:
    ```bash
    sudo mkdir mpich3
    cd ~/mpich3
    ```
    * Put the dir in $HOME/
4. Download the "3.2 version" of MPICH:
    ```bash
    sudo wget --tries=inf http://www.mpich.org/static/downloads/3.2/mpich-3.2.tar.gz
    ```
    * I had to add "--tries=inf" because after 20 redirects the program "wget" would stop searching.
    * It eventually found it.
5. Unzip the file you just downloaded:
    ```bash
    sudo tar xfz mpich-3.2.tar.gz
    ```
    * I got to this point with an 8GB micro SD card, but it failed saying "not enough space on disk".
    * I tried again from the very beginning of this project using a 16GB micro SD card, then I was able to unzip it.
    * I encountered a problem; "unable to resolve host". I had to edit "/etc/hosts" and "/etc/hostname" to match the hostname of the device. Then it worked.
6. Create folders for mpi:
    ```bash
    sudo mkdir /home/rpimpi/ /home/rpimpi/mpi-install /home/pi/mpi-build
    ```
7. Install gfortran:
    ```bash
    sudo apt-get install gfortran
    ```
    * This took about 10 minutes.
8. Configure and install mpich:
    ```bash
    sudo /home/pi/mpich3/mpich-3.2/configure -prefix=/home/rpimpi/mpi-install
    ```
    * This took about 10 minutes.
    ```bash
    sudo make
    ```
    * This took a long time, about 1 hour.
    ```bash
    sudo make install
    ```
    * This took about 30 seconds.
9. Edit the bash script that runs everytime the Pi starts:
    ```bash
    cd ..
    vim .bashrc
    ```
    * Add the following to the END of the file;
    ```bash
    PATH=$PATH:/home/rpimpi/mpi-install/bin
    ```
    * Save and exit.
10. Reboot the Pi.
    ```bash
    sudo reboot
    ```
11. Test that the MPI works.
    ```bash
    mpiexec -n 1 hostname
    ```
    * If you are successful, then you should see the hostname of your Pi return to you.

### Script 3
1. Download MPI4PY.
    ```bash
    sudo wget --tries=inf https://bitbucket.org/mpi4py/mpi4py/downloads/mpi4py-2.0.0.tar.gz
    ```
    * I had to add "--tries=inf" here again.
    * The author's instructions added the "s" in "https" and left out the "www." in the address.
    * This took about 1 minute.
2. Unzip the file.
    ```bash
    sudo tar -zxf mpi4py-2.0.0.tar.gz
    ```
    * Took less than 1 minute.
3. Change directories.
    ```bash
    cd mpi4py-2.0.0
    ```
4. Install the "python-dev" package.
    ```bash
    sudo aptitude install python-dev
    ```
    * I had an error with the locale settings; changed them to
        ```bash
        LANGUAGE
        LC_ALL
        LANG="enUS.UTF-8"
        ```
    * After fixing the locale problem, I could continue.
5. Run the setup.
    ```bash
    python setup.py build
    sudo python setup.py install
    ```
    * Originally, I got an error that says "the folder doesn't exist" and I can't remember how I fixed it.
6. Set the python path with `export PYTHONPATH=/home/pi/mpi4py-2.0.0`
7. Test that MPI works on your device.
    ```bash
    mpiexec -n 5 python demo/helloworld.py
    ```
    * If you are successful, then running this line of code will return 5 different lines of "helloworld".

### Script 4
1. Run these commands from the controlling Pi.
    ```bash
    ssh-keygen
    cd ~
    cd .ssh
    cp id_rsa.pub PiController
    ssh pi@169.254.115.156
    ssh-keygen
    cd .ssh
    cp id_rsa.pub pi01
    scp pi@169.254.115.156:/home/pi/.ssh/PiController .
    cat PiController >> authorized_keys
    exit

    ssh pi@169.254.115.157
    ssh-keygen
    cd .ssh
    cp id_rsa.pub pi02
    scp pi@169.254.115.156:/home/pi/.ssh/PiController .
    cat PiController >> authorized_keys
    exit

    ssh pi@169.254.115.158
    ssh-keygen
    cd .ssh
    cp id_rsa.pub pi03
    scp pi@169.254.115.156:/home/pi/.ssh/PiController .
    cat PiController >> authorized_keys
    exit
    ```
    * If there is a problem with the 9th line (where you use scp) then you may need to run `sudo apt-get install openssh-server`

### Script 5
1. Run this from the controlling Pi.
    ```bash
    cd ~
    cd .ssh
    scp pi@<pi address>:/home/pi .ssh/PiController
    cat <Pi Key> >> authorized_keys
    ```
    * With this, you are gathering the keys from each of the other Pi nodes into a single file of authorized keys.
    * The controlling Pi gets all the slave keys.
    * Each slave gets the keys of all the other Pi nodes.
    * Basically, all the Pi nodes share all the same keys.

### Setting up a static IP address
1. Don't edit this file, "/etc/network/interfaces".
2. Edit this file, "/etc/dhcpcd.conf".
3. Scroll all the way to the bottom of the file and add one or both of wlan0 and eth0 settings;
    ```bash
    interface eth0
    static ip_address=XXX.XXX.XXX.XXX/XX
    static routers=XXX.XXX.XXX.XXX
    static domain_name_servers=XXX.XXX.XXX.XXX

    interface wlan0
    static ip_address=XXX.XXX.XXX.XXX/XX
    static routers=XXX.XXX.XXX.XXX
    static domain_name_servers=XXX.XXX.XXX.XXX
    ```
    * Replace X's with your IP addresses.
    * "routers" and "domain name servers" values can be the same.
    * "routers" is the "gateway" value listed in the Pi's configuration. 
    * "routers" might be the address of the router you want to use if you are connecting via a router. 
    * You can add muliple IP addresses separated by a space.
4. Save and reboot.
5. Double check after reboot with `ifconfig`.

### Bluetooth Devices
```bash 
sudo bluetoothctl
agent on
default agent
scan on
pair <XX:XX:XX:XX:XX:XX>
connect <XX:XX:XX:XX:XX:XX>
```
* When connecting next time, use steps 2, 4 and 6.

### Network Setup
inet        169.254.115.156     (address)  (changed 156 for each pi to 157, 158, 159)
Bcast       169.254.255.255     (broadcast)
subnet      255.255.0.0         (netmask)
gateway     0.0.0.0             (gateway)
destination 169.254.0.0         (network)

### General Notes
* Must use 16GB micro SD card or bigger. I tried with an 8GB card and it failed.
* `sudo apt-get update && sudo apt-get upgrade` can take a long time, ~15min.
* Install putty
* Editing `/etc/network/interfaces` does not work for setting up a static IP address (follow the script).
* Don't try to boot from a zip file, it doesn't work.
* Be careful when making new files while root. You might accidentally save teh file in a "root" location, not in the location you wanted
* Bcast == Gateway in the IP configuration.

### Useful commands throughout this project
```bash
cat /etc/*release
cat /etc/resolv.conf
dmesg | grep eth0
e2fsck
fsck
ftp
gparted
ifconfig eth0
ip link
iptables
lftp
lshw -C network
fdisk
ip addr show
lsblk
lspci
lsusb
mkfs.ext4 /dev/<location>
mke2fs -t ext4 /dev/<location>
netstat -ie
nmap
ping
putty
route -ne
scp
sftp
ssh
ssh-keygen
sudo eject /dev/<location>
sudo ifconfig eth0 up
sudo ifconfig wlan0 down
sudo raspi-config
sudo reboot
sudo ufw allow 22
sudo service ssh restart
sudo service ssh status
systemctl status networking.service
traceroute
uname -or
vim
vim /etc/dhcpcd.conf
wget --tries=inf <http address>
```

### 7-inch Cocopar HDMI Touchscreen
1. Update Raspbian OS.
2. Open `~/bin/config.txt`
3. Add;
    ```bash
    if 7inch HDMI LCD (C)-(1024 600)
    max_usb_current=1
    hdmi_group=2
    hdmi_mode=87
    hdmi_cvt 1024 600 60 6 0 0 0
    hdmi_drive=1

    if 7inch HDMI LCD (B)/5inch HDMI LCD-(800 480)
    max_usb_current=1
    hdmi_group=2
    hdmi_mode=87
    hdmi_cvt 800 480 60 6 0 0 0
    hdmi_drive=1
    ```
4. Shut off the Pi, plug in the screen and restart the Pi.
    * Touchscreen ability works with the screen is powered by a USB directly from the Pi.

### 3.5-inch Kedei HDMI Touchscreen
1. Update Raspbian OS.
2. Open `~/bin/config.txt`
3. Add;
    ```bash
    hdmi_group=2
    hdmi_mode=87
    hdmi_cvt 480 320 60 6 0 0 0
    ```
    * Touchscreen ability works with the screen is powered by a USB directly from the Pi.

    


