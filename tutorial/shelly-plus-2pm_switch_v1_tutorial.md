# Walkthrough of the Driver shelly-plus-2pm_switch_v1
Below i would like to explain to you how the driver works in detail and which commands and functions are used and why.

This Walkthrough is based on the driver shelly-plus-2pm_switch in Version 1. The code of the driver officially offered in meta may change with future versions. Therefore a copy of version 1 has been placed here in the folder “tutorial”. This will help you to follow the tutorial in the future as well.

## Shelly API
The driver makes use of the Shelly Device Api for Gen2+ using the RPC protocol. This is the general description that underlies all communication between meta (NEEO) and the Shelly device. To learn more about the Shelly API, please have a look at [their documentation](https://shelly-api-docs.shelly.cloud/gen2/). All commands to the Shelly device are sent via http get requests.

## Driver Structure
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
18  "discover":  {...},
43  "register":  {...},
48  "template":  {...}
96  }
```
### Use multiple Devices with one Driver
To enable the driver to address multiple Shelly devices, the driver must contain a section in which a list of the devices to be addressed is created.\
In this example, the list of "mydevices" is created over the course of the objects “persistedvariables”, “variables”, “discover” and “register”.
The actual driver for controlling the Shelly devices from this list is then located in the separate “template” section.

Note: In the case of simpler drivers for controlling only one device, the above-mentioned splitting can be dropped. In this case, the object template is not required and its content can be stored directly in the root level of the json.

## Detailed Walkthrough

### Driver Header (Lines 1-5)
<details>
  <summary>click to see code:</summary>
 
```javascript
 1  { "name":"Shelly Plus 2PM (Mode: Switch)", 
 2  "manufacturer":"Shelly",
 3  "version":1,
 4  "compatible-meta":"1.0.5",
 5  "type":"MUSICPLAYER",
```
</details>

The header of the driver (here lines 1-5) must contain at least the describing attributes of the driver. These are:\
- name: \<Name of the Driver\>
- manufacturer: \<Name of the Device Manufacturer\>
- version: \<Ascending whole number\> Will be forwarded and processed by the neeo brain and visible in the neeo app. Please note: raising the version number will tell the neeo brain to update the driver features, such as buttons, labels, sliders etc. However, with the following driver structure split into "discover" and "template" this feature does NOT work. Devices with drivers split like that must be removed and reinstallled to the neeo brain to fully update all features (new or reanamed buttons etc.).
- type: \<Type\> according to the official [NEEO SDK](https://neeoinc.github.io/neeo-sdk/#src-lib-models-devicebuilder.ts-settype). The selection of the device type has an effect on the automatic generation of recipes and the general presentation in the NEEO user interface. Unfortunately, there is not a suitable type for every imaginable type of device to choose from, so you have to see which type fits best respectively produces what you need. In this case "MUSICPLAYER" was selected, as musicplayer automatically generates a visible recipe for each device added.
- compatible-meta: \<Version this Driver was developed in\> This information is not evaluated or processed anywhere. I added this parameter myself in order to have an indication on which version of meta the driver is sure to work with. You can omit this parameter or use it. As you wish.

### persistedvariables (Lines 6-10)
<details>
  <summary>click to see code:</summary>
 
```javascript
6   "persistedvariables":{
7     "IsRegistered": false,
8     "ToInitiate": true,
9     "MyDevices":"[]"
10  },
```
</details>

The persistedvariables are a set of variables that can be stored in a persistent memory for later use. All persisted variables currently in can be saved by triggering the hidden button “__PERSIST”. This driver uses:\
- IsRegistered: \<false\> This variable is forwarded to the NEEO SDK. False triggers the registration process (e.g. entering "security code") when a new device is added in the app.
- ToInitiate: \<true\> Must be true for the registration process to work properly.
- MyDevices: This is the variable in which all devices that are compatible with this driver are stored. 

### variables (Lines 11-17)
<details>
  <summary>click to see code:</summary>
 
```javascript
11  "variables":{
12  "NewDevice":"[]",
13  "NewUri": "",
14  "NewPort":"",
15  "CompatibleModel":"[\"SNSW-002P16EU\", \"SNSW-102P16EU\"]",
16  "CompatibleProfile":"switch"
17  },
```
</details>

Variables are parameters that are used during the running time of meta. If meta is restarted, the values are reset to the values defined here.
- NewDevice: This will contain all the Information for a new device that will potentially be added to "MyDevices".
- NewUri: Adress of the new Device.
- NewPort: Port of the new Device.
- CompatibleModel: Here i list all the Shelly model types that are compatible with this driver
- CompatibleProfile: This is the mode, the Shelly need to be set to, in order to be compatible with this driver

### discover (Lines 18-42)
<details>
  <summary>click to see code:</summary>
 
```javascript
18  "discover": {
19    "welcomeheadertext": "Shelly Plus 2PM in Switch Mode",
20    "welcomedescription":"powered by meta v2 \n by JAC459 \n X \n Driver Development \n by MarkusM",
21    "initcommandset":[{
22      "label":"",
23      "type":"static",
24      "command":"$RegistrationCode",
25      "queryresult":"",
26      "evalwrite":[
27        {"variable":"NewDevice", "value":"DYNAMIK let device = {\"ip\":\"\", \"port\":\"\", \"model\":\"\", \"name\":\"\", \"mac_address\":\"\", \"profile\":\"\", \"auth_en\":\"\" }; device.ip =  \"$Result\".split(\":\")[0]; device.port = (\"$Result\".split(\":\")[1]!=undefined?\"$Result\".split(\":\")[1]:\"80\"); JSON.stringify(device); "},
28        {"variable":"NewUri",    "value":"DYNAMIK \"$Result\".split(\":\")[0] "},
29        {"variable":"NewPort",   "value":"DYNAMIK \"$Result\".split(\":\")[1]!=undefined?\"$Result\".split(\":\")[1]:\"80\"; "}
30      ]
31      },{
32      "type":"http-get",
33      "command": "http://$NewUri:$NewPort/rpc/Shelly.GetDeviceInfo",
34      "queryresult" : "$.",
35      "evalwrite":[
36        {"variable":"NewDevice", "value":"DYNAMIK let device = $NewDevice; device.name = (JSON.parse(\"$Result\").name == null ? \"Shelly Plus 2PM \"+device.ip : JSON.parse(\"$Result\").name); device.model = JSON.parse(\"$Result\").model; device.mac_address=JSON.parse(\"$Result\").mac; device.profile=JSON.parse(\"$Result\").profile; device.auth_en=JSON.parse(\"$Result\").auth_en; JSON.stringify(device); "},
37        {"variable":"MyDevices", "value":"DYNAMIK let device = $NewDevice; let mydevices = $MyDevices; device?.name != undefined && $CompatibleModel.includes(device?.model) && \"$CompatibleProfile\" == device?.profile && device?.auth_en == false ? ( (mydevices=mydevices || []).some(i=>i.ip==device.ip) ? mydevices[mydevices.findIndex(i=>i.ip==device.ip)]=device : mydevices.push(device) ) : mydevices ; JSON.stringify(mydevices); "}
38      ],
39      "evaldo":[{"test":"DYNAMIK let device = $NewDevice; $CompatibleModel.includes(device?.model) && \"$CompatibleProfile\" == device?.profile && device?.auth_en == false; ","then":"__PERSIST", "or":""}]
40      }],
41      "command": {"type": "static", "command": "$MyDevices", "queryresult": "$.*"}
42  },
```
</details>

In "discover" the content of the first page of the regestration process in the NEEO app is set up. In addition, a set of three commands ("initcommandset") is chained that are to be executed prior to proceeding with registration.

#### First initialisation Command: Processing $RegistrationCode
The first command is set to take the content of the variable "$RegistrationCode" and process it for later use. The content of this variable are whatever is entered  in the NEEO app or the web UI during the second page of the registration process as "security code". Also refer to chapter below [register (Lines 43-47)](#register-lines-43-47)

- label: Is set to empty (probably unnecessary to even have this here).
- type: The selected command type is "static" as we only want to do some "calculations" without sending any commands to external devices.
- command: Here we get the content of the variable "RegistrationCode" involved. The dollar symbol is the indicator for meta that a variable name is following in the code. "RegistrationCode" is a variable that is set globaly by meta. It does not need to be set in the [beginning of the driver)](#variables-lines-11-17).
- queryresult: No query needed in this case as we want "RegistrationCode" beeing handed over to evalwrite as is. For the following steps of this command the result of this query will be available as the variable "$RESULT".
- evalwrite: Here we are setting variables of meta to new values. Each variable is called by their name and the values are set. Using the prefix "DYNAMIK" in the value triggers meta to interpret all the following code of the json key "value" as javascript code. Javascript code can easily be tested in online compliers. I like to use [this one](https://onecompiler.com/javascript). You will find a link to selected javascript representations used in "DYNAMIK" of this driver. Your can than run the code in the browser and experiment for yourself. Please note that in DYNAMIK some speacial characters need to be "escaped" using a backslash.
   - variable "NewDevice": An empty temporary variable "device" is created and the "$RESULT" (security code) is split into pieces (ip and port) and saved into that. The variable "device" is than stringifyed to json and the result of that will be "NewDevice". [JS representation](https://onecompiler.com/javascript/436xr9yck)
   - variable "NewUri": The "$RESULT" is proceccced to obtain the IP address of the device. This is done by splitting a potentially existing port from the "$RESULT" that was entered as the "security code". This is done by splitting the "security code" at the ":" and keeping the front half. [JS representation](https://onecompiler.com/javascript/436xqxu5t)
   - variable "NewPort: The "$RESULT" is proceccced to obtain the port address of the device. This is done by splitting the "security code" at the ":" and keeping the back half. Also when no port was inculded, the port will be set to 80.

#### Second initialisation Command: Checking the entered IP address
With the next command we want to call the Shelly device to check whether the entry was correct and the device can be reached. In addition, we want to use the feedback from the device to check whether it is compatible with our driver or not.

- type: The selected command type is "http-get" as we want to call an http adress and get a response.
- command: This is the http address we want to call for the request. In this case we are calling for example: ```http://192.168.178.81:80/rpc/Shelly.GetDeviceInfo``` You can test this kind of http request with a web browser. Calling the command will yield for example the following response.
  <details>
    <summary>click to see response:</summary>

  ```json
  {"name":"Shelly Plus 2PM Switch 81",
   "id":"shellyplus2pm-90150689487c",
   "mac":"XXXXXXXXXXXX","slot":1,
   "model":"SNSW-102P16EU",
   "gen":2,"fw_id":
   "20240726-114505/1.4.0-gb2aeadb",
   "ver":"1.4.0",
   "app":"Plus2PM",
   "auth_en":false,
   "auth_domain":null,
   "profile":"switch"}
  ```
  </details>

- queryresult: This will extract information from the json to be used downstream. There are loads of online query testers out there. I like to use [this one](https://www.jsonquerytool.com/). Using the query "$." basically means "forward unchanged" (but brackets are added), as you can see below.
  ![grafik](https://github.com/user-attachments/assets/8c3f4bbe-0c1c-4611-8e3b-566229db90a1)  
- evalwrite: Here we are resetting variables of meta to new values depending on the response. Each variable is called by their name and the values are set. Using the prefix "DYNAMIK" in the value triggers meta to interpret all the following code of the json key "value" as javascript code.
   - variable "NewDevice": The temporary variable "device" is created and is set to the content of the already existing "NewDevice". Then more information is added to the variable based on the new info from the response. First we check whether the name in the response is "null" (this is the case when not set in the Shelly App). If true, we create a name based on "Shelly Plus 2 PM" followed by the ip adsress. If false, we take the name from the response. Then more information like "model", "mac", "profile" and "auth_en" is read from the response and written into the variable The variable "device" is than stringifyed to json and the result of that will be "NewDevice".
   - variable "MyDevices": Now we want to check if the $NewDevice matches a number of parameters to ensure compatibility with this driver. If so, the new device is added to "MyDevices". For that we are creating temporary variables for $NewDevice and $MyDevices to handle them more comfortably in the javascript code. Then a check is performed where the current model and profile is compared to the $CompatibleModel and $CompatibleProfile defined in the beginning of the driver. Also it is checked if authorization if deactivated on the device. If all of those conditions are true, the $NewDevice is added to $MyDevices looking at $MyDevices and checking if this device already exists. If so the entry is replaced. If the device is new to $MyDevices it will be added (inserted in the end).
   - evaldo: With evaldo we can trigger button action based on a test. Here we are testing basically the same as above, to then trigger the hidden button __PERSIST and write the updated "$MyDevices" to permanent memory (\<DriverName\>-DataStore.json). BTW: __PERSIST is a hidden button already programmed into meta. Other userdefined hidden buttons can be created too by using two underscores infront of a button name.

#### Third initialisation Command: Forwarding $MyDevices
With the third command of the initialisation commands we want to pass the compatible devices ($MyDevices) and including the new device, to the registration process of meta.

- type: The selected command type is "static" as we dont need to send any commands to external devices.
- command: Here load the content of the variable $MyDevices. If already have tow devices it may look like that:
  <details>
    <summary>click to see variable MyDevices:</summary>
   
    ```json
  [{"ip":"192.168.178.81",
  "port":"80",
  "model":"SNSW-102P16EU",
  "name":"Shelly Plus 2PM Switch 81",
  "mac_address":"XXXXXXXXX",
  "profile":"switch",
  "auth_en":false},
  {"ip":"192.168.178.82",
  "port":"80",
  "model":"SNSW-102P16EU",
  "name":"Shelly Plus 2PM Switch 82",
  "mac_address":"XXXXXXXXX",
  "profile":"switch",
  "auth_en":false}]
  ``` 
  </details>

- queryresult: This will extract information from the json to be used downstream. Using the query "$.*" basically means "forward unchanged", as you can see below.
  ![grafik](https://github.com/user-attachments/assets/d9a37be4-bc83-46a9-a4c5-88e4766c124e)

The variable $MyDevices will hearby be handed as the $RESULT to next stage, the actual code of the driver. See [template)](#template-lines-48-95).

Note: The approach of checking a new device for compatability and then adding it to $MyDevices can be found in many of my drivers (in slightly modified variants).

### register (Lines 43-47)
<details>
  <summary>click to see code:</summary>
 
```javascript
43  "register":{
44    "registerheadertext": "Shelly",
45    "registerdescription": "Please enter the IP Adress of your Shelly device. \n For example: 192.168.178.1 \n Port 80 is used by default. Add the port if different.",
46    "command": {"type": "static", "command": ".", "queryresult": "$.*"}
47  },
```
</details>

In "register" the content of the second page of the regestration process in the NEEO app is set up.
- registerheadertext: Deadline displayed.
- registerdescription: Description displayd. You can use "\n" to put the following text in a new line.
- command: In case of the Shelly driver this command is "unused" as previously set device list "MyDevices" will be forwarded to "template".

At the bottom of this second page of the registration process, the “security code” has to be entered. The content of this field will be handed to the variable "$RegistrationCode".

### template (Lines 48-95)
<details>
  <summary>click to see code:</summary>
 
```javascript
48  "template":{
49    "dynamicname":"DYNAMIK_INST_START DYNAMIK JSON.parse(\"$Result\").name DYNAMIK_INST_END",
50    "dynamicid":  "DYNAMIK_INST_START DYNAMIK JSON.parse(\"$Result\").ip   DYNAMIK_INST_END",
51    "variables":{
52      "ShellyURI": "DYNAMIK_INST_START DYNAMIK JSON.parse(\"$Result\").ip   DYNAMIK_INST_END",
53      "ShellyPort":"DYNAMIK_INST_START DYNAMIK JSON.parse(\"$Result\").port DYNAMIK_INST_END",
54      "Switch0Output":false,
55      "Switch1Output":false,
56      "Switch0Status":"",
57      "Switch1Status":"",
58      "Switch0ActionStatus":"",
59      "Switch1ActionStatus":""
60    },
61    "labels":{
62      "StatusSwitch0": {"label":"SWITCH 0", "listen":"Switch0Status", "actionlisten":"Switch0ActionStatus"},
63      "StatusSwitch1": {"label":"SWITCH 1", "listen":"Switch1Status", "actionlisten":"Switch1ActionStatus"}
64    },
65    "listeners":{
66      "ShellySwitch0" : {"type":"http-get", "isHub":false, "command": "http://$ShellyURI:$ShellyPort/rpc/Switch.GetStatus?id=0", "pooltime":"3000", "poolduration":"",
67        "evalwrite": [
68          {"variable": "Switch0Output", "value": "DYNAMIK JSON.parse(\"$Result\").output" },
69          {"variable": "Switch0Status", "value": "DYNAMIK ($Switch0Output==true?\"ON \":\"OFF \") + JSON.parse(\"$Result\").apower + \"W \" + JSON.parse(\"$Result\").current + \"A \" + JSON.parse(\"$Result\").voltage + \"V\" " }
70        ]
71      },
72      "ShellySwitch1" : {"type":"http-get", "isHub":false, "command": "http://$ShellyURI:$ShellyPort/rpc/Switch.GetStatus?id=1", "pooltime":"3000", "poolduration":"",
73        "evalwrite": [
74          {"variable": "Switch1Output", "value": "DYNAMIK JSON.parse(\"$Result\").output" },
75          {"variable": "Switch1Status", "value": "DYNAMIK ($Switch1Output==true?\"ON \":\"OFF \") + JSON.parse(\"$Result\").apower + \"W \" + JSON.parse(\"$Result\").current + \"A \" + JSON.parse(\"$Result\").voltage + \"V\" " }
76        ]
77      }
78    },
79    "switches":{
80      "Switch0" : {"label":"SWITCH 0", "listen":"Switch0Output", "evaldo":[{"test":"DYNAMIK $Switch0Output", "then":"SWITCH 0 ON", "or":"SWITCH 0 OFF"}]},
81      "Switch1" : {"label":"SWITCH 1", "listen":"Switch1Output", "evaldo":[{"test":"DYNAMIK $Switch1Output", "then":"SWITCH 1 ON", "or":"SWITCH 1 OFF"}]}
82    },
83    "buttons":{
84      "POWER ON":    {"label":"", "type":"static",   "command":"", "evaldo":[{"test":true,"then":"__INITIALISE", "or":""}]},
85      "POWER OFF":   {"label":"", "type":"static",   "command":"", "evaldo":[{"test":true,"then":"__CLEANUP",    "or":""}]},
86
87      "SWITCH 0 ON":     {"label":"", "type":"http-get", "command":"http://$ShellyURI:$ShellyPort/rpc/Switch.Set?id=0&on=true", "evalwrite":[{"variable":"Switch0ActionStatus", "value":"DYNAMIK JSON.parse(\"$Result\").was_on==true ? \"stays on\"   : (JSON.parse(\"$Result\").was_on==false ? \"turned on\" : \"Command failed\") "}] },
88      "SWITCH 0 OFF":    {"label":"", "type":"http-get", "command":"http://$ShellyURI:$ShellyPort/rpc/Switch.Set?id=0&on=false","evalwrite":[{"variable":"Switch0ActionStatus", "value":"DYNAMIK JSON.parse(\"$Result\").was_on==true ? \"turned off\" : (JSON.parse(\"$Result\").was_on==false ? \"stays off\" : \"Command failed\") "}] },
89      "SWITCH 0 TOGGLE": {"label":"", "type":"http-get", "command":"http://$ShellyURI:$ShellyPort/rpc/Switch.Toggle?id=0",      "evalwrite":[{"variable":"Switch0ActionStatus", "value":"DYNAMIK JSON.parse(\"$Result\").was_on==true ? \"turned off\" : (JSON.parse(\"$Result\").was_on==false ? \"turned on\" : \"Command failed\") "}] },
90
91      "SWITCH 1 ON":     {"label":"", "type":"http-get", "command":"http://$ShellyURI:$ShellyPort/rpc/Switch.Set?id=1&on=true", "evalwrite":[{"variable":"Switch1ActionStatus", "value":"DYNAMIK JSON.parse(\"$Result\").was_on==true ? \"stays on\"   : (JSON.parse(\"$Result\").was_on==false ? \"turned on\" : \"Command failed\") "}] },
92      "SWITCH 1 OFF":    {"label":"", "type":"http-get", "command":"http://$ShellyURI:$ShellyPort/rpc/Switch.Set?id=1&on=false","evalwrite":[{"variable":"Switch1ActionStatus", "value":"DYNAMIK JSON.parse(\"$Result\").was_on==true ? \"turned off\" : (JSON.parse(\"$Result\").was_on==false ? \"stays off\" : \"Command failed\") "}] },
93      "SWITCH 1 TOGGLE": {"label":"", "type":"http-get", "command":"http://$ShellyURI:$ShellyPort/rpc/Switch.Toggle?id=1",      "evalwrite":[{"variable":"Switch1ActionStatus", "value":"DYNAMIK JSON.parse(\"$Result\").was_on==true ? \"turned off\" : (JSON.parse(\"$Result\").was_on==false ? \"turned on\" : \"Command failed\") "}] }
94    }
  }
```
</details>

The "template" is the actual code of the driver that is used during the operation of the device. It also contains the name and id of the device.

In the beginning we need to set the name and id of the driver respectivly the device instance (this driver can be used with multiple devices simultaniously):
- dynamicname: This is the name that will be displayed in the registration process (list of compatible devices). For that the $RESULT (previously $MyDevices) is evaluated. Using "DYNAMIK_INST_START" as a prefix in a value is telling meta to look at the $Result of the registration process for that.
- dynamicid: This is the id of the device, which helps meta to distinguish between different devices. We are using the IP adress here.

For everything else, we examine the code in smaller chunks, separated according to their function.

#### variables (Lines 51-60)
<details>
  <summary>click to see code:</summary>
 
```javascript
51    "variables":{
52      "ShellyURI": "DYNAMIK_INST_START DYNAMIK JSON.parse(\"$Result\").ip   DYNAMIK_INST_END",
53      "ShellyPort":"DYNAMIK_INST_START DYNAMIK JSON.parse(\"$Result\").port DYNAMIK_INST_END",
54      "Switch0Output":false,
55      "Switch1Output":false,
56      "Switch0Status":"",
57      "Switch1Status":"",
58      "Switch0ActionStatus":"",
59      "Switch1ActionStatus":""
60    },
```
</details>

Variables are parameters that are used during the running time of meta. If meta is restarted, the values are reset to the values defined here.
- ShellyURI: IP address Shelly device as obtained from the variable $MyDevices.
- ShellyPort: Port of the Shelly device as obtained from the variable $MyDevices.
- Switch0Output: Creating a new (boolean) variable to keep track of the output state of switch 0 (same for switch 1).
- Switch0Status: Creating a new (string) variable to keep track of the status of switch 0 to be displayed in a label (same for switch 1).
- Switch0ActionStatus: Creating a new (string) variable to keep track of the action status of switch 0 to be displayed in a label (same for switch 1).

#### labels (Lines 61-64)
<details>
  <summary>click to see code:</summary>
 
```javascript
61    "labels":{
62      "StatusSwitch0": {"label":"SWITCH 0", "listen":"Switch0Status", "actionlisten":"Switch0ActionStatus"},
63      "StatusSwitch1": {"label":"SWITCH 1", "listen":"Switch1Status", "actionlisten":"Switch1ActionStatus"}
64    },
```
</details>

To display the state of the two built in relays of the Shelly device two labels are created.
- label: Name of the label in the NEEO UI.
- listen: Name of the variable with content for the label.
- actionlisten:  Name of the variable with content that is displayed just for a split second. Than the label reverts back to the variable that is entered in "listen".

#### listeners (Lines 65-78)
<details>
  <summary>click to see code:</summary>
 
```javascript
65    "listeners":{
66      "ShellySwitch0" : {"type":"http-get", "isHub":false, "command": "http://$ShellyURI:$ShellyPort/rpc/Switch.GetStatus?id=0", "pooltime":"3000", "poolduration":"",
67        "evalwrite": [
68          {"variable": "Switch0Output", "value": "DYNAMIK JSON.parse(\"$Result\").output" },
69          {"variable": "Switch0Status", "value": "DYNAMIK ($Switch0Output==true?\"ON \":\"OFF \") + JSON.parse(\"$Result\").apower + \"W \" + JSON.parse(\"$Result\").current + \"A \" + JSON.parse(\"$Result\").voltage + \"V\" " }
70        ]
71      },
72      "ShellySwitch1" : {"type":"http-get", "isHub":false, "command": "http://$ShellyURI:$ShellyPort/rpc/Switch.GetStatus?id=1", "pooltime":"3000", "poolduration":"",
73        "evalwrite": [
74          {"variable": "Switch1Output", "value": "DYNAMIK JSON.parse(\"$Result\").output" },
75          {"variable": "Switch1Status", "value": "DYNAMIK ($Switch1Output==true?\"ON \":\"OFF \") + JSON.parse(\"$Result\").apower + \"W \" + JSON.parse(\"$Result\").current + \"A \" + JSON.parse(\"$Result\").voltage + \"V\" " }
76        ]
77      }
78    },
```
</details>

To keep track of the status of the Shelly device http requests are sent to it in regular intervals, asking for information. Actually two http requests are sent as the state of each of the two switches must be requested under a different adsress.

- type: The selected command type is "http-get" as we want to call an http adress and get a response.
- command: This is the http address we want to call for the request. In this case we are calling for example: ```http://192.168.178.81:80/rpc/Shelly.GetDeviceInfo``` You can test this kind of http request with a web browser. Calling the command will yield for example the following response.
  <details>
    <summary>click to see response:</summary>

  ```json
  {"id":0,
   "source":"HTTP_in",
   "output":false,
   "apower":0.0,
   "voltage":232.9,
   "freq":50.0,
   "current":0.000,
   "pf":0.00,
   "aenergy":{"total":0.000,"by_minute":[0.000,0.000,0.000],"minute_ts":1737663180},
   "ret_aenergy":{"total":0.000,"by_minute":[0.000,0.000,0.000],"minute_ts":1737663180},
   "temperature":{"tC":48.2,"tF":118.8}}
  ```
  </details>

- pooltime: This is the time in millisecond after which the command is repeated to ask for the state again.
- poolduration: empty (i never had to use this one before)
- (queryresult: Could potentially be used here. As it is not further specified meta uses the default "$.*" to forward the respone unchanged)
- evalwrite: Here we are resetting variables of meta to new values depending on the response. Each variable is called by their name and the values are set. Using the prefix "DYNAMIK" in the value triggers meta to interpret all the following code of the json key "value" as javascript code.
   - variable "Switch0Output": Forwards the value "output" from the above response. The value is either "true" or "false" depending on the power state.
   - variable "Switch0Status": Here the content of the label is built. Depending on the variable $Switch0Output the label either starts with "ON" or "OFF". After that the measured wattage, current and voltage of the Shelly device are listed.

#### switches (Lines 79-82)
<details>
  <summary>click to see code:</summary>
 
```javascript
79    "switches":{
80      "Switch0" : {"label":"SWITCH 0", "listen":"Switch0Output", "evaldo":[{"test":"DYNAMIK $Switch0Output", "then":"SWITCH 0 ON", "or":"SWITCH 0 OFF"}]},
81      "Switch1" : {"label":"SWITCH 1", "listen":"Switch1Output", "evaldo":[{"test":"DYNAMIK $Switch1Output", "then":"SWITCH 1 ON", "or":"SWITCH 1 OFF"}]}
82    },
```
</details>

This code defines two switches in the NEEO UI for this driver.
- label: Name of the switch in the NEEO UI.
- listen: Name of the variable the switch is "listening" regarding thestate to display (true/ false).
- evaldo: With evaldo we can trigger button action based on a test. Here we are testing the state of the variable "$Switch0Output" (true/ false) to then eihter trigger the button "SWICHT ON 0" or the button "SWICHT OFF 0".

*Known Issue/ limitation: Due to a known bug in the original firmware of the remote control, updating the status of the switches on the remote control does not work properly. This is caused by the driver having two switch instances. However the status is updated correctly in the app and the web UI. As a workaround the regular buttons can be used in combination with the lable to retrieve ths status.*

#### buttons (Lines 83-94)
<details>
  <summary>click to see code:</summary>
 
```javascript
83    "buttons":{
84      "POWER ON":    {"label":"", "type":"static",   "command":"", "evaldo":[{"test":true,"then":"__INITIALISE", "or":""}]},
85      "POWER OFF":   {"label":"", "type":"static",   "command":"", "evaldo":[{"test":true,"then":"__CLEANUP",    "or":""}]},
86
87      "SWITCH 0 ON":     {"label":"", "type":"http-get", "command":"http://$ShellyURI:$ShellyPort/rpc/Switch.Set?id=0&on=true", "evalwrite":[{"variable":"Switch0ActionStatus", "value":"DYNAMIK JSON.parse(\"$Result\").was_on==true ? \"stays on\"   : (JSON.parse(\"$Result\").was_on==false ? \"turned on\" : \"Command failed\") "}] },
88      "SWITCH 0 OFF":    {"label":"", "type":"http-get", "command":"http://$ShellyURI:$ShellyPort/rpc/Switch.Set?id=0&on=false","evalwrite":[{"variable":"Switch0ActionStatus", "value":"DYNAMIK JSON.parse(\"$Result\").was_on==true ? \"turned off\" : (JSON.parse(\"$Result\").was_on==false ? \"stays off\" : \"Command failed\") "}] },
89      "SWITCH 0 TOGGLE": {"label":"", "type":"http-get", "command":"http://$ShellyURI:$ShellyPort/rpc/Switch.Toggle?id=0",      "evalwrite":[{"variable":"Switch0ActionStatus", "value":"DYNAMIK JSON.parse(\"$Result\").was_on==true ? \"turned off\" : (JSON.parse(\"$Result\").was_on==false ? \"turned on\" : \"Command failed\") "}] },
90
91      "SWITCH 1 ON":     {"label":"", "type":"http-get", "command":"http://$ShellyURI:$ShellyPort/rpc/Switch.Set?id=1&on=true", "evalwrite":[{"variable":"Switch1ActionStatus", "value":"DYNAMIK JSON.parse(\"$Result\").was_on==true ? \"stays on\"   : (JSON.parse(\"$Result\").was_on==false ? \"turned on\" : \"Command failed\") "}] },
92      "SWITCH 1 OFF":    {"label":"", "type":"http-get", "command":"http://$ShellyURI:$ShellyPort/rpc/Switch.Set?id=1&on=false","evalwrite":[{"variable":"Switch1ActionStatus", "value":"DYNAMIK JSON.parse(\"$Result\").was_on==true ? \"turned off\" : (JSON.parse(\"$Result\").was_on==false ? \"stays off\" : \"Command failed\") "}] },
93      "SWITCH 1 TOGGLE": {"label":"", "type":"http-get", "command":"http://$ShellyURI:$ShellyPort/rpc/Switch.Toggle?id=1",      "evalwrite":[{"variable":"Switch1ActionStatus", "value":"DYNAMIK JSON.parse(\"$Result\").was_on==true ? \"turned off\" : (JSON.parse(\"$Result\").was_on==false ? \"turned on\" : \"Command failed\") "}] }
94    }
```
</details>

This code defines a number of buttons in the NEEO UI for this driver.

##### POWER ON/ POWER OFF
With this particular diver the buttons "POWER ON" and "POWER OFF" are not actually meant to turn on or off the Shelly device or the built-in relays. The "POWER ON" button is just meant to initialise the above listeners for monitoring the particular Shelly device by triggering the internal hidden button "__INITIALISE". The button "POWER OFF" is stoping the listeners by triggering "__CLEANUP".
- label: empty, so the name will be used.
- type: The selected command type is "static" as we dont need to send any commands to external devices.
- command: empty
- evaldo: With evaldo we trigger the internal hidden buttons "__INITIALISE" and "__CLEANUP"

##### SWITCH 0 ON/OFF/TOGGLE
The buttons "SWITCH 0 ON", "SWITCH 0 OFF" and "SWITCH 0 TOGGLE" are meant to actually control the Shelly device. A set of those buttons is also available for switch 1.The response of all those underlying http request is basically the same. Therefore, the further explanation is made for only the button "SWITCH 0 ON".
- label: empty, so the name will be used.
- type: The selected command type is "http-get" as we want to call an http adress and get a response.
- command: This is the http address we want to call for the request. In this case we are calling for example: ```http://192.168.178.81:80/rpc/Switch.Set?id=0&on=true```. Calling the command will yield for example the following response.
  <details>
    <summary>click to see response:</summary>

  ```json
  {"was_on":false}
  ```
  </details>

- evalwrite: Here we are resetting variables of meta to new values depending on the response. Each variable is called by their name and the values are set. Using the prefix "DYNAMIK" in the value triggers meta to interpret all the following code of the json key "value" as javascript code.
   - variable "Switch0ActionStatus": The content of this variable is meant to be display the switching state (after the button is pressed) for a fraction of a second in the corresponding label. The response will either contain "was_on:true" or "was_on:false". Depending on this value and the button it is used in a combination of contitions is checked to display the corret state.

<br/>
<br/>

### Closing Words
I hope you have learned something about the architecture of a driver for meta and got some inspiration for writing your own.

And if you would like to support what I have created here in my spare time, you are welcome to buy me a [coffee](https://www.paypal.me/MarkusMas721).
