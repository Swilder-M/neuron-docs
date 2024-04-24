# 数采及转发流程

Neuron 是运行在各类物联网边缘网关硬件上的工业协议网关软件，旨在解决工业 4.0 背景下设备数据统一接入难的问题。通过将来自繁杂多样工业设备的不同协议类型数据转换为统一标准的物联网 MQTT 消息，实现设备与工业物联网系统之间、设备彼此之间的互联互通，进行远程的直接控制和信息获取，为智能生产制造提供数据支撑。

Neuron 支持同时为多个不同通讯协议设备、数十种工业协议进行一站式接入及 [MQTT 协议](https://www.emqx.com/zh/mqtt-guide)转换，仅占用超低资源，即可以原生或容器的方式部署在 X86、ARM 等架构的各类边缘硬件中。同时，用户可以通过基于 Web 的管理控制台实现在线的网关配置管理。其强大的性能使得它能够连接数百个工业设备，轻松处理超过 10,000 个数据点。

## 数据采集

1. [**查看可用插件**](../introduction/plugin-list/plugin-list.md)：Neuron 南向插件是实现特定协议以访问外部设备的通信驱动程序，安装并激活相应插件的许可证后，即可使用驱动插件，具体可查看[许可证政策](../introduction/license/license-policy.md)。由于 Neuron 采取了松散耦合架构，每个插件作为独立的线程运行，不会相互干扰，因此您可根据选择选择符合业务需求的插件。

   Neuron 支持的插件列表如下，不同设备所需的配置参数有所不同，您可点击表格链接快速了解不同南向设备的参数配置。

   | 应用领域       | 插件名称                                                     | 应用领域     | 插件名称                                                     |
   | -------------- | ------------------------------------------------------------ | ------------ | ------------------------------------------------------------ |
   | **全球标准**   | [Modbus TCP <br />Modbus TCP QH](./south-devices/modbus-tcp/modbus-tcp.md) | **PLC 驱动** | [Siemens S7 ISO TCP](./south-devices/siemens-s7/s7.md)       |
   |                | [Modbus RTU](./south-devices/modbus-rtu/modbus-rtu.md)       |              | [Siemens S5 FetchWrite](./south-devices/siemens-fetchwrite/fetchwrite.md) |
   |                | [OPC UA](./south-devices/opc-ua/overview.md)                 |              | [Mitsubishi 3E](./south-devices/mitsubishi-3e/overview.md)   |
   |                | [OPC DA](./south-devices/opc-da/overview.md)                 |              | [Mitsubishi 1E](./south-devices/mitsubishi-1e/mitsubishi-1e.md) |
   |                | [EtherNet/IP(CIP)](./south-devices/ethernet-ip/ethernet-ip.md) |              | [Mitsubishi FX](./south-devices/mitsubishi-fx/overview.md)   |
   | **电力**       | [IEC60870-5-104](./south-devices/iec-104/iec-104.md)         |              | [Omron FINS TCP](./south-devices/omron-fins/omron-fins.md)   |
   |                | [IEC61850](./south-devices/iec61850/overview.md)             |              | [Omron FINS UDP](./south-devices/omron-fins/omron-fins-udp.md) |
   |                | [DL/T645-2007](./south-devices/dlt645-2007/dlt645-2007.md)   |              | [Beckhoff ADS](./south-devices/ads/ads.md)                   |
   |                | [DL/T645-1997](./south-devices/dlt645-1997/dlt645-1997.md)   |              | [Panasonic Mewtocol](./south-devices/panasonic-mewtocol/overview.md) |
   | **楼宇自动化** | [BACnet/IP](./south-devices/bacnet-ip/bacnet-ip.md)          |              | [Profinet IO](./south-devices/profinet/profinet.md)          |
   |                | [KNXnet/IP](./south-devices/knxnet-ip/knxnet-ip.md)          |              | <!--Allen-Bradley DF1 with doc to be added-->                |
   | **环境监测**   | [HJ212-2017](./south-devices/hj212-2017/hj212-2017.md)       | **石油行业** | [NON A11](./south-devices/nona11/nona11.md)                  |

2. [**创建南向驱动**](./south-devices/south-devices.md)：根据工业协议为设备通信选择所有必需的南向插件。根据协议规范，每个南向插件只能与一个设备或关联多个设备的一条消息总线建立连接。用户可选择通过插件或[模版](./templates/templates.md)的方式连接南向设备。

3. [**连接南向设备**](./south-devices/south-devices.md#设置组和点位)：通过创建组和点位连接南向设备，创建好组和点位后，即可从数据监控中获取点位的实时值。为方便用户操作，Neuron 支持通过离线 Excel 文件[批量导入](./import-export/import-export)相关配置信息。

::: tip

重复步骤 2 和 3，直到创建了所有必要的驱动程序、组和点位。
:::

## 数据转发

[**创建北向应用**](./north-apps/north-apps.md)：选择需要的北向插件以实现数据的传送。每个北向插件只能连接到一个目的地，如代理或应用程序。目前 Neuron 支持以下北向应用：

- [MQTT](./north-apps/mqtt/overview.md)
- [eKuiper](./north-apps/ekuiper/overview.md)
- [SparkPlugB](./north-apps/sparkplugb/overview.md)
- [WebSocket](./north-apps/websocket/websocket.md)
- [Monitor](./north-apps/monitor/overview.md)

[**订阅南向设备**](./north-apps/north-apps.md#订阅南向数据)

创建北向设备后，还需订阅组。在此步骤中，不需要设置组和点位。北向节点可以订阅在南向节点中创建的任何组。建立订阅后，相应组的数据将按照组的频率持续发布到北向节点。

具体流程如下图所示：

![配置步骤](./assets/config.png)

### [API](../api/http-api.md)

Neuron 还提供一组配置 API，用于工业物联网平台、MES 或其他控制系统集成。
