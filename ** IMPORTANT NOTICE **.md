# ** IMPORTANT NOTICE **
## About this branch
This repository is moving to a new branching model.

### 1. Master branch
The master branch is an integration branch where bug fixes/features are gathered for compiling and functionality testing.

### 2. Release branches
The release branch is where releases are maintained and hot fixes (with names like *release/v2.x.x*) are added.
Please ensure that all your production work are tracked with the release branches, so we can push out bug fixes quickly and achieve simple maintenance.

## Roadmap
*ESP8266_RTOS_SDK*'s framework is quite outdated and different from [esp-idf](https://github.com/espressif/esp-idf) and we are planning to migrate ESP8266_RTOS_SDK to esp-idf eventually, after v2.0.0. 

However, we will firstly provide a new version of ESP8266 SDK (*ESP8266_RTOS_SDK v3.0*), which shares the same style with esp-idf, because the multi-CPU architecture is not supported by esp-idf for the time being.

Actions to be taken for *ESP8266_RTOS_SDK v3.0* include the following items:

1. Modify the framework to esp-idf style
2. Restructure some core libraries including Wi-Fi libraries and LibMain
3. Update some third-party libraries including FreeRTOS, lwIP, mbed TLS, noPoll, libcoap, SPIFFS, cJSON, wolfSSL, etc.
4. Update some drivers
5. Others

---

## Requirements

Both the xcc and gcc compilers can be used to compile the project. However, it is recommended that the gcc compiler be used. 

For more information about the gcc compiler, please refer to [esp-open-sdk](https://github.com/pfalcon/esp-open-sdk).

## Compiling

1. Clone *ESP8266_RTOS_SDK*, e.g., `to ~/ESP8266_RTOS_SDK`.

```
    $git clone https://github.com/espressif/ESP8266_RTOS_SDK.git
```

2. Modify *gen_misc.sh* or *gen_misc.bat*:

	* For Linux:
	```
    $export SDK_PATH=~/ESP8266_RTOS_SDK
    $export BIN_PATH=~/ESP8266_BIN
	```
	* For Windows:
	```
    set SDK_PATH=/c/ESP8266_RTOS_SDK
    set BIN_PATH=/c/ESP8266_BIN
	```
	
	You can use *ESP8266_RTOS_SDK/examples/project_template* as your project template, which can be copied anywhere, e.g., `to ~/workspace/project_template`.

3. Generate bins: 
	* For Linux:
	
	```
    ./gen_misc.sh
	```
	* For Windows:

	```
    gen_misc.bat
	```

## Downloading

1. *eagle.app.v6.flash.bin* should be downloaded to the address of *0x00000* in the flash.

2. *eagle.app.v6.irom0text.bin* should be downloaded to the address of *0x40000* in the flash.

3. *blank.bin* should be downloaded to the address of *0x7E000* in the flash.