# ConfigAssist
A litewave library allowing easy editing of application variables and quick configuration of **esp32/esp8266** devices 
using a json dictionary.
<p align="center">
  <img src="docs/config.png">
</p>

## Description
**ConfigAssist** will generate a web form that allows editing of application variables such as **Wifi ssid**,**Wifii pasword**. 
A json description text, with a `variable name`, a `default value` and a `label` for each variable must be defined containing all the 
variables that the application will use. 

A simple html page with an edit form will automatically generated allowing the editing of these variables from the web 
Browser. After editing variables can be used in an application with operator **[]**  i.e. ```conf['variable']```.
The configuration data  will be stored in the **SPIFFS** as an **ini file** <em>(Plain Text)</em> and will be 
loaded on each reboot.

## How it works
On first run when no data (ini file) is present, **ConfigAssist** will start an **AccessPoint** and load the
default dictionary with variables to be edited. A web form will be generated in order to 
initialize application variables using the **web page** from the remote host connected to AccesPoint.
Data will be saved on local storage and will be available on next reboot. 
ConfigAssist uses c++ vectors to dynamically store variables and binary search for speeding the process.

## How to use variables
ConfigAssist consists of single file "configAssist.h" that must be included in your application 
The application variables can be used directly by accessing the **class** itself by operator **[]**
i.e.

+ `String ssid = conf["st_ssid"];`
+ `int pinNo = conf["st_ssid"].toInt();`
+ `digitalWrite(conf["led_pin"].toInt(), 0)`;
+ `float float_value = atof(conf["float_value"].c_str());`

## Variables definition with JSON dictionary
In your application sketch file you must define a json dictionary that includes all the information needed 
for the html form to be generated. See example below. Each variable will be displayed on config page with the order 
Defined in the json file.

for example ..
```
const char* appConfigDict_json PROGMEM = R"~(
[
  {
      "name": "st_ssid",
     "label": "Name for WLAN (Ssid to connect)",
   "default": ""
  },
  {
      "name": "st_pass",
     "label": "Password for WLAN",
   "default": ""
  },
  {
      "name": "host_name",
     "label": "Host name to use for MDNS and AP",
   "default": "ConfigAssist"
  }  
]
```

## Project definitions in your main app

+ include the **configAssist**  class
  - `#include "configAssist.h"  // Setup assistant class`

+ Define your static instance
  - `Config conf;        //configAssist class`

+ in your setup function you must init the config class with a pointer to the dictionary
  - `onf.init(appConfigDict_json);`
 
## Access point handler
Define a web server handler function for the **configAssist**. This function will be passed to 
conf.setup in order for configAssist to handle form AP requests
```
// Handler function for AP config form
static void handleAssistRoot() { 
  conf.handleFormRequest(&server); 
}
```
## Setup function
```
void setup()
  // Must have storage to read from
  STORAGE.begin(true);
  
 //Initialize config with application's json dictionary
 conf.init(appConfigDict_json);  

  //Failed to load config or ssid empty
  if(!conf.valid() || conf["st_ssid"]=="" ){ 
    //Start Access point server and edit config
    //Will reboot for saved data to be loaded
    conf.setup(server, handleAssistRoot);
    return;
  }
  ...
  ```

## Compile
Download the files **configAssist.h** and **configAssistPMem.h** and put it in the same directory
as your **sketch foler**. Then include the **configAssist,h** in your application and compile..

+ compile for arduino-esp3 or arduino-esp8266.
+ In order to compile you must install **ArduinoJson** library.
+ if your variables exceed **MAX_PARAMS** increase this value in class header.

###### If you get compilation errors on arduino-esp32 you need to update your arduino-esp32 library in the IDE using Boards Manager
