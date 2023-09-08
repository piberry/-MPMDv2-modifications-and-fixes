# Start here if you want to do the whole process from scratch.  
### Go to step 1.1 to dump the complete flash.  
### Go to step 1.2 if your board is bricked and you just want to recover.
### Go to step 3 to dump your factory_app partition.  
### Go to step 5 to flash the factory_app partition.  

### Prerequisite  
Mandatory:  
Solder a pin header to the ESP32LCD board as seen [here](https://github.com/piberry/-MPMDv2-modifications-and-fixes/blob/ff6dc8617a9fa8b2c46c69943a134510c10867a0/pics/20220801_200219.jpg).  
You need a FTDI or similar device to read/write to the chip. I used a BTT Writer V1.0.  
Use the information from the original [HowTo_unbricking_4U.md](https://github.com/piberry/-MPMDv2-modifications-and-fixes/blob/ff6dc8617a9fa8b2c46c69943a134510c10867a0/HowTo_unbricking_4U.md) document to wire the FTDI to the pin header on the board.  
Download esptool.exe.  
Optional:  
Get a hex editor.  
Install the latest version of python.  
Install [esp32_image_parser](https://github.com/piberry/esp32_image_parser).   
It´s crucial to use the fork by samyk with support for non-DMA DRAM data.  

1.1.Dump flash of the ESP32 WROOM 32E chip with esptool*:  
esptool.exe --chip esp32 --port COM8 --baud 115200 read_flash 0 0x400000 mpmdv2esp32_v1_4_2.bin  
esptool.py 4.6.2  
Serial port COM8  
Connecting.....  
Chip is ESP32-D0WD-V3 (revision 3)  
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None  
Crystal is 40MHz  
MAC: 94:b9:7e:5f:11:11  
Uploading stub...  
Running stub...  
Stub running...  
4194304 (100 %)  
4194304 (100 %)  
Read 4194304 bytes at 0x0 in 380.5 seconds (88.2 kbit/s)...  

If your dump file is not 4MB large, there´s something wrong.  
The created dump can be used to flash another board but be aware that it contains the serial number/MAC address and wifi credentials of the donor board and this will overwrite your own data.  
It can be altered with a hex editor easily, you could also insert your own data.  

1.2.Flash a full dump back to the chip*:  
If you just need to recover, use the mpmdv2esp32_v1_5.bin that i supplied and flash it.  
esptool.exe --chip esp32 --port COM8 --baud 115200 write_flash mpmdv2esp32_v1_5.bin  

If your board isn´t bricked but you aren´t able to update the LCD firmware the regular way, a better way that preserves your own data is to only read/write the main firmware partition.  
Here´s how to do it. Since the offset and lenght of the factory_app partition is shown next under entry 3, you may directly jump to step 3 or step 5.

2.Use esp32_image_parser to get the partition table of the dump:  
python esp32_image_parser.py show_partitions mpmdv2esp32_v1_4_2.bin  
reading partition table...  
entry 0:  
  label      : nvs  
  offset     : 0x9000  
  length     : 16384  
  type       : 1 [DATA]  
  sub type   : 2 [WIFI]  

entry 1:  
  label      : otadata  
  offset     : 0xd000  
  length     : 8192  
  type       : 1 [DATA]  
  sub type   : 0 [OTA]  

entry 2:  
  label      : phy_init  
  offset     : 0xf000  
  length     : 4096  
  type       : 1 [DATA]  
  sub type   : 1 [RF]  

entry 3:  
  label      : factory_app  
  offset     : 0x10000  
  length     : 3604480  
  type       : 0 [APP]  
  sub type   : 0 [FACTORY]  

entry 4:  
  label      : update_app  
  offset     : 0x380000  
  length     : 524288  
  type       : 0 [APP]  
  sub type   : 16 [ota_0]  

MD5sum:  
8f87b3d9c2de5f82d2023b7324702cbe  
Done  

3.The factory_app partition contains the main part of the firmware. Now that we know offset and length, we can dump just that partition*:  
esptool.exe read_flash 0x10000 3604480 factory_app_v1_4_2.bin  
esptool.py v4.6.2  
Found 1 serial ports  
Serial port COM8  
Connecting....  
Detecting chip type... Unsupported detection protocol, switching and trying again...  
Connecting....  
Detecting chip type... ESP32  
Chip is ESP32-D0WD-V3 (revision v3.0)  
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None  
Crystal is 40MHz  
MAC: 94:b9:7e:5f:11:11  
Uploading stub...  
Running stub...  
Stub running...  
3604480 (100 %)  
3604480 (100 %)  
Read 3604480 bytes at 0x00010000 in 328.1 seconds (87.9 kbit/s)...  
Hard resetting via RTS pin...  

4.ToDo: Use a hex editor to extract LCD firmware 1.5 from lcd.efm, merge with factory_app_v1_4_2.bin to obtain a flashable factory_app_v1_5.bin.  

5.Flash the updated factory_app_v1_5.bin back to the chip*:  
esptool.exe --chip esp32 --port COM8 --baud 115200 write_flash 0x10000 factory_app_1_5.bin  
esptool.py v4.6.2  
Serial port COM8  
Connecting....  
Chip is ESP32-D0WD-V3 (revision v3.0)  
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None  
Crystal is 40MHz  
MAC: 94:b9:7e:5f:11:11  
Uploading stub...  
Running stub...  
Stub running...  
Configuring flash size...  
Flash will be erased from 0x00010000 to 0x0037ffff...  
Compressed 3604480 bytes to 1655025...  
Wrote 3604480 bytes (1655025 compressed) at 0x00010000 in 149.3 seconds (effective 193.1 kbit/s)...  
Hash of data verified.  

Done! Disconnect FTDI from PC and the mainboard and power up your MPMDv2. Go to settings > info to confirm that you are on LCD firmware 1.5.  
You can now print directly from cura via the Octoprint plugin.

*--chip --port and --baud are optional. Normally you don´t need to provide these, the correct settings are autodetected.
