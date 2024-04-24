# DTU 连接示例

您可选择通过 Neuron Modbus RTU 插件的串口模式直接与 ModBus RTU 设备进行串口通信，也可首先通过有人云 DTU 实现硬件级的串口汇集，并将串口数据转换到网口，之后再连接 Neuron。

DTU 支持数据的双向转换，支持将 RS232、RS485、RS422 等常见的串口数据与 TCP/IP 数据进行相互转换，并通过无线通信网络进行传输。DTU 一般采用的通信方式有 2/3/4G、NB-IoT、LoRaWAN、WIFI 等。

在有人云 DTU 透传模式下，您可直接连接 Neuron 的 ModBus RTU 模块的 Ethernet 模式，；在有人云 DTU ModBus TCP 模式下，您需要连接 ModBus TCP 插件。

本节将以有人云 DTU ModBus TCP 模式为例，演示如何通过 Neuron [Modbus TCP 插件](../../../modbus-tcp/modbus-tcp.md)建立连接。 

<img src="./assets/neuron-dtu.png" alt="neuron-dtu" style="zoom:50%;" />

## DTU 主要配置项

<img src="./assets/DTU.png" alt="dtu-config" style="zoom:50%;" />

其中的关键配置项如下表所示：

| 配置项                  | 说明                                                    |
| -------------------- | ------------------------------------------------------- |
| **工作方式** | 一般 DTU 都支持 TCP、UDP 以客户端或服务端方式进行连接。以及以 Modbus TCP 标准协议传输数据，或是 None（透传）数据|
| **服务器地址** | DTU 作为客户端时，填写 Neuron Modbus 绑定的地址。|
| **本地/远程端口** | DTU 作为客户端时，填写 Neuron Modbus 绑定的端口，DTU 作为服务端时，为 DTU 的端口。 |

:::tip
DTU 一般都支持串口心跳包，或者是使能网络心跳包，以及注册包，这些特性在标准 Modbus 协议中都无法使用，Neuron 目前还无法兼容处理这些特性，使用 Neuron 连接 DTU 时，注意关闭相关选项。
:::


## Client/Server 模式

Client/Server 又称客户/伺服器模式，简称 C/S 模式，是一种网络通讯架构，用以将通讯建立连接的双方以客户端（Clent）与服务器（Server）的身份区分开来。

在 TCP 协议中，客户端属于请求的发起者，主动给服务器发送连接请求，服务端被动等待来自客户端的请求。

Client 和 Server 建立连接的工作流程如下图所示。

<img src="./assets/client_server.png" alt="client_server" style="zoom:40%;" />

## 连接作为 Client 的 Neuron

本节主要介绍 Neuron 作为 Client，DTU 作为 Server 时，Neuron 与 DTU 的相关配置。

Neuron 作为 Client，主动向 DTU 发起连接请求，用户需要保证 Neuron -> DTU 的网络连通性。

### 配置 DTU Server

首先，需要配置 DTU 与串口连接的参数，其次，需要配置 DTU 与 Neuron 建立连接的 Socket 参数，如下图所示。
![tcp-server](./assets/tcp-server.png)

* 工作方式：TCP Server、Modbus TCP；
* 填写未被使用的本地端口，无需填写远程端口；
* 其他参数为可选配置。

### 查看 DTU IP

在配置 Neuron 南向驱动时需要作为 Server 端的 DTU 的 IP，如下图所示。
![dtu-ip-config](./assets/dtu-ip-config.png)

### 配置 Neuron（Client 模式）

在南向驱动管理中建立插件为 modbus-tcp 的节点，并进行驱动配置，如下图所示。

![image-20230712104126402](./assets/neuron-client-config.png)

* 连接模式选择 Client
* IP 地址填写 DTU 的 IP 地址
* 端口号填写 DTU 配置的端口

## 连接作为 Server 的 Neuron

本节主要介绍 Neuron 作为 Server，DTU 作为 Client 时， Neuron 与 DTU 的相关配置。DTU 作为 Client，主动向 Neuron 发起连接请求，用户需要保证 DTU -> Neuron 的网络连通性。

**应用场景**

这种连接方式通常可用于以下场景中，在某些 DTU 使用 4G 上网时，因为 Neuron 无法主动连接到 DTU，所以，Neuron 只能选择 Server 模式，由 DTU 主动连接到 Neuron。

此外，当 Neuron 与 DTU 不在同一局域网内时，也可以将 Neuron 运行环境的局域网 IP 及端口映射到公网，并将 Neuron 作为 Server 端，DTU 作为 Client 端。

### 配置 DTU Client

首先，需要配置 DTU 与串口连接的参数，其次，需要配置 DTU 与 Neuron 建立连接的 Socket 参数，如下图所示。
![tcp-client](./assets/tcp-client.png)

* 远程服务器地址，填写作为 Server 端运行 Neuron 的 IP 地址；
* 本地端口，默认不填写；
* 远程端口，由于每个 TCP Server 端口都会在客户端指定的端口上监听传入的 TCP 流量，因此，需要用户自定一个未被占用的端口，用以客户端与服务端之间进行握手建立连接。

::: tip
可在 Server 端的终端执行以下指令，确定监听端口是否被占用。

```bash
# 确认端口是否占用
$ netstat -anp |grep <port>
```
:::

### 配置 Neuron（Server 模式）

在南向驱动管理中建立插件为 modbus-tcp 的节点，并进行驱动配置，如下图所示。

![image-20230712095508962](./assets/neuron-server-config.png)

* 连接模式选择 Server
* IP 地址填写 0.0.0.0
* Port，填写监听端口

## 数据监控

完成点位的配置后，您可点击 **监控** -> **数据监控**查看设备信息以及反控设备，具体可参考[数据监控](../../../../../admin/monitoring.md)。
