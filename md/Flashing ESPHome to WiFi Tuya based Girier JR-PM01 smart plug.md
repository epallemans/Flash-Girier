About 2 years ago, I started to get curious about the electricity consumption of different devices in the house. I use Home Assistant and I prefer Zigbee devices, so I looked around on AliExpress for a Zigbee Smart plug. Unfortunately I messed up when I placed the order and I ordered 5 WiFi based smart plugs instead of Zigbee. I was not happy with my mistake, but they were usable through the Tuya app, although this means they have to go to the cloud. I used them for a little while, but in the end they got replaced by Zigbee plugs and they ended up in a drawer.
Recently I again needed a smart plug, and I started to wonder if it would be possible to flash them with some firmware that can talk to Home Assistant. It took me a lot of surfing, but I finally managed to install ESPHome on them without having to open them or without soldering. I'm very gratefull to all the fantastic people that put these instructions on line, so I decided to give something back and document the way I flashed the plugs.
The most important info I was missing, was simply the confirmation that it is actually possible. If you don't know that it can be done, you are inclined to give up each time something doesn't work.
![[20241121_134944.jpg]]
# Procedure
How does this work? Well, to be honest, it was not very simple. There is a method that requires opening the plug and soldering (see below), but I didn't like that, certainly not for 5 plugs. The other method uses an exploit in the Tuya firmware. It is called cloudcutter and you can read all about it here: [Tuya cloudcutter](https://github.com/tuya-cloudcutter/tuya-cloudcutter/tree/main)
You will need a Raspberry Pi (according to the instructions it should also be possible without, but I didn't try that) and some basic knowledge about setting up a Raspberry Pi, Home Assistant and ESPHome . The ESPHome add on should already be installed in Home Assistant.
These are the steps
1 - Install esphome kickstarter firmware using cloudcutter on a separate Raspberry Pi
2 - Connect the device to your network
3 - Create and install ESPHome firmware from YAML with everything needed for your smartplug
4 - Link the plug to Home Assistant 
5 - Calibrate the energy consumption parameters
## Step 1: Install esphome kickstarter firmware using cloudcutter
You will need a separate Raspberry Pi for this if you want to do it the same way I did.
Follow these instructions to prepare the Raspberry: [Install cloudcutter on Raspberry Pi](https://github.com/tuya-cloudcutter/tuya-cloudcutter/blob/main/HOST_SPECIFIC_INSTRUCTIONS.md)
In the last step, you launch cloudcutter with this command ( without any additional parameters). **Important !** Make sure that the Tuya app on your mobile phone can not interfere with the process, by putting the phone in Airplane mode.
```
  sudo ./tuya-cloudcutter.sh -r
```
At the first prompt, you choose option 2 (Flash 3rd party firmware):
```
 1) Detach from the cloud and run Tuya firmware locally
2) Flash 3rd Party Firmware
[?] Select your desired operation [1/2]:
```
At the next prompt, select 'By firmware version and name'
```
[?] How do you want to choose the device?: By manufacturer/device name
   By manufacturer/device name
   ► By firmware version and name
   From device-profiles (i.e. custom profile)
```
Then you select the 1.1.8 - BK7231N / oem_bk7231n_plug
```
[?] Select the firmware version and name: 
   1.1.7 - BK7231N / oem_bk7231n_ceiling_light_ty
   1.1.7 - BK7231N / oem_bk7231n_plug
   1.1.7 - BK7231T / oem_bk7231s_light_pir_ty
   1.1.7 - BK7231T / oem_bk7231s_light_ty
   1.1.71 - BK7231T / bk7231t_common_user_config_ty
   1.1.8 - BK7231N / farylink_soc_hzs_5_light_15155
 ► 1.1.8 - BK7231N / oem_bk7231n_plug
   1.1.8 - BK7231T / oem_bk7231s_rnd_switch
   1.1.80 - BK7231T / bk7231t_common_user_config_ty
   1.1.9 - BK7231N / oem_bk7231n_water_sensor_plus
   1.1.9 - BK7231T / bk7231s_common_iot_config_ty
   1.1.9 - BK7231T / oem_bk7231s_ceiling_light_ty
   1.1.9 - BK7231T / oem_bk7231s_light_db_cree
```
Choose the ESPHome  kickstart firmware
```
[?] Select your custom firmware file for BK7231N chip: ESPHome-Kickstart-v23.08.29_bk7231n_app.ota.ug.bin
 ► ESPHome-Kickstart-v23.08.29_bk7231n_app.ota.ug.bin
   OpenBeken-v1.17.697_bk7231n.ug.bin
```
The script will continue and ask you to put the device in 'Slow blink mode'. 
```
Selected Device Slug: jiangsu-barep-baa-032-wall-outlet
Selected Profile: oem-bk7231n-plug-1.1.8-sdk-2.3.1-40.00
Selected Firmware: ESPHome-Kickstart-v23.08.29_bk7231n_app.ota.ug.bin

================================================================================
Place your device in AP (slow blink) mode.  This can usually be accomplished by either:
Power cycling off/on - 3 times and wait for the device to fast-blink, then repeat 3 more times.  Some devices need 4 or 5 times on each side of the pause
Long press the power/reset button on the device until it starts fast-blinking, then releasing, and then holding the power/reset button again until the device starts slow-blinking.
See https://support.tuya.com/en/help/_detail/K9hut3w10nby8 for more information.
================================================================================

```
On the plug, press the button for about 10 seconds (the relay will click and the red LED will turn on and off before you reach the 10 seconds, just keep pressing). When the blue LED starts blinking, release the button. It is now in 'Fast blink mode'. Press again for another 10 seconds until the blue LED starts blinking slowly. It is now in 'Slow blink mode'.
The Raspberry will try to find the device and send the exploit code. 
```
Wiping NetworkManager configs
Scanning for open Tuya SmartLife AP
Error: Device 'wlan0' not found.
.Error: Device 'wlan0' not found.
.Error: Device 'wlan0' not found.
.Error: Device 'wlan0' not found.
.Error: Device 'wlan0' not found.
.Error: Device 'wlan0' not found.
..............
Found access point name: "SmartLife-6354", trying to connect...
Device 'wlan0' successfully activated with '60ab1d4e-ab3d-452e-ab84-5d146f04fde1'.
Connected to access point.
Waiting 1 sec to allow device to set itself up...
Running initial exploit toolchain...
Exploit run, saved device config too!
output=/work/configured-devices/ehUZbz22urxU.deviceconfig
Saved device config in /work/configured-devices/ehUZbz22urxU.deviceconfig
================================================================================
Power cycle and place your device in AP (slow blink) mode again.  This can usually be accomplished by either:
Power cycling off/on - 3 times and wait for the device to fast-blink, then repeat 3 more times.  Some devices need 4 or 5 times on each side of the pause
Long press the power/reset button on the device until it starts fast-blinking, then releasing, and then holding the power/reset button again until the device starts slow-blinking.
See https://support.tuya.com/en/help/_detail/K9hut3w10nby8 for more information.
================================================================================
```
When it is ready, you are asked to power cycle the device (remove it from the outlet and then plug it in again) and put it again in 'Slow blinking mode'. It will then connect again and flash the new firmware.
```
Wiping NetworkManager configs
Scanning for open Tuya SmartLife AP
.......
Found access point name: "A-6354", trying to connect...
Device 'wlan0' successfully activated with 'ade4a32e-78df-4bd8-9190-e09d5e4ee43e'.
Connected to access point.
Configured device to connect to 'cloudcutterflash'
Device is connecting to 'cloudcutterflash' access point. Passphrase for the AP is 'abcdabcd' (without ')
Flashing custom firmware...

================================================================================
Wait for up to 10-120 seconds for the device to connect to 'cloudcutterflash'. This script will then show the firmware upgrade requests sent by the device.
================================================================================

Using WLAN adapter: wlan0
Configuration file: /dev/stdin
Using interface wlan0 with hwaddr b8:27:eb:57:2d:d9 and ssid "cloudcutterflash"
wlan0: interface state UNINITIALIZED->ENABLED
wlan0: AP-ENABLED 
```

In theory the process should continue, but for all my plugs it stopped here. If you should have the same issue as me, connect the plug using the Tuya app on your phone as explained below in 'Solving `wlan0: AP-ENABLED`'

```
Using PSK v1 - Received PSK ID version 01
Processing endpoint /v2/url_config
Processing endpoint tuya.device.active
Processing endpoint tuya.device.dynamic.config.get
Processing endpoint tuya.device.upgrade.get
Processing endpoint tuya.device.upgrade.status.update
Processing endpoint /files/ESPHome-Kickstart-v23.08.29_bk7231n_app.ota.ug.bin
Processing endpoint tuya.device.uuid.pskkey.get
Firmware update progress: 7%
Firmware update progress: 25%
Firmware update progress: 55%
Firmware update progress: 85%
[Firmware Upload] /files/ESPHome-Kickstart-v23.08.29_bk7231n_app.ota.ug.bin send complete, request range: bytes=0-561887/561888
Firmware update progress: 98%
Firmware file has been sent and MQTT reported a progress of nearly complete.  Waiting 15 seconds to ensure flashing completes.
Flashing should be complete.  It takes about 15 seconds for the device to reboot and verify the flash was valid.
Please wait about 30 seconds then look for signs of activity from the firmware you supplied (either watch for AP mode or check if it joined your network).
Device MAC address: 1c:90:ff:5b:63:54

```
## Step 2: Connect the plug to your WiFi network
You will need your phone for this. Disable mobile data and connect the Wifi to SSID 'kickstart-bk7231n'. Using your phone browser go to site http://192.168.4.1. Select your Wifi network and password and connect the device.
## Step 3: Create and flash new ESPHome firmware 
Launch the ESPHome add on in Home Assistant and click 'New Device'. Give it a name, choose device type BK72xx and then CB2S Wi-Fi module. Skip the installation and click EDIT on the newly created device. Add this YAML below the auto generated code:
```
web_server:
  port: 80

time:
  - platform: homeassistant
    id: homeassistant_time

text_sensor:
  - platform: version
    name: version
  - platform: wifi_info
    ip_address:
      name: ip
    ssid:
      name: ssid
    bssid:
      name: bssid

binary_sensor:
  - platform: gpio
    id: binary_switch_1
    pin:
      number: P10
      inverted: true
      mode: INPUT_PULLUP
    on_press:
      then:
        - switch.toggle: switch_1

switch:
  - platform: gpio
    id: switch_1
    name: Relay 1
    pin: P26
    restore_mode: RESTORE_DEFAULT_ON
    # uncomment next lines ig you prefer the LED to represent the on/off status
    # on_turn_on:
    #  - light.turn_on: light_status
    # on_turn_off:
    #  - light.turn_off: light_status

light:
  - platform: status_led
    id: light_status
    pin:
      number: P23
      inverted: true

sensor:
  - platform: uptime
    name: uptime
  - platform: wifi_signal
    name: wifi_signal
    update_interval: 10s
  - platform: hlw8012
    model: BL0937
    change_mode_every: 3
    update_interval: 3s
    cf_pin:
      number: P7
      inverted: true
    cf1_pin:
      number: P6
      inverted: true
    sel_pin:
      number: P24
      inverted: true
    current:
      name: Current
      filters:
        - multiply: 0.453
    voltage:
      name: Voltage
    power:
      name: Power
    energy:
      name: Energy
    voltage_divider: 781
    current_resistor: 0.001 ohm
```

Copy the api encryption key from the auto generated code, you will need it later. You can off course allways come back if you need it.
Choose install, manual download, and after the new firmware has been compiled, download the UF2 package to your PC.
Then connect to http://kickstart-bk7231n.local. If you still see the WiFi selection screen, you might be tempted to use the OTA firmware update on the bottom of the screen. This will not work ! Instead, shut down and restart the plug. You should now see a black kickstarter screen with a lot of info about the processor, pin assignment, ... Under 'OTA Update' select the UF2 file and click update.
## Step 4: Link the plug to Home Assistant
In Home assistant go to ESPHome integration (not the add-in, but the integration). Your device should already be auto detected (probably still with kickstarter name, it doesn't matter). Click install and paste the encryption key you copied in the previous step.
## Step 5:  Calibrate the energy consumption parameters
In order to get correct measurements, your plug needs to be calibrated. On the ESPHome page for the hlw8012 you'll find a clear explanation. [Sensor hlw8012](https://esphome.io/components/sensor/hlw8012.html) I used another Zigbee plug as 'calibrated power meter'. After measuring enter the values in the on line calculator (see link above) and update voltage_divider, current_resistor and multiply values in the YAML file. Compile and reinstall the new firmware to your plug.
# Other info that might be useful
## Solving `wlan0: AP-ENABLED`
On 2 plugs the install of the kickstarter software did not work right away. The cloudcutter software on the Raspberry Pi kept hanging after wlan0: AP-ENABLED. Finally I found the solution here: https://www.elektroda.com/rtvforum/topic3970277.html#20658238. Thank you MnM1! 
You have to add it using the Tuya software:
1 - Keep cloudcutter running and waiting in the 'AP-ENABLED' mode.
2 - Remove the location permission on the Tuya app on your movile phone.
3 - Start the Tuya app
4 - Add a device using the plus sign in the upper right corner and then choose 'Add device'
5 - The app will complain that it doesn't have Location permissions, choose 'already set, ignore'. Do this every time you get the message in the coming steps.
6 - Choose 'Add manually' - 'Electrical' - 'Socket' - 'Socket (Wi-Fi)'
7 - Enter Wi-Fi Name 'cloudcutterflash' and Wi-Fi password 'abcdabcd' 
8 - Power cycle your plug and put it in 'Slow blinking' mode by following instructions on your phone
9 - As instructed by the Tuya software: connect your phone to Wi-Fi with SSID A-xxxx with xxxx being 4 digits.
10 - Installation of the firmware should continue in the cloudcutter screen after about 1 minute
## Opening the plug (not needed!)
You don't need to open the plug if you flash it using the above exploit. If however you should want to open it for another reason, here is the method that worked for me : [# How to open smart socket [WiFi Smart Socket]](https://www.youtube.com/watch?v=VLUnsElVz20&t=46s) It worked for me, but it was not as easy as this video suggests: after cracking all the sides, I still needed to pull the two pieces apart with pliers.
## Flashing with UART (only if the other method doesn't work)
I did not try this method, but if the other method does not work and you are comfortable with soldering, you could try this:  [# Smart Plug GIRIER JR-PM01 16A Energy Monitoring CB2S / BK7231N](https://www.elektroda.com/rtvforum/topic3937910.html) 
## Checking the software version
I have absolutely no idea if there are plugs around with other firmware versions, or whether these instructions work on other version, but my plugs were on 1.1.8.
If you want to find out the version number, install the Tuya app, link the plug, in 'All devices' tap on the plug (it needs to be on line), tap the pencil in the upper right corner and select 'Device update'. Mine was on 1.1.8.
![[Screenshot_20241120_175122_Tuya Smart.jpg]]
