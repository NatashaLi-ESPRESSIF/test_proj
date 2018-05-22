ESP-MESH
========

概述
--------—
[Mesh]_ 网络是“多跳(multi-hop)”网络，可以实现无线 mesh 路由的前提条件是需要各个 mesh 节点首先在物理层互联起来，然后 mesh 路由算法在这些物理层链路中选择传输路径，ESP-MESH 是一种高可靠、广覆盖的 WLAN 网络，是一种非常适合于覆盖大面积开放区城(包括室外和室内)的无线区域网络解决方案。

无线 mesh 网络是一种与传统的无线网络完全不同的网络。传统的无线接入技术中，主要采用点到点或者点到多点的拓扑结构。这种拓扑结构中一般都存在一个中心节点，例如移动通信系统中的基站、802.11 无线局域网 (WLAN) 中的接入点 (AP) 等等。中心节点与各个无线终端通过单跳无线链路相连，控制各无线终端对无线网络的访问；同时，又通过有线链路与有线骨干网相连，提供到骨干网的连接。而在 ESP 无线 mesh 网络中，采用树状 mesh 拓扑结构，具有根节点和各分支节点，易于网络扩展及故障隔离。任意设备都可以竞争做根节点，根节点发生故障后网络会自动推举一个新的根节点出现有效解决了树形拓扑对根节点依赖性太大的问题。

.. figure:: ../../_static/mesh_network_architecture.png
    :align: center
    :alt: Mesh Network Architecture

    ESP-MESH 网络架构

简介
------------

ESP-MESH 具备自组织，自修复的功能，组网时间快，控制速度快。
ESP-MESH 根据节点在网络内的功能，定义了三种类型的节点，分别是根节点，中间节点和叶子节点；
- 根节点：有能力构建一个 mesh 网络的节点，是 mesh 网络访问外部 IP 网络的唯一出口，具有网络转发能力；
- 中间节点：叶子节点到根节点之间所有节点的统称，能够在其各自的父节点和子节点之间转发数据；
- 叶子节点：只发送自己产生的数据，不转发其他节点数据的节点。
各节点根据其在 mesh 网络内所处的层级形成父/子节点关系，实现数据的转发。根节点作为 mesh 网络的出口，是与路由器直接连接的节点，能够在其子节点和路由器之间转发数据；路由器的接入设备数量及带宽直接影响到网络出口设备访问外部 IP 网络的吞吐量。

.. figure:: ../../_static/mesh_network_topology.png
    :align: center
    :alt: Mesh Network Topology

    ESP-MESH 网络拓扑结构和数据流


以图(附图)所示的 mesh 网络为例，节点 C、D 是中间节点，也是根节点的子节点；节点 A、B 是节点 C 的子节点，节点 E 是节点 D 的子节点；节点 A、B、E 为没有子节点的叶子节点。
Mesh 基于二层转发，除根节点外其余不需要安装 TCP/IP 协议栈，图节点协议栈（附图）。

功能描述
--------------------

1. Mesh 组网
^^^^^^^^^^^^^^^^^^^^^
**(1) Mesh 配网**

在 ESP-MESH 网络中，路由器是必需的。用户需要为每个节点配置 SSID，密码和信道。如果路由器处于隐藏状态，用户需要为节点配置 BSSID。（对于网状配置解决方案，请参考 ESP-Mesh IoT 解决方案的链接。该链接即将发布。）

Mesh 配网所需的信息包含在 VIE 中，包括节点类型，网络中节点所在的层，网络允许的最大层数，子节点的数量，允许连接到单个节点的最大节点数量等等。

**(2) 推选根节点**

在 mesh 网络内确定未有根节点时，所有设备分别将其与路由器的实时信号强度 (RSSI) 进行广播发送；
所有设备分别进行一次扫描，每个设备从接收到的其他设备与路由器的实时信号强度、以及本身与路由器的实时信号强度中，选出实时信号强度最大的设备作为根节点候选者进行广播发送；
所有设备分别进行再次扫描，每个设备从接收到的其他设备分别选出的根节点候选者中，选出实时信号强度最大的作为新的根节点候选者，并再次进行广播发送，直至选出唯一根节点。
ESP-MESH 可将设备的信号强度等信息在整个 mesh 网络内相互传递，使得所有设备都可以参与比较并投票，以推选出最优根节点。


ESP-MESH also employs methods to accelerate the convergence of the root node election.

**(3) 推选父节点**

ESP-MESH provides a method for selecting the strongest parent node in a mesh network. According to this method, a node obtains information about other nodes from received VIE messages, and generates a set of parent nodes. If the parent set comprises at least two nodes, the one with the highest performance parameter is selected as the preferred parent. According to this method, a preferred parent node is selected because of the node type and the performance parameter of each node in the parent set. This method ensures that the preferred parent is the optimal one, thus reducing packet loss rate which, in turn, improves network performance.

2. 路由生成与维护
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


3. 网络管理
^^^^^^^^^^^^^^^^^^^^^

+-----------------------+------------------------------------------------------------------------------------------+
| 功能              | 描述                                                                             |
+=======================+==========================================================================================+
|  自修复         |Self-healing allows such routing-based network to operate when a node breaks down or when |
|                       |a connection becomes unreliable.                                                          |
|                       |                                                                                          |
|                       |根节点发生异常，譬如断电 |
|                       |处于二层的节点检测到根节点loss后会重新推举新的根节点|
|                       |the root node and all the nodes on the second layer break down, the nodes on the third    |
|                       |layer will initialize root node election and a new root node will be elected eventually.  |
|                       |                                                                                          |
|                       |中间节点及叶子节点发生异常离开网络后 |
|                       |mesh stack会依赖station的重连机制，当重连超过预先设定的次数后放弃重连|
|                       |重新选择新的父节点加入网络                       |
+-----------------------+------------------------------------------------------------------------------------------+
|根节点动态切换       |用户可调用 :cpp:func:`esp_mesh_waive_root` 来实现网络内根节点的切换。     |
|                       |用户可指定新的根节点，也可以交由网络自动选择新的根节点。        |

+-----------------------+------------------------------------------------------------------------------------------+
|根节点冲突处理|只处理连在同一个路由器的根节点的冲突 |
|                       |对于连在相同SSID不同BSSID的路由器的根节点冲突不予处理。     |
+-----------------------+------------------------------------------------------------------------------------------+
|动态切换父节点    |解决移动节点位置不停变化导致与父节点的信号变弱甚至不能正常通信的问题 |
|                       |监测到此类情况发生后 |
|                       |自动重新选择更优的父节点加入网络           |
|                       |                                                                                          |
|                       |解决移动节点位置不停变化导致与父节点的信号变弱甚至不能正常通信的问题 |
|                       |监测到此类情况发生后 |
|                       |自动重新选择更优的父节点加入网络  |
+-----------------------+------------------------------------------------------------------------------------------+
|环路避免，检测与解决  |在选择父节点时剔除已在自身路由表中的节点来避免环路的发生    |
|依靠路径验证机制和能量传递机制来检测环路                                        |
|                       |                                                                                          |
|                       |当检测到环路发生后，主动断开并用定义的reason code告诉对方环路发生。 |
|                       
+-----------------------+------------------------------------------------------------------------------------------+
|信道迁移         |TO-DO                                                                                     |
+-----------------------+------------------------------------------------------------------------------------------+
|孤立节点避免与解决|TO-DO                                                                                     |

+-----------------------+------------------------------------------------------------------------------------------+

4. 数据传输
^^^^^^^^^^^^^^^^^^^^

+-----------------------+------------------------------------------------------------------------------------------+
| 功能              |描述                                                                              |
+=======================+==========================================================================================+
|可靠           |ESP-MESH provides P2P(point-to-point) retransmission on mesh layer.                       |
+-----------------------+------------------------------------------------------------------------------------------+
|上行流程    |在mesh网络的任意一个节点作为父节点时 |
|                       |分别为它的每个子节点的上行数据分配有接收窗口   |
|                       |并对接收窗口的大小进行动态维护   |
|                       |子节点发包前向父节点发送窗口请求 |
|                       |父节点将窗口请求中与子节点待发送包相对应的请求序列号 |
|                       |与父节点从子节点处最近一次收到的包的序列号进行比对|
|                       |并计算出接收窗口的大小回复给子节点    |
|                       |子节点根据回复的接收窗口大小发包                                        |
|                       |                                                                                          |
|                       |另外，还考虑到整个mesh网络的出口只有一个|
|                       |只有网络内的根节点才具有访问外部IP网络的能力 |
|                       |如果网络内其他节点不知道根节点与服务器间的连接状态|
|                       |一味往根节点发送数据包 |
|                       |希望通过根节点转发到外部 IP 网络 |
|                       |此时就会导致发往根节点的数据包出现丢包      |
|                       |或产生没有必要的发包    |
|                     
+-----------------------+------------------------------------------------------------------------------------------+
|Supporting multicast 支持组播包  |只有指定的设备可以收到这个包|
需要通过 API :cpp:func:`esp_mesh_send` 里的 option 来指定设备。|
+-----------------------+------------------------------------------------------------------------------------------+
|Supporting broadcast 支持广播包  |ESP-MESH provides a method to avoid a waste of bandwidth.                                 |
|packets                |                                                                                          |
|                       |1. 中间节点所传输的广播包是从其父节点收到的广播包时 |
|                       |将该广播包复制一份传递给自己|
|                       |并将所述广播包下发给其子节点                              |
|                       |                                                                                          |
|                       |2. 中间节点所传输的广播包是由自己产生广播包时     |
|                       |将该广播包上发给其父节点，并将所述广播包下发给其子节点                |
|                       |                                                                                          |
|                       |3. 中间节点所传输的广播包是从其子节点收到的广播包时|
|                       |将该广播包传递给自己    |
|                       |将所述广播包复制一份上发给其父节点并将所述的广播包下发给中间节点的其余子节点  |                                                                                       |
|                       |4. 叶子节点产生广播包时|
|                       |直接将所述广播包上发给其父节点                                                             |
|                       |                                                                                          |
|                       |5. 根节点所传输的广播包是该根节点产生的广播包时 |
|                       |将所述广播包下发给其子节点                 |
|                       |                                                                                          |
|                       |6. 根节点所传输的广播包是从其子节点收到的广播包时，|
|                       |将所述广播包下发给根节点的其余子节点。|
|                       |                                                                                          |
|                       |7. 任意节点收到始发地址是自己地址的广播包时， |
|                       |将此广播包扔掉                                               |
|                       |                                                                                          |
|                       |8. 任意节点收到来自其父节点的且始发地址是自己子节点地址的广播包时，|
|                       |将此广播包扔掉 |
+-----------------------+------------------------------------------------------------------------------------------+
|Group control          |Firsty users must specify a group ID for the device via :cpp:func:`esp_mesh_set_group_id`.|
|                       |Then when one packet is sent target to this group, only devices in this group can receive |
|                       |it.                                                                                       |
+-----------------------+------------------------------------------------------------------------------------------+

5. 性能
^^^^^^^^^^^^^^

+--------------------+------------------------------------------------------------------------------------------+
| 功能          | 描述                                                                              |
+====================+==========================================================================================+
|组网时间    |少于 15 秒。The time is from tests executed on a network with 50 devices.       |
+--------------------+------------------------------------------------------------------------------------------+
|修复时间       |If a root node breaks down, less than 10 seconds is taken for the network to detect that  |
|                    |and generate a new root. If a parent node breaks down, less than 5 seconds is taken for   |
|                    |its child nodes to detect that and reselect a new parent node.                            |
|                    |The time is also from tests executed on a network with 50 devices.                        |
+--------------------+------------------------------------------------------------------------------------------+
|Layer forward delay |30ms. The delay is from tests executed on a network with 100 devices and all devices did  |
|                    |not enable AMPDU.                                                                         |
+--------------------+------------------------------------------------------------------------------------------+
|丢包率   |max: %0.32 从第二层传到第四层的数据；min: %0.00                        |
|                    |The results are also from tests executed on a network with 100 devices.                   |
+--------------------+------------------------------------------------------------------------------------------+
|网络容量   |根据softAP允许的最大连接数及网络允许的最大层数来确定网络的最大容量|                                                                                |
+--------------------+------------------------------------------------------------------------------------------+

**Note:** All device are configured 6 connections and 6 layers during the above mentioned tests.

6. 安全与加密
^^^^^^^^^^^^^^^^^^^^^^^^^^
**(1) Uses WPA2-PSK**

**(2) AES Encryption for Mesh VIE**

7. 低功耗 (TO-DO)
^^^^^^^^^^^^^^^^^^^^^^^^^^^
**(1) Network Sleep  网络休眠**

**(2) Standalone Station 制作叶子节点**

8. 用户干预网络 (TO-DO)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
+-----------------------+---------------------------------------------------------------------------------------+
| 功能             | 描述                                                                           |
+=======================+=======================================================================================+
|Specifying the node  指定节点  |用户指定某个节点在网络中做根节点，中间节点或者叶子节点 |
|
+-----------------------+---------------------------------------------------------------------------------------+
|指定父节点 |用户为某个节点指定父节点                                  |                                                                                      |
+-----------------------+---------------------------------------------------------------------------------------+
|指定节点所处的层数  |用户为某个节点指定在网络中所处的层数    |
+-----------------------+---------------------------------------------------------------------------------------+

How to Write a Mesh Application
-------------------------------

**ESP-MESH API Error Code**

We suggest that users regularly check the error code and add relevant handlers accordingly.

ESP-MESH Programming Model
--------------------------

**Software Stack is demonstrated below:**

.. figure:: ../../_static/mesh_software_stack.png
    :align: center
    :alt: ESP-MESH Software Stack

    ESP-MESH Software Stack

**System Events delivery is demonstrated below:**

.. figure:: ../../_static/mesh_events_delivery.png
    :align: center
    :alt: System Events Delivery

    ESP-MESH System Events Delivery


ESP-MESH events define almost all system events for any application tasks needed. The events include the Wi-Fi connection status of the station interface, the connection status of child nodes on the softAP interface, and the like. Firstly, application tasks need to register a mesh event callback handler via the [API]_ :cpp:func:`esp_mesh_set_config`. This handler is used for receiving events posted from the mesh stack and the LwIP stack. Application tasks can add relevant handlers to each event.

**Examples:**

(1) Application tasks can use Wi-Fi station connect statuses to determine when to send data to a parent node, to a root node or to external IP network.
(2) Application tasks can use Wi-Fi softAP statuses to determine when to send data to child nodes.

Application tasks can access the mesh stack directly without having to go through the [LwIP]_ stack. The LwIP stack is not necessery for non-root nodes.
:cpp:func:`esp_mesh_send` and :cpp:func:`esp_mesh_recv` are used in the application tasks to send and receive messages over the mesh network.

**Notes:**

Since current ESP-IDF does not support system initializing without calling :cpp:func:`tcpip_adapter_init`, application tasks still need to perform the LwIP initialization and do remember firstly
1. stoping the DHCP server service on the softAP interface
2. stoping the DHCP client service on the station interface.

Code Example:

:cpp:func:`tcpip_adapter_init`;

:cpp:func:`tcpip_adapter_dhcps_stop`;

:cpp:func:`tcpip_adapter_dhcpc_stop`;

The root node is connected with a router. Thus, in the application mesh event handler, once a node becomes the root, the DHCP client service must be started immediately to obtain IP address unless static IP settings is used.


Glossary
--------
.. [MESH] An Espressif network solution
.. [API] Application interface
.. [LwIP] a security protocol