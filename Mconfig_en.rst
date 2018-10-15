Mconfig
=========

Mconfig (Mesh Network Configuration) is a network configuration solution for ESP-MESH, which aims to send the configuration information to ESP-MESH devices in a convenient and efficient manner.

Mconfig uses `ESP-Mesh App <https://github.com/EspressifApp/Esp32MeshForAndroid/raw/master/release/mesh.apk>`_. During the network configuration, RSA algorithms is used for key agreement, 128-AES algorithms for data encryption, and CRC algorithms for checking and verification.

Terminology
----

========================= ==========================================================
Terminology               Description
========================= ==========================================================
Device                    Any devices that are connected to or can be connected to ESP-Mesh network.
Whitelist                 A device list that is used to verify the devices during the process of the chained-way network configuration. It includes the device's MAC address list and the MD5 (Message-Digest Algorithm 5) value of the device's public key. The whitelist can be generated through Bluetooth scanning or imported by scanning the device's QR code.
Configuration Information The configuration information related to Router and ESP-MESH.
AES                       Short for Advanced Encryption Standard, and also known as Rijndael in the field of cryptography, AES is a block encryption standard employed by U.S. Federal Government.
RSA                       It's a asymmetric encryption algorithm that is widely used in the areas of public-key encryption and e-commerce.
CRC                       Short for Cyclic Redundancy Check, CRC is a hash function that can generate a short, fixed-length verification code based on the network data packet or PC files. It's mainly used to detect or check any possible errors during data transmission or after data are saved.
========================= ==========================================================

Process
---------
s
.. image:: http://static.zybuluo.com/zzc/81wfetbemid0wnruynzn3oox/image.png

Further to the above illustrated figure, please find the detailed description of the process below:  

1. :ref:`Mconfig-BluFi`: The App implements network configuration for a single device through Bluetooth.
    a. The App scans the Bluetooth packet of the devices, and generates a whitelist, i.e. a device list that includes Devices (A), (B), (C) and (D). It will then connect and transfer the configuration information and the device list to Device (A) that has the most intense signal; 
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

Mconfig-BluFi is a network configuration protocol based on BluFi (a Bluetooth network configuration protocol defined by Espressif), with additional features such as advertising packet definition, RSA encryption and ID authentication. It generally involves the hardwares like mobiles, devices and routers. The network configuration process consists of four phases: Device Finding, Key Agreement, Data Communication, and Verification of Network Configuration.

.. image:: http://static.zybuluo.com/zzc/ebd1cbmlresw0lf7joffggeu/image.png

.. note::

    To use Mconfig-BluFi, the Bluetooth protocol stack must be enabled, and meanwhile note the followings:

    1. Firmware size: As the firmware size will increase about 500 KB, it is recommended to modify the flash partition table and ensure the size of the partition that stores the firmware exceeds 1 MB;
    2. Memory usage: Extra 30 KB of the memory will be used, and if you want to release this used memory, please be aware that the Bluetooth will only function as usual after reboot.


Device Finding
^^^^^^^^^

While the devices send Bluetooth advertising packets periodically through BLE, the App scans to receive the packets, and screens them based on their signal intensity. It then generates a whitelist to avoid adding wrong surrounding devices to its network. Please find the process shown below:

.. image:: http://static.zybuluo.com/zzc/780b7i6txyn8kkzyw3qqm77j/image.png

There are two types of Bluetooth advertising packet: Advertising Data and Scan Response. Advertising Data is used to store the customized data of a specific product, while Scan Response is used to store the information of network configuration.

- Advertising Data
    1. The maximum length is 31 byte;
    2. The data format must meet the requirements of `Bluetooth Specification <https://www.libelium.com/forum/libelium_files/bt4_core_spec_adv_data_reference.pdf>`_.

- Scan Response
    1. Device name uses 10 byte;
    2. Manufacturer information uses 14 byte. See the details in the table below:

========== ======== ====================
Field      Length   Description
========== ======== ====================
company id 2 byte   The only ID assigned to SIG member companies by Bluetooth SIG 
OUI        2 byte   The Mconfig Blufi ID that is used to filter broadcast packet, and it takes the form of 0x4d, 0x44, 0x46, i.e. "MDF"
version    2 bit    The current version is 0
whitelist  1 bit    Whether to enable whitelist filter
security   1 bit    Whether to verify the validity of the devices in the whitelist
reserved   4 bit    Reserved for future extension
sta mac    6 byte   MAC address of the device sta
tid        2 byte   Device type
========== ======== ====================


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

======= ======= ============================
Type    Length  Data
======= ======= ============================
1 byte  1 byte  Varies according to different types
======= ======= ============================

- Please find the table below with the detailed descriptions of different types of data fields:
    .. image:: http://static.zybuluo.com/zzc/rmk4xn2y8o9xmvm1a0084htn/image.png


+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| Type         | Definition                             | Length (byte) | Description                                                                              |
+==============+========================================+===============+==========================================================================================+
|                                                     Network Configuration Information                                                                            |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 1            | BLUFI_DATA_ROUTER_SSID                 | 32            | SSID of thr router                                                                       |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 2            | BLUFI_DATA_ROUTER_PASSWORD             | 64            | Password of the router                                                                   |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 3            | BLUFI_DATA_ROUTER_BSSID                | 6             | MAC address of router, there are more than one router with the same SSID, BSSID field    |
|              |                                        |               | is mandatory. A risk that more than one root is connected with different BSSID will      |
|              |                                        |               | appear. It means more than one mesh network is established with the same mesh ID.        |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 4            | BLUFI_DATA_MESH_ID                     | 6             | Mesh network identification. Nodes with the same mesh ID can communicate with each other.|                                          
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 5            | BLUFI_DATA_MESH_PASSWORD               | 64            | Password of mesh                                                                         |                            +--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 6            | BLUFI_DATA_MESH_TYPE                   | 1             | Device type (only support MESH_IDLE, MESH_ROOT, and MESH_NODE),                          |
|              |                                        |               | MESH_ROOT and MESH_NODE is only used in no routing scheme                                |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
|                                                     Network Configuration Information                                                                            |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 16           | BLUFI_DATA_VOTE_PERCENTAGE             | 1             | Vote percentage threshold for the approval of being a root                               |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 17           | BLUFI_DATA_VOTE_MAX_COUNT              | 1             | Max vote times in self-healing                                                           |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 18           | BLUFI_DATA_BACKOFF_RSSI                | 1             | RSSI threshold for connecting to the root                                                |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 19           | BLUFI_DATA_SCAN_MIN_COUNT              | 1             | Minimum scan times before being a root                                                   |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 20           | BLUFI_DATA_SCAN_FAIL_COUNT             | 1             | Parent selection failure times. If the scan times reach this value, will disconnect with |
|              |                                        |               | the associated children and start self-healing. Default: 60                              |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 21           | BLUFI_DATA_MONITOR_IE_COUNT            | 1             |                                                                                          |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 22           | BLUFI_DATA_ROOT_HEALING_MS             | 2             | Delay time before the start of root healing                                              |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 23           | BLUFI_DATA_ROOT_CONFLICTS_ENABLE       | 1             | Allow more than one root in one network                                                  |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 24           | BLUFI_DATA_FIX_ROOT_ENALBLE            | 1             | Enable fixed root setting for the device                                                 |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 25           | BLUFI_DATA_CAPACITY_NUM                | 2             | Mesh network capacity                                                                    |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 26           | BLUFI_DATA_MAX_LAYER                   | 1             | Configure max layer                                                                      |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 27           | BLUFI_DATA_MAX_CONNECTION              | 1             | Configure Mesh softAP max connection                                                     |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 28           | BLUFI_DATA_ASSOC_EXPIRE_MS             | 2             | Mesh softAP associate expired time                                                       |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 29           | BLUFI_DATA_BEACON_INTERVAL_MS          | 2             | Mesh softAP beacon interval                                                              |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 30           | BLUFI_DATA_PASSIVE_SCAN_MS             | 2             | Mesh sta passive scan time                                                               |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 31           | BLUFI_DATA_MONITOR_DURATION_MS         | 2             | Monitor duration of the parent's weak RSSI. Will switch to a better parent if the RSSI   |
|              |                                        |               | continues to be weak during this duration_ms                                             |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 32           | BLUFI_DATA_CNX_RSSI                    | 1             | RSSI threshold for keeping a good connection with the parent                             |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 33           | BLUFI_DATA_SELECT_RSSI                 | 1             | RSSI threshold for parent selection. Its value should be greater than switch_rssi        |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 34           | BLUFI_DATA_SWITCH_RSSI                 | 1             | RSSI threshold for the reselection of a better parent                                    |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 35           | BLUFI_DATA_XON_QSIZE                   | 1             | The number of mesh buffer queues                                                         |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 36           | BLUFI_DATA_RETRANSMIT_ENABL            | 1             | Enable retransmission on mesh stack                                                      |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 37           | BLUFI_DATA_DROP_ENABLE                 | 1             | In case a root has been changed, enable the new root to drop the previous packet         |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
|                                                     Network Configuration Information                                                                            |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 64           | BLUFI_DATA_WHITELIST                   | 6 * N         | Device address                                                                           |
|              |                                        | 32 * N        | Verify the validity of the public key to avoid forgery device attacks                    |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+


test 1:


+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| Type         | Definition                             | Length (byte) | Description                                                                              |
+==============+========================================+===============+==========================================================================================+
|                                                     Network Configuration Information                                                                            |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 1            | BLUFI_DATA_ROUTER_SSID                 | 32            | SSID of thr router                                                                       |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 2            | BLUFI_DATA_ROUTER_PASSWORD             | 64            | Password of the router                                                                   |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 3            | BLUFI_DATA_ROUTER_BSSID                | 6             | MAC address of router, there are more than one router with the same SSID, BSSID field    |
|              |                                        |               | is mandatory. A risk that more than one root is connected with different BSSID will      |
|              |                                        |               | appear. It means more than one mesh network is established with the same mesh ID.        |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 4            | BLUFI_DATA_MESH_ID                     | 6             | Mesh network identification. Nodes with the same mesh ID can communicate with each other.|                                          
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 5            | BLUFI_DATA_MESH_PASSWORD               | 64            | Password of mesh                                                                         |                            +--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 6            | BLUFI_DATA_MESH_TYPE                   | 1             | Device type (only support MESH_IDLE, MESH_ROOT, and MESH_NODE),                          |
|              |                                        |               | MESH_ROOT and MESH_NODE is only used in no routing scheme                                |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+


test 2:



+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| Type         | Definition                             | Length (byte) | Description                                                                              |
+==============+========================================+===============+==========================================================================================+
| 1            | BLUFI_DATA_ROUTER_SSID                 | 32            | SSID of thr router                                                                       |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 2            | BLUFI_DATA_ROUTER_PASSWORD             | 64            | Password of the router                                                                   |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 3            | BLUFI_DATA_ROUTER_BSSID                | 6             | MAC address of router, there are more than one router with the same SSID, BSSID field    |
|              |                                        |               | is mandatory. A risk that more than one root is connected with different BSSID will      |
|              |                                        |               | appear. It means more than one mesh network is established with the same mesh ID.        |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 4            | BLUFI_DATA_MESH_ID                     | 6             | Mesh network identification. Nodes with the same mesh ID can communicate with each other.|                                          
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 5            | BLUFI_DATA_MESH_PASSWORD               | 64            | Password of mesh                                                                         |                            +--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 6            | BLUFI_DATA_MESH_TYPE                   | 1             | Device type (only support MESH_IDLE, MESH_ROOT, and MESH_NODE),                          |
|              |                                        |               | MESH_ROOT and MESH_NODE is only used in no routing scheme                                |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+

test 3：



+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| Type         | Definition                             | Length (byte) | Description                                                                              |
+==============+========================================+===============+==========================================================================================+
|                                                     Network Configuration Information                                                                            |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 1            | BLUFI_DATA_ROUTER_SSID                 | 32            | SSID of thr router                                                                       |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 2            | BLUFI_DATA_ROUTER_PASSWORD             | 64            | Password of the router                                                                   |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 3            | BLUFI_DATA_ROUTER_BSSID                | 6             | MAC address of router, there are more than one router with the same SSID, BSSID field    |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 4            | BLUFI_DATA_MESH_ID                     | 6             | Mesh network identification. Nodes with the same mesh ID can communicate with each other.|                                          
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 5            | BLUFI_DATA_MESH_PASSWORD               | 64            | Password of mesh                                                                         |                            +--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+
| 6            | BLUFI_DATA_MESH_TYPE                   | 1             | Device type (only support MESH_IDLE, MESH_ROOT, and MESH_NODE),                          |
+--------------+----------------------------------------+---------------+------------------------------------------------------------------------------------------+

Verification of Network Configuration
^^^^^^^^^

When the device receives the information of network configuration from AP, it will connect to AP to verify that the information is correct, and return the connection status as well as the verification result to the App, which is shown below:

====== ============================ ====================
Type   Definition                   Description
====== ============================ ====================
0      ESP_BLUFI_STA_CONN_SUCCESS   Connecting to router succeeds
1      ESP_BLUFI_STA_CONN_FAIL      Connecting to router fails
300    BLUFI_STA_PASSWORD_ERR       Password configuration error
301    BLUFI_STA_AP_FOUND_ERR       Router is not found
302    BLUFI_STA_TOOMANY_ERR        Reach router's maximum number of connections 
====== ============================ ====================

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

=========== ================
Type        Data
=========== ================
Element ID  0xDD
Length      0X04
OUI         0X18, 0XFE, 0X34
Type        0X0F
=========== ================


    - Master sets a window period, and only the request from Slave in this period can be accepted;
    - The identification of chained network configuration is sent through Wi-Fi beacon, and if a device is in STA mode only, Master can't be activated;

2. Slave enables Wi-Fi Sniffer, which keeps switching channels to sniff Wi-Fi advertising packets, in order to find the identification of chained network configuration. When it finds Masters, it will stop switching channels, and send the request for network configuration to the Master with the most intense signal.
     - Slave usually switches channels when in operation. Therefore, prior to its use, the function of network self-forming in ESP-MESH should be disabled.

Key Agreement
^^^^^^^^^

1. Master receives the request for network configuration from Slave, and checks if Slave is in the whitelist. If the device ID authentication needs to be enabled, it is necessary to implement MD5 algorithms for the public RSA key received by the device, and check the validity of the RSA key against the whitelist;
2. Master removes Vendor IE identification of chained network configuration in Wi-Fi beacon;
3. Master randomly generates 128-bit data as the key to communicate with Slave, encrypts it with the received public RSA key, and then send the encrypted key to Slave through ESP-NOW;
4. Slave receives the Response from Master, and decrypts it with the private RSA key to acquire the communication key with Master.

Data Communication
^^^^^^^^^

1. Master uses a key to encrypt the information of network configuration and the whitelist with AES algorithms, and send it to Slave through ESP-NOW;
2. Slave uses the key to decrypt the data it receives with AES algorithms, and completes network configuration. It then stops to function as Slave and switches to Master mode.

.. Note::

     As ESP-NOW implements data encryption on the data link layer, an identical key must be used for the communicated devices, and the key should be written in flash or directly downloaded to the firmware.