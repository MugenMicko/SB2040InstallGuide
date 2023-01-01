**Setting up the SB2040 + CanBoot on my RPi **

Steps:
Clone the CanBoot git repository
Build the CanBoot bootloader
Flash CanBoot on the SB2040
Build Klipper for the SB2040
Flash Klipper to the SB2040

Clone the CanBoot git repository
Turn on the printer and SSH into the RPi

Execute the following commands:

cd ~
git clone https://github.com/Arksine/CanBoot
This results in the creation and population of directory ~/CanBoot

Build the CanBoot bootloader
Execute the following commands:

cd ~/CanBoot
make menuconfig
Here is a screenshot of the settings I used for the make menuconfig step:

Please note that my CAN network is configured with bus speed set to 1000000

CanBootMakeMenuconfig

(UPDATE: Changed menu selections from how I did this originally so that the blue LED next to the boot button on the SB2040 will flash after a successful CanBoot flash, to distinguish from a successful Klipper flash where the blue LED remains lit.)

When done, type Q and Y to save the updated configuration

Execute the following commands:

make
Done building the CanBoot bootloader

Flash CanBoot on the SB2040
Starting state is as follows:

RPi is turned on
SB2040 is not plugged into anything
Get a USB-A to USB-C cable and plug the USB-A end of the cable into the RPi

Press and hold the boot button on the SB2040 while plugging in the USB-C end of the cable to the SB2040

Continue to hold the boot button for three seconds and then release it

Check to see if you can see the SB2040:

lsusb
Look for “Raspberry Pi RP2 Boot” and make note of the ID value

I did not see that description, but I did see the following new entry: Printer
Based on my experience, it appears that all SB2040 boards have USB ID "2e8a:0003"
Perform the following command:

sudo make flash FLASH_DEVICE=2e8a:0003
If all went well, then you should see something like this:

CanBootFlash

If the SB2040 was successfully flashed with CanBoot:

Unplug the USB cable from the SB2040 and the RPi
Safely power down the printer
Reconnect the CAN cables to the SB2040
Turn on the printer
The blue LED next to the boot button on the SB2040 should be flashing
Build Klipper for the SB2040
At this point in the process, the pre-existing Klipper image has been partially overwritted by the CanBoot bootloader.

This means that we have to rebuild Klipper, and we need to address the fact that we now have a bootloader on the SB2040, whereas before we did not.

This is accomplished by assigning a value to the Bootloader offset.

The printer should be on, so SSH into the RPi

Perform the following commands:

cd ~/klipper
make menuconfig
Here is a screenshot of the settings I used for the make menuconfig step:

Please note the updated Bootloader offset setting
CanBootMakeMenuconfigKlipper

When done, type Q and Y to save the updated configuration

Perform the following command:

make -j4
Done building Klipper for the SB2040

Flash Klipper to the SB2040
Make a note of the CAN UUID that you defined in your Klipper printer configuration file

My CAN UUID is d063055012c2
Make sure that Klipper is not running on the RPi by executing the following command:

sudo service klipper stop
Execute the following commands to flash Klipper to the SB2040:

cd ~/CanBoot/scripts
python3 flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u d063055012c2
If all went well, then you should see something like this: CanBootKlipperFlash

Restart Klipper by executing the following command:

 sudo service klipper start
