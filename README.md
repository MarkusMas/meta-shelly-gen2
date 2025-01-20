# meta-shelly-gen2
This repository contains a set of drivers to control Shelly devices of Gen2 upwards using meta v2 and the original NEEO remote.

For Shelly devices of Gen1 please refer to the repository [meta-shelly-driver](https://github.com/vistalba/meta-shelly-driver) by @vistalba.

### Device Compatibility
The drivers in this repository make use of the [Shelly Device Api for Gen2+](https://shelly-api-docs.shelly.cloud/gen2/) using the RPC protocol. The compatibility of your Shelly devices is automatically checked during step 12 of the instructions below. All your compatible devices will be listed for installation.

Further requirement: The IP addresses of your Shelly devices must be fixed!

#### Authentication
Shelly devices can be additionaly proteced by setting a password for each individual device. This password can be set (and/or removed) by accessing the following page using a web browser: `http://\<shelly-ip\>/#/settings/authentication`

Authentication is NOT supported for now. This means that Shelly devices for which a password has been set for authentication are NOT compatible for the time being. This functionality may be added at a later stage.

For Developers: Shelly devices of the first generation are using a very straight forward aproach to authenticate called "basic auth", where the username and password is encoded into the adressline of the http requests. From Gen2 upwards this was replaced by a ["digest authenticaton"](https://shelly-api-docs.shelly.cloud/gen2/General/Authentication) which is way more complicated to implement in the driver format of meta. This authentication method requires two http requests each to execute a command and computing authentication credentials in between.

So the decision was made to initially publish the driver without authentication.

## How to Install a meta-shelly-gen2 Driver
### a) Installation via meta-core Driver
0. Start meta-core on the NEEO remote or in the web-UI
1. Open the directory "Settings"
2. Browse to the submenu "Library"
3. From the library select the driver for your device. For example: "Shelly Plus 2PM (Mode: Switch)"
4. Click "Download this Driver"
5. Click "Activate Driver"
6. Restart meta using meta-core. Go to: "Global Settings" > "Restart meta" > "Restart meta now"!
7. Close/ Power off meta-core.
8. Open the NEEO App or the web-UI and go to: "Devices" > "Add a Device"
9. Search for "shelly"
10. Select the driver you are interested in.
11. Click "next"
12. When asked for the "<security code>", enter the IP adress of your Shelly Device (For example: "192.168.178.1") and click "verify". [Please Note: Port 80 is used by default. Add the port if different. For example: "192.168.178.1:81"] A communication and compatability check is performed in the background. If successful, your device will be added to the list of compatible devices displayed on the next page.
13. Select the device you want to add to your NEEO and click "next".
14. Follow the remaining instructions in the app to install your device to your NEEO system.
 
### b) Manual Installation for advanced Users
1. Download the respective driver file for your device (for example: "shelly-plus-2pm_switch.json") to the Subfolder "active" of your meta Installation.
2. Restart meta
3. Follow the Instructions above starting at Step 8.

## How to use the Shelly Devices on the NEEO Remote
### Device
For each Shelly device added, one device will be added in the Devices section of your NEEO system. The device device name is automatically retrieved from the settings of your shelly during the installation (Step 12). Changing the name in the settings of your shelly at a later stage wont affect the name in the NEEO system, however you can rename you Shelly device in the NEEO app at any time. For that open the NEEO App or the web-UI and go to "Devices" and click the tree dots next to the device.

### Using Recipes
Due to being added as device type "MUSICPLAYER" a recipe is also automatically created for each shelly device in the respective room. If you need, you can disable the automatically generated recipes. For that open the NEEO App or the web-UI and go to "Recipes" and flick the switch.

If you want, you can hide the automatically generated recipe and integrate the features of your Shelly device into other (customized) recipes.

### Buttons "POWER ON" and "POWER OFF"
The buttons "POWER ON" and "POWER OFF" are part of each Shelly driver. These button actions do not actually turn on or off the Shelly device or the built-in relays. The "POWER ON" button initialises the monitoring of the status of the particular Shelly device. The button "POWER OFF" will stop the monitoring.\
Some features like labels, sliders and switches will only update properly, when the button "POWER ON" of the respective Shelly device is added to your (custom) recipe, and therefore monitoring is activated. Also make sure, to add the "POWER OFF" button to the power off routine off your (custom) recipe to stop monitoring this Shelly device.

Note: The buttons "POWER ON" and "POWER OFF" are automatically added to the automatically generated recipes of each Shelly device.

### All other Shortcuts, Labels, Sliders and Switches
Using the actual features of the Shelly devices is done by adding shortcuts, labels, sliders or switches to the slide of a recipe. The features of each Shelly type are different. Please refer to the next chapter.

## Supported Shelly Devices and Features
The follwing Shelly devices of Gen2 upwards are supported and/ or planned to be added. Please contact us on telegram if you have a request for a new Shelly driver.

### Shelly Plus 2PM (Mode: Switch)
The Shelly device ["Shelly Plus 2PM"](https://www.shelly.com/products/shelly-plus-2pm) contains two relays and is capable of power metering (PM) each channel. It can either be set to use both relays independently switch mode. Or it can be set to control covers/ blinds.\
This driver is designed for the mode "switch".

#### Buttons
- SWITCH 0 ON: discrete command to power on the relay
- SWITCH 0 OFF: discrete command to power off the relay
- SWITCH 0 TOGGLE: command to toggle the power state of the relay depending on the current state.

All the above buttons are also available for "SWITCH 1 ..."

#### Lables
- SWITCH 0: contains the current status of the channel including power status (ON/OFF), power consumption (Wattage), current (Amps) and voltage (Volt) . for example: "SWITCH 0: ON 50W 2A 228.6V"

The above label is also available for "SWITCH 1"

#### Switches
- SWITCH 0: Toggle switch with

The above switch is also available for "SWITCH 1"

Known Issue/ limitation: Due to a known bug in the original firmware of the remote control, updating the status of the switches on the remote control does not work properly. This is caused by the driver having two switch instances. However the status is updated correctly in the app and the web UI. As a workaround the regular buttons can be used in combination with the lable to retrieve ths status.

#### Versions
##### Version 1
- First release

### In the Pipeline: Shelly Plus 2PM (Mode: Cover)
The Shelly device ["Shelly Plus 2PM"](https://www.shelly.com/products/shelly-plus-2pm) contains two relays and is capable of power metering (PM) each channal. It can either be set to use both relays independently switch mode. Or it can be set to control covers/ blinds.\
This driver is designed for the mode "cover".

*planned for the end of Q1 2025*

### In the Pipeline: Shelly Dimmer 0/1-10V PM Gen3
The Shelly device ["Shelly Dimmer 0/1-10V PM Gen3"](https://www.shelly.com/products/shelly-0-1-10v-dimmer-pm-gen3) is a dimming controller with a voltage ranging from 0-10 Volts. It is typically used to controll power supplies of LED strips.
This device is also capable of power metering (PM).

*planned for the end of Q2 2025*

## Driver Walkthrough for interested Developers (Tutorial)
It is planned to write a walkthroug of the driver "Shelly Plus 2PM (Mode: Switch)" in Version 1 as a tutorial. This can follow at a later stage.

## ToDos
- Add "digest authentication" so password protected Shellys can be used.
