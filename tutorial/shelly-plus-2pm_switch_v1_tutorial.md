# Walkthrough of the Driver shelly-plus-2pm_switch_v1
Below i would like to explain to you how the driver works in detail and which commands and functions are used and why.

This Walkthrough is based on the driver shelly-plus-2pm_switch in Version 1. The code of the driver officially offered in meta may change with future versions. Therefore a copy of version 1 has been placed here in the folder “tutorial”. This will help you to follow the tutorial in the future as well.

### Shelly API
The driver makes use of the Shelly Device Api for Gen2+ using the RPC protocol. This is the general description that underlies all communication between meta (NEEO) and the Shelly device. To learn more about the Shelly API, please have a look at [their documentation](https://shelly-api-docs.shelly.cloud/gen2/). All commands to the Shelly device are sent via http get requests.

### Driver Structure
The file format used for drivers for meta is "json". As such, all basic rules for formatting json files must be taken into account. Unfortunately, comment sections cannot be created in the json file by using the usual special characters.

The driver uses the following structure and elements:

```javascript
 1  { "name":"Shelly Plus 2PM (Mode: Switch)", 
 2  "manufacturer":"Shelly",
 3  "version":1,
 4  "compatible-meta":"1.0.5",
 5  "type":"MUSICPLAYER",
 6  "persistedvariables":{...},
11  "variables": {...},
18  "register":  {...},
23  "discover":  {...},
48  "template":  {...}
96  }
```
#### Use multiple Devices with one Driver
To enable the driver to address multiple Shelly devices, the driver must contain a section in which a list of the devices to be addressed is created.\
In this example, the list of "mydevices" is created over the course of the objects “persistedvariables”, “variables”, “register” and “discover”.
The actual driver for controlling the Shelly devices from this list is then located in the separate “template” section.

Note: In the case of simpler drivers for controlling only one device, the above-mentioned splitting can be dropped. In this case, the object template is not required and its content can be stored directly in the root level of the json.

### Detailed Walkthrough

#### Driver Header (Lines 1-5)
```javascript
 1  { "name":"Shelly Plus 2PM (Mode: Switch)", 
 2  "manufacturer":"Shelly",
 3  "version":1,
 4  "compatible-meta":"1.0.5",
 5  "type":"MUSICPLAYER",
```
The header of the driver (here lines 1-5) must contain at least the describing attributes of the driver. These are:\
- name: \<Name of the Driver\>
- manufacturer: \<Name of the Device Manufacturer\>
- version: \<Ascending whole number\> Will be forwarded and processed by the neeo brain and visible in the neeo app. Please note: raising the version number will tell the neeo brain to update the driver features, such as buttons, labels, sliders etc. However, with the following driver structure split into "discover" and "template" this feature does NOT work. Devices with drivers split like that must be removed and reinstallled to the neeo brain to fully update all features (new or reanamed buttons etc.).
- type: \<Type\> according to the official [NEEO SDK](https://neeoinc.github.io/neeo-sdk/#src-lib-models-devicebuilder.ts-settype). The selection of the device type has an effect on the automatic generation of recipes and the general presentation in the NEEO user interface. Unfortunately, there is not a suitable type for every imaginable type of device to choose from, so you have to see which type fits best respectively produces what you need. In this case "MUSICPLAYER" was selected, as musicplayer automatically generates a visible recipe for each device added.
- compatible-meta: \<Version this Driver was developed in\> This information is not evaluated or processed anywhere. I added this parameter myself in order to have an indication on which version of meta the driver is sure to work with. You can omit this parameter or use it. As you wish.

#### persistedvariables (Lines 6-10)
```javascript
6   "persistedvariables":{
7     "IsRegistered": false,
8     "ToInitiate": true,
9     "MyDevices":"[]"
10  },
```

The persistedvariables are a set of variables that can be stored in a persistent memory for later use. All persisted variables currently in can be saved by triggering the hidden button “__PERSIST”. This driver uses:\
- IsRegistered: \<false\> This variable is forwarded to the NEEO SDK. False triggers the registration process (e.g. entering "security code") when a new device is added in the app.
- ToInitiate: \<true\> Must be true for the registration process to work properly.
- MyDevices: This is the variable in which all devices that are compatible with this driver are stored. 

#### variables (Lines 11-17)
```javascript
11  "variables":{
12  "NewDevice":"[]",
13  "NewUri": "",
14  "NewPort":"",
15  "CompatibleModel":"[\"SNSW-002P16EU\", \"SNSW-102P16EU\"]",
16  "CompatibleProfile":"switch"
17  },
```

Variables are parameters that are used during the running time of meta. If meta is restarted, the values are reset to the values defined here.
- NewDevice: This will contain all the Information for a new device that will potentially be added to "MyDevices".
- NewUri: Adress of the new Device.
- NewPort: Port of the new Device.
- CompatibleModel: Here i list all the Shelly model types that are compatible with this driver
- CompatibleProfile: This is the mode, the Shelly need to be set to, in order to be compatible with this driver




...
