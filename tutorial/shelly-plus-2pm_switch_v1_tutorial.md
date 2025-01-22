# Walkthrough of the Driver shelly-plus-2pm_switch_v1
Below i would like to explain to you how the driver works in detail and which commands and functions are used and why.

This Walkthrough is based on the driver shelly-plus-2pm_switch in Version 1. The code of the driver officially offered in meta may change with future versions. Therefore a copy of version 1 has been placed here in the folder “tutorial”. This will help you to follow the tutorial in the future as well.

### Shelly API
The driver makes use of the Shelly Device Api for Gen2+ using the RPC protocol. This is the general description that underlies all communication between meta (NEEO) and the Shelly device. To learn more about the Shelly API, please have a look at [their documentation](https://shelly-api-docs.shelly.cloud/gen2/). All commands to the Shelly device are sent via http get requests.

### Detailed Walkthrough
#### Driver Structure
The file format used for drivers for meta is "json". As such, all basic rules for formatting json files must be taken into account. Unfortunately, comment sections cannot be created in the json file by using the usual special characters.\

The driver uses the following basic structure:

```
 1  { "name":"Shelly Plus 2PM (Mode: Switch)", 
 2  "manufacturer":"Shelly",
 3  "version":1,
 4  "compatible-meta":"1.0.5",
 5  "type":"MUSICPLAYER",
 6  "persistedvariables":{...}
11  "variables": {...}
18  "register":  {...}
23  "discover":  {...}
48  "template":  {...}
96  }
```

#### Driver Header (Lines 1-5)
The header of the driver (here lines 1-5) must contain at least the describing attributes of the driver. These are:\
- name: \<Name of the Driver\>
- manufacturer: \<Name of the Device Manufacturer\>
- version: \<Ascending whole number\> Will be forwarded and processed by the neeo brain and visible in the neeo app. Please note: raising the version number will tell the neeo brain to update the driver features, such as buttons, labels, sliders etc. However, with the following driver structure slipt into "discover" and "template" this feature does NOT work. Devices with drivers split like that must be removed and reinstallled to the neeo brain to fully update all features (new or reanamed buttons etc.).
- type: \<Type\> according to the official [NEEO SDK](https://neeoinc.github.io/neeo-sdk/#src-lib-models-devicebuilder.ts-settype). The selection of the device type has an effect on the automatic generation of recipes and the general presentation in the NEEO user interface. Unfortunately, there is not a suitable type for every imaginable type of device to choose from, so you have to see which type fits best respectively produces what you need. In this case "MUSICPLAYER" was selected, as musicplayer automatically generates a visible recipe for each device added.

...
