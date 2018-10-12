Mconfig
=========

Mconfig (Mesh Network Configuration) is a network configuration solution for ESP-MESH, which aims to send the configuration information to ESP-MESH devices in a convenient and efficient manner.

Mconfig uses `ESP-Mesh App <https://github.com/EspressifApp/Esp32MeshForAndroid/raw/master/release/mesh.apk>`_. During the network configuration, RSA algorithms is used for key agreement, 128-AES algorithms for data encryption, and CRC algorithms for checking and verification.

Terminology
----

.. image:: http://static.zybuluo.com/zzc/5bhlouom9u52ciclpkmk6ys4/Selection_357.png


Process
---------

.. image:: http://static.zybuluo.com/zzc/81wfetbemid0wnruynzn3oox/image.png

1. :ref:`Mconfig-BluFi`: The App implements network configuration for a single device through Bluetooth. It can transfer and add customized data segments to BLE advertising packet and configuration information;
    a. The App scans the Bluetooth packet of the devices, and generates a white list, i.e. a device list naming the devices in alphabetical order (A to Z). It will then connect and transfer the configuration information and the device list to Device (A) that has the most intense signal; 
    b. Device (A) uses the configuration information to connect to AP;
    c. Device (A) checks if the configuration information is correct according to the status returned by AP;
    d. Device (A) returns the network configuration status to the App, and meanwhile:
        - In case of a correct message: notifies other devices to request for the information of network configuration through Wi-Fi beacon;
        - In case of an error message: returns the possible failure reasons to the App, and goes back to the original status in step a.

2. :ref:`Mconfig-Chain`: The Master transfers the information of network configuration to the Slave through Wi-Fi.
    a. Device (A) sends the notification to Devices (B) and (E). Upon the receipt of the notification, Devices (B) and (E) send the requests for network configuration to Device (A);
    b. Device (A) checks the requesting devices;
        - If Devices (E) is not in the device list, its request will be ignored;
        - If Devices (B) is in the device list, Device (A) will send to it the information of network configuration.
    c. Device (B) repeats step 2 and implements the network configuration for Devices (C) and (D) when it receives the information of network configuration.

3. `Building a network <https://docs.espressif.com/projects/esp-idf/zh_CN/latest/api-guides/mesh.html#building-a-network>`_
    a. Root node selection: Devices (A), (B), (C) and (D) elect Device (A) as a root node according to the signal intensity between each node and the router;
    b. Second layer formation: Once the elected root node is connected to the router, the idle node Device (B) within the coverage of the root node will connect to Device (A), thereby forming the second layer of the network;
    c. Formation of remaining layers: Devices (C) and (D) connect to Device (B) that functions as an intermediate parent node, thereby forming leaf nodes.

4. Add a new device
    a. The App scans the Bluetooth advertising packet of the devices, and a prompt will pop up asking whether Devices (E) and (F) need to be added. If Device (F) is to be added, the App will send a command to add this new Device (F); 
    b. Device (A) as a root node receives the command and forwards it to the devices in Mesh network. When other Devices (B), (C) and (D) receive this command as well to add Device (F), Mconfig-Chain will be activated;
    c. Device (F) sends a request for network configuration to Device (C) which it considers has the most intense signal; 
    d. Device (C) sends the information of network configuration to Device (F) through Wi-Fi;
    e. Device (F) connects to the Mesh network when it receives the information of network configuration.

Security
---------

- Data security
    - RSA, an algorithms that features asymmetric encryption, is used to generate a random public key;
    - AES, an algorithms that features symmetric encryption, is used to encrypt all the information of network configuration to ensure the security of data transmission.

- Device security
    - Mconfig-BluFi: Uses the App to screen the devices based on their signal intensity to avoid potential attack from the disguised devices;
    - Mconfig-Chain: Sets a window period for network configuration, and only the request for network configuration in this period can be accepted, to avoid potential attack from the disguised devices.

- ID security (sign is optional)
    - Downloads a public/private RSA key pair to every device, and upload the public key to cloud for future verification;
    - Encrypts the flash partition that stores the public/private RSA key pair.

Notice
---------

If you want to customize network configuration, please be sure to:

- Verify password: The non-root nodes in ESP-MESH don't check the router information. Instead, they only check if the configuration within ESP-MESH network is correct;
- Configure channels: IOS can't acquire the information of its connected router, and therefore the channel information can only be obtained by the device;
- Configure BSSID: When ESP-MESH connects to a hidden router, the BSSID of this router must be configured.

.. ---------------------- Mconfig-BluFi --------------------------

.. _Mconfig-BluFi:

Mconfig-BluFi
--------------

Mconfig-BluFi is a network configuration method based on BluFi (a Bluetooth network configuration protocol defined by Espressif), with additional features such as advertising packet definition, RSA encryption and ID authentication. It generally involves the hardwares like mobiles, devices and routers. The network configuration process consists of four phases: Device Finding, Key Agreement, Data Communication, and Verification of Network Configuration.

.. image:: http://static.zybuluo.com/zzc/ebd1cbmlresw0lf7joffggeu/image.png

.. note::

    To use Mconfig-BluFi, the Bluetooth protocol stack must be enabled, and meanwhile note the followings:

    1. Firmware size: As the firmware size will increase about 500 KB, it is recommended to modify the flash partition table and ensure the size of the partition that stores the firmware exceeds 1 MB;
    2. Memory usage: Extra 30 KB of the memory will be used, and if you want to release this used memory, please be aware that the Bluetooth will only function as usual after reboot.


Device Finding
^^^^^^^^^

While the devices send Bluetooth advertising packets periodically through BLE, the App scans to receive the packets, and screens them based on their signal intensity. It then generates a white list to avoid adding wrong surrounding devices to its network. Please find the process shown below:

.. image:: http://static.zybuluo.com/zzc/780b7i6txyn8kkzyw3qqm77j/image.png

There are two types of Bluetooth advertising packet: Advertising Data and Scan Response. Advertising Data is used to store the customized data of a specific product, while Scan Response is used to store the information of network configuration.

- Advertising Data
    1. The maximum length is 31 Byte;
    2. The data format must meet the requirements of `Bluetooth Specification <https://www.libelium.com/forum/libelium_files/bt4_core_spec_adv_data_reference.pdf>`_.

- Scan Response
    1. 设备名称: 10 Byte，
    2. 厂家信息: 14 Byte, see the details in the table below:

    .. image:: http://static.zybuluo.com/zzc/ojeghgxxxw46najfmcy9lc3l/Selection_356.png

Key Agreement
^^^^^^^^^

1. The App connects and sends the request for network configuration through BLE to Device (A) that has the most intense signal;
2. Device (A) receives the request and returns public RSA key to the App;
3. The App verifies the validity of the public RSA key;
4. The App randomly generates a 128-bit key, and encrypts it with the  public RSA key. Later this encrypted key will be sent to Device (A);
5. Device (A) decrypts its received data with the private RSA key, to obtain the aforementioned 128-bit key which will then be encrypted with AES to secure the data transmission between the App and Device (A).

Data Communication
^^^^^^^^^

The App combines the information of network configuration and the device list into a data packet, and then transfers it as BluFi Custom data.

- TLV (Type-length-value or Tag-length-value) is used to differentiate data:

    .. image:: http://static.zybuluo.com/zzc/oux6p8ap5jo3sgdurd74w8ik/Selection_359.png

- Please find the table below with the detailed descriptions of different types of data fields:
    .. image:: http://static.zybuluo.com/zzc/rmk4xn2y8o9xmvm1a0084htn/image.png

Verification of Network Configuration
^^^^^^^^^

When the device receives the information of network configuration from AP, it will connect to AP to verify that the information is correct, and return the connection status as well as the verification result to the App, which is shown below:

.. image:: http://static.zybuluo.com/zzc/jwpvk0a5dree9ikk2baq5wpx/image.png

.. ---------------------- Mconfig-Chain --------------------------

.. _Mconfig-Chain:

Mconfig-Chain
--------------

Mconfig-Chain is a network configuration protocol for devices based on `ESP-NOW <https://docs.espressif.com/projects/esp-idf/zh_CN/latest/api-reference/wifi/esp_now.html?highlight=espnow>`_, a kind of connectionless Wi-Fi communication protocol defined by Espressif.

Currently, there are three ways of Wi-Fi network configuration: BLE, sniffer and softAP, all of which are designed for the network configuration of a single device. Therefore, these three ways are not applicable for ESP-MESH network that usually involves the network configuration of multiple devices. Mconfig-Chain, specially designed for ESP-MESH network configuration, features a chained and transferable configuration process, which means each device that has been connected to the network can implement network configuration for other devices. With Mconfig-Chain, it is much more efficient to realize a wide-range network configuration.

Mconfig-Chain classifies devices into two types: Master (a device that initiates a connection) and Slave (a device that accepts a connection request). The network configuration process consists of three phases: Device Finding, Key Agreement, and Data Communication.

.. image:: http://static.zybuluo.com/zzc/fayq49vhvsjk7shvgq37qyp5/image.png

Device Finding
^^^^^^^^

1. Master adds the identification of chained network configuration to Vendor IE in Wi-Fi beacon, and awaits the request for network configuration from Slave;
    - Examples of Vendor IE identification are shown below:

    .. image:: http://static.zybuluo.com/zzc/2q1t2hfxem703lroc0s53udo/Selection_360.png

    - Master sets a window period, and only the request from Slave in this period can be accepted;
    - The identification of chained network configuration is sent through Wi-Fi beacon, and if a device is in STA mode only, Master can't be activated;

2. Slave enables Wi-Fi Sniffer, which keeps switching channels to sniff Wi-Fi advertising packets, in order to find the identification of chained network configuration. When it finds Masters, it will stop switching channels, and send the request for network configuration to the Master with the most intense signal.
     - Slave usually switches channels when in operation. Therefore, prior to its use, the function of network self-forming in ESP-MESH should be disabled.

Key Agreement
^^^^^^^^^

1. Master receives the request for network configuration from Slave, and checks if Slave is in the white list. If the device ID authentication needs to be enabled, it is necessary to implement MD5 algorithms for the public RSA key received by the device, and check the validity of the RSA key against the white list;
2. Master removes Vendor IE identification of chained network configuration in Wi-Fi beacon;
3. Master randomly generates 128-bit data as the key to communicate with Slave, encrypts it with the received public RSA key, and then send the encrypted key to Slave through ESP-NOW;
4. Slave receives the Response from Master, and decrypts it with the private RSA key to acquire the communication key with Master.

Data Communication
^^^^^^^^^

1. Master uses a key to encrypt the information of network configuration and the white list with AES algorithms, and send it to Slave through ESP-NOW;
2. Slave uses the key to decrypt the data it receives with AES algorithms, and completes network configuration. It then stops to function as Slave and switches to Master mode.

.. Note::

     As ESP-NOW implements data encryption on the data link layer, it is recommended to use the identical key whenever encrypting a product, and the key should be written in flash or directly downloaded to the firmware.