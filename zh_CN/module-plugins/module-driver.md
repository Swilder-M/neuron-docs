# 应用和设备说明

本节主要介绍了北向应用和南向设备的参数配置，南向设备的点位信息配置规范。

**注意** uint16 对应 word 类型。uint32 对应 dword 类型。

## MQTT

Neuron 从设备采集到的数据可以通过 MQTT 应用程序传输到 MQTT Broker 中，用户也可以通过 MQTT 应用程序向 Neuron 发送指令。

### 应用配置

| 字段           | 说明                         |
| ------------- | ---------------------------- |
| **client-id** | MQTT 客户端 ID                |
| **ssl**       | 是否启用 mqtt ssl，默认 false  |
| **host**      | MQTT Broker 主机              |
| **port**      | MQTT Broker 端口号            |
| **username**  | 连接到 Broker 时使用的用户名    |
| **password**  | 连接到 Broker 时使用的密码      |
| **ca-path**   | ca 路径                       |
| **ca-file**   | ca 文件                       |

## Modbus TCP

Modbus 协议包括三种协议：Modbus TCP、Modbus RTU 和 Modbus RTU over TCP。

### 设备配置

| 字段                  | 说明                                                    |
| -------------------- | ------------------------------------------------------- |
| **connection mode** | 驱动程序连接到设备的方式，默认为 client，即把 Neuron 作为客户端使用 |
| **host**            | 当 Neuron 作为客户端使用时，host 指远程设备的 IP。当 Neuron 作为服务端使用时，host 指 Neuron 在本地使用的 IP，默认可填写 0.0.0.0  |
| **port**           | 当 Neuron 作为客户端使用时，post 指远程设备的 TCP 端口。当 Neuron 作为服务端使用时，host 指 Neuron 在本地使用的 TCP 端口，默认为 502  |

### 支持的数据类型

* INT16
* INT32
* UINT16
* UINT32
* FLOAT
* BIT
* STRING

### 地址格式

> SLAVE!ADDRESS\[.BIT][#ENDIAN]\[.LEN\[H]\[L]\[D]\[E]]</span>

#### **SLAVE**

必填，指从机地址或者是站点号。

#### **ADDRESS**

必填，指寄存器地址。Modbus 协议有四个区域，每个区域最大有 65536 个寄存器，每个区域的地址范围如下表所示。需要注意的是实际应用中一般不需要 65536 这么大的存储区，一般 PLC 厂家普遍采用 10000 以内的地址范围，请注意根据设备的区域及功能码，正确填写点位地址。

| 区域                       | 地址范围          | 属性        | 寄存器大小     | 功能码        | 数据类型 |
| ------------------------- | ---------------- | ---------- | ------------- | ------------ | ------- |
| coil（线圈）                | 000001 ~ 065536 | 读/写       | 1bit          | 0x01,0x05,0x0f | bit     |
| input（离散量输入）          | 100001 ~ 165536 | 读          | 1bit          | 0x02          | bit     |
| input register（输入寄存器） | 300001 ~ 365536 | 读          | 16bit         | 0x04          | bit,int16,uint16,int32,uint32,float,string |
| hold register（保持寄存器）  | 400001 ~ 465536 | 读/写       | 16bit         | 0x03,0x06,0x10 | bit,int16,uint16,int32,uint32,float,string |

**注意** 一些设备文件会使用功能码和寄存器地址来描述指令，因为寄存器地址编号是从 0 开始的，所以每个区域的寄存器地址范围为 0 ～ 65535。首先，根据功能码确定地址的最高位数，并在寄存器地址上加1，作为 Neuron 的使用地址。

例如，功能码是 0x03，寄存器地址是 0，Neuron 使用的地址是 400001。功能码是 0x02，寄存器地址是 5，Neuron 使用的地址是 100006。

#### **.BIT**

选填，一个寄存器地址的某一位，例如：
| 地址         | 数据类型 | 说明                                                |
| ----------- | ------- | --------------------------------------------------- |
| 1!300004.0  | bit     | 指站号为1，离散量输入区域，地址为 300004，第 0 位    |
| 1!400010.4  | bit     | 指站号为1，保持寄存器区域，地址为 400010，第 4 位    |
| 2!400001.15 | bit     | 指站号为2，保持寄存器区域，地址为 400001，第 15 位   |

#### **#ENDIAN**

选填，字节顺序，适用于 int16/uint16/int32/uint32/float 数据类型，详细说明见下表。
| 符号 | 字节顺序 | 支持的数据类型        | 备注 |
| --- | ------- | ------------------ | ----- |
| #B  | 2,1     | int16/uint16       |       |
| #L  | 1,2     | int16/uint16       | 不填，默认字节顺序 |
| #LL | 1,2,3,4 | int32/uint32/float | 不填，默认字节顺序 |
| #LB | 2,1,4,3 | int32/uint32/float | |
| #BB | 3,4,1,2 | int32/uint32/float | |
| #BL | 4,3,2,1 | int32/uint32/float | |

*例如*：
| 地址         | 数据类型  | 说明       |
| ----------- | -------- | --------- |
| 1!300004    | int16    | 指站号为 1，离散量输入区域，地址为 300004，字节顺序为 #L |
| 1!300004#B  | int16    | 指站号为 1，离散量输入区域，地址为 300004，字节顺序为 #B |
| 1!300004#L  | uint16   | 指站号为 1，离散量输入区域，地址为 300004，字节顺序为 #L |
| 1!400004    | int16    | 指站号为 1，保持寄存器区域，地址为 400004，字节顺序为 #L |
| 1!400004#L  | int16    | 指站号为 1，保持寄存器区域，地址为 400004，字节顺序为 #L |
| 1!400004#B  | uint16   | 指站号为 1，保持寄存器区域，地址为 400004，字节顺序为 #B |
| 1!300004    | int32    | 指站号为 1，离散量输入区域，地址为 300004，字节顺序为 #LL |
| 1!300004#BB | uint32   | 指站号为 1，离散量输入区域，地址为 300004，字节顺序为 #BB |
| 1!300004#LB | uint32   | 指站号为 1，离散量输入区域，地址为 300004，字节顺序为 #LB |
| 1!300004#BL | float    | 指站号为 1，离散量输入区域，地址为 300004，字节顺序为 #BL |
| 1!300004#LL | int32    | 指站号为 1，离散量输入区域，地址为 300004，字节顺序为 #LL |
| 1!400004    | int32    | 指站号为 1，保持寄存器区域，地址为 400004，字节顺序为 #LL |
| 1!400004#LB | uint32   | 指站号为 1，保持寄存器区域，地址为 400004，字节顺序为 #LB |
| 1!400004#BB | uint32   | 指站号为 1，保持寄存器区域，地址为 400004，字节顺序为 #BB |
| 1!400004#LL | int32    | 指站号为 1，保持寄存器区域，地址为 400004，字节顺序为 #LL |
| 1!400004#BL | float    | 指站号为 1，保持寄存器区域，地址为 400004，字节顺序为 #BL |

#### .LEN\[H]\[L]\[D]\[E]

当数据类型为 string 类型时，**.LEN** 是必填项，表示字符串需要占用的字节长度，每个寄存器中包含**H**，**L**，**D** 和**E** 四种存储方式，如下列表格所示。
| 符号 | 说明                                  |
| --- | ------------------------------------- |
| H   | 一个寄存器存储两个字节，高字节在前低字节在后 |
| L   | 一个寄存器存储两个字节，低字节在前高字节在后 |
| D   | 一个寄存器存储一个字节，且存储在低字节      |
| E   | 一个寄存器存储一个字节，且存储在高字节      |

*例如*：
| 地址         | 数据类型 | 说明 |
| ----------- | ------- | --------- |
| 1!300001.10  | String  | 指站号为1，离散量输入区域，地址为 300001，字符长度为 10，字节顺序为 L，即占用的地址为 300001 ～ 300005 |
| 1!300001.10H | String  | 指站号为1，离散量输入区域，地址为 300001，字符长度为 10，字节顺序为 H，即占用的地址为 300001 ～ 300005 |
| 1!300001.10L | String  | 指站号为1，离散量输入区域，地址为 300001，字符长度为 10，字节顺序为 L，即占用的地址为 300001 ～ 300005 |
| 1!400001.10  | String  | 指站号为1，保持寄存器区域，地址为 400001，字符长度为 10，字节顺序为 L，即占用的地址为 400001 ～ 400005 |
| 1!400001.10H | String  | 指站号为1，保持寄存器区域，地址为 400001，字符长度为 10，字节顺序为 H，即占用的地址为 400001 ～ 400005 |
| 1!400001.10L | String  | 指站号为1，保持寄存器区域，地址为 400001，字符长度为 10，字节顺序为 L，即占用的地址为 400001 ～ 400005 |
| 1!400001.10D | String  | 指站号为1，保持寄存器区域，地址为 400001，字符长度为 10，字节顺序为 D，即占用的地址为 400001 ～ 400010 |
| 1!400001.10E | String  | 指站号为1，保持寄存器区域，地址为 400001，字符长度为 10，字节顺序为 E，即占用的地址为 400001 ～ 400010 |

## OPC UA

### 设备配置

| 字段               | 说明                      |
| ----------------- | ------------------------- |
| **endpoint url**  | 远程访问 PLC 的地址，默认值是`opc.tcp://127.0.0.1:4840/` |
| **username**      | 连接到 PLC 时，使用的用户名 |
| **password**      | 连接到 PLC 时，使用的密码 |
| **cert-file**     | 提供登录用户认证的证书 |
| **key-file**      | 私钥文件，用于提供签名和加密传输 |

### 支持的数据类型

* BYTE
* INT8
* INT16
* INT32
* INT64
* UINT8
* UINT16
* UINT32
* UINT64
* FLOAT
* DOUBLE
* BOOL
* BIT
* STRING

### 地址格式

> IX!NODEID</span>

**IX** 命名空间索引。

**NODEID** 节点 ID。

*例如*:

* 2!Device1.Module1.Tag1 指命名空间索引为2，节点 ID 为 Device1.Module1.Tag1。

**注意** 关于命名空间索引和节点 ID 的解释，请参考 OPC UA 标准。

## Siemens S7 ISOTCP

s7comm 插件用于带有网络端口的西门子PLC，如，s7-200/300/400/1200/1500。

### 设备配置

| 字段      | 说明                     |
| -------- | ------------------------ |
| **host** | 远程 PLC 的 IP            |
| **ip**   | 远程 PLC 的端口，默认为 102 |
| **rack** | PLC 机架号，默认为 0       |
| **slot** | PLC 插槽号，默认为 1       |

**注意** 当使用S7COMM插件访问S7 1200/1500 PLC时，你需要使用西门子软件（TIA16）对PLC进行一些设置。( 详细设置请参考 [PLC 设置](./plc-settings/siemens-s7-1200-1500.md) )

* 优化块访问必须被关闭。
* 访问级别必须是**完全**，**连接机制**必须允许 GET/PUT。

### 支持的数据类型

* INT16
* UINT16
* INT32
* UINT32
* FLOAT
* DOUBLE
* BIT
* STRING

### 地址格式

> AREA ADDRESS\[.BIT][.LEN]</span>

#### AREA ADDRESS

| 区域  | 数据类型                                           | 属性   | 备注         |
| ---- | ------------------------------------------------- | ----- | ------------ |
| I    | int16/uint16/bit                                  | 读    | 输入          |
| O    | int16/uint16/bit                                  | 读/写 | 输出          |
| F    | int16/uint16/bit                                  | 读/写 | 标志          |
| T    | int16/uint16/bit                                  | 读/写 | 计时器        |
| C    | int16/uint16/bit                                  | 读/写 | 计数器        |
| DB   | int16/uint16/bit/int32/uint32/float/double/string | 读/写 | 全局数据块     |

*例如*：
| 地址 | 数据类型 | 说明 |
| ------ | ------- | -------- |
| I0         | int16   | I 区域，地址为 0 |
| I1         | uint16  | I 区域，地址为 1 |
| O2         | int16   | O 区域，地址为 2 |
| O3         | uint16  | O 区域，地址为 3 |
| F4         | int16   | F 区域，地址为 4 |
| F5         | int16   | F 区域，地址为 5 |
| T6         | int16   | T 区域，地址为 6 |
| T7         | int16   | T 区域，地址为 7 |
| C8         | uint16  | C 区域，地址为 8 |
| C9         | uint16  | C 区域，地址为 9 |
| DB10.DBW10 | int16   | 10 数据块中，起始数据字为 10 |
| DB12.DBW10 | uint16  | 12 数据块中，起始数据字为 10 |
| DB10.DBW10 | float   | 10 数据块中，起始数据字为 10 |
| DB11.DBW10 | double  | 11 数据块中，起始数据字为 10 |

#### .BIT

选填，指某一地址的某一位。

*例如*：

| 地址         | 数据类型 | 说明                    |
| ----------- | ------- | ---------------------- |
| I0.0        | bit     | I 区域，地址为0，第 0 位  |
| I0.1        | bit     | I 区域，地址为0，第 1 位  |
| O1.0        | bit     | O 区域，地址为1，第 0 位  |
| O1.2        | bit     | O 区域，地址为1，第 2 位  |
| F2.1        | bit     | F 区域，地址为2，第 1 位  |
| F2.2        | bit     | F 区域，地址为2，第 2 位  |
| T3.3        | bit     | T 区域，地址为3，第 3 位  |
| T3.4        | bit     | T 区域，地址为3，第 4 位  |
| C4.5        | bit     | C 区域，地址为4，第 5 位  |
| C4.6        | bit     | C 区域，地址为4，第 6 位  |
| DB1.DBW10.1 | bit     | 1 数据块中，起始数据字为 10，第 0 位  |
| DB2.DBW1.15 | bit     | 2 数据块中，起始数据字为 1，第 15 位  |

#### .LEN

当数据类型为 string 类型时，是必填项，表示字符串长度。

*例如*：

| 地址          | 数据类型 | 说明                                    |
| -------------| ------- | -------------------------------------- |
| DB1.DBW12.20 | string  | 1 数据块中，起始数据字为 12，字符串长度为 20 |

## OMRON FINS on TCP

fins插件用于带有网口的欧姆龙 PLC，如 CP2E。

### 设备配置

| 字段 | 说明 |
| -------- | ------------- |
| **host** | 远程 PLC 的 ID |
| **port** | 远程 PLC 的端口，默认为 9600 ｜

### 支持的数据类型

* UINT8
* INT8
* INT16
* UINT16
* INT32
* UINT32
* FLOAT
* DOUBLE
* BIT
* STRING

### 地址格式

> AREA ADDRESS\[.BIT]\[.LEN\[H]\[L]]</span>

#### AREA ADDRESS

| 区域 | 数据类型                                      | 属性     | 备注        |
| ---- | ------------------------------------------- | ------- | ---------- |
| CIO  | 除 uint8/int8 外的所有类型                    | 读/写    | CIO 区     |
| A    | 除 uint8/int8 外的所有类型                    | 读       | 辅助区     |
| W    | 除 uint8/int8 外的所有类型                    | 读/写    | 工作区     |
| H    | 除 uint8/int8 外的所有类型                    | 读/写    | 保持区     |
| D    | 除 uint8/int8 外的所有类型                    | 读/写    | 数据存储区  |
| P    | 除 uint8/int8 外的所有类型，但 bit 只支持读     | 读/写    | PVs        |
| F    | int8/uint8                                  | 读      | 完成标志    |
| EM   | 除 uint8/int8 外的所有类型                    | 读/写    | 扩展内存    |

*例如*：

| 地址     | 数据类型 | 说明             |
| ------- | ------- | --------------- |
| F0      | uint8  | F 区域，地址为 0   |
| F1      | int8   | F 区域，地址为 1   |
| CIO1    | int16  | CIO 区域，地址为 1 |
| CIO2    | uint16 | CIO 区域，地址为 2 |
| A2      | int32  | A 区域，地址为 2   |
| A4      | uint32 | A 区域，地址为 4   |
| W5      | float  | W 区域，地址为 5   |
| W10     | float  | W 区域，地址为 10   |
| H20     | double | H 区域，地址为 20   |
| H30     | uint32 | H 区域，地址为 30   |
| D10     | int32  | D 区域，地址为 10   |
| D20     | float  | D 区域，地址为 20   |
| EM10    | float  | EM 区域，地址为 10   |

#### .BIT

选填，指某一地址的某一位。

*例如*：
| 地址      | 数据类型 | 说明                   |
| -------- | ------ | ---------------------- |
| CIO0.0   | bit | CIO 区域，地址为 0，第 0 位  |
| CIO1.2   | bit | CIO 区域，地址为 1，第 2 位  |
| A2.1     | bit | A 区域，地址为 2，第 1 位  |
| A2.3     | bit | A 区域，地址为 2，第 3 位  |
| W3.4     | bit | W 区域，地址为 3，第 4 位  |
| W3.0     | bit | W 区域，地址为 3，第 0 位  |
| H4.15    | bit | H 区域，地址为 4，第 15 位  |
| H4.10    | bit | H 区域，地址为 4，第 10 位  |
| D5.2     | bit | D 区域，地址为 5，第 2 位  |
| D5.3     | bit | D 区域，地址为 5，第 3 位  |
| EM10.0   | bit | EM 区域，地址为 10，第 0 位  |

#### .LEN\[H]\[L]

当数据类型是 string 类型时，是必填项，**.LEN** 表示字符串长度，包含 **H** 和 **L** 两种字节顺序，不填默认是 **H** 字节顺序。

*例如*：

| 地址       | 数据类型  | 说明 |
| --------- | -------- | -------- |
| CIO0.20   | string | CIO 区域，地址为 0，字符串长度 20 个字节，字节顺序为 L  |
| CIO1.20H  | string | CIO 区域，地址为 1，字符串长度 20 个字节，字节顺序为 H  |
| A2.10L    | string | A 区域，地址为 2，字符串长度 10 个字节，字节顺序为 L  |
| A2.30     | string | A 区域，地址为 2，字符串长度 30 个字节，字节顺序为 L  |
| W3.40H    | string | W 区域，地址为 3，字符串长度 40 个字节，字节顺序为 H  |
| W3.10     | string | W 区域，地址为 3，字符串长度 10 个字节，字节顺序为 L  |
| H4.15L    | string | H 区域，地址为 4，字符串长度 15 个字节，字节顺序为 L  |
| H4.10     | string | H 区域，地址为 4，字符串长度 10 个字节，字节顺序为 L  |
| D5.20H    | string | D 区域，地址为 5，字符串长度 20 个字节，字节顺序为 H  |
| D5.30     | string | D 区域，地址为 5，字符串长度 30 个字节，字节顺序为 L  |
| EM10.10   | string | EM 区域，地址为 10，字符串长度 10 个字节，字节顺序为 L  |

## Mitsubishi MELSEC-Q E71

qna3e 插件用于通过以太网访问三菱的QnA兼容PLC，包括Q系列（MC）、iQ-F系列（SLMP）和iQ-L系列。

### 设备配置

| 字段 | 说明 |
| -------- | -------------------------- |
| **host** | 远程 PLC 的 ID               |
| **ip**   | 远程 PLC 的端口号，默认为 2000 |

### 支持的数据类型

* INT16
* UINT16
* INT32
* UINT32
* FLOAT
* DOUBLE
* BIT
* STRING

### 地址格式

> AREA ADDRESS\[.BIT]\[.LEN\[H]\[L]]</span>

#### AREA ADDRESS

| 区域 |数据类型 | 属性  | 备注                           |
| ---- | --------- | ---------- | -------------------------------- |
| X    | bit       | 读/写 | 输入继电器  (Q/iQ-F)             |
| DX   | bit       | 读/写 | (Q/iQ-F)                         |
| Y    | bit       | 读/写 | 输出继电器 (Q/iQ-F)            |
| DY   | bit       | 读/写 | (Q/iQ-F)                         |
| B    | bit       | 读/写 | 链接继电器 (Q/iQ-F)              |
| SB   | bit       | 读/写 | 链接专用继电器               |
| M    | bit       | 读/写 | 内部继电器 (Q/iQ-F)          |
| SM   | bit       | 读/写 | 特殊寄存器 (Q/iQ-F)           |
| L    | bit       | 读/写 | 锁存器 (Q/iQ-F)             |
| F    | bit       | 读/写 | 信号器 (Q/iQ-F)             |
| V    | bit       | 读/写 | 边缘继电器 (Q/iQ-F)              |
| S    | bit       | 读/写 | (Q/iQ-F)                         |
| TS   | bit       | 读/写 | 定时器触点 (Q/iQ-F)           |
| TC   | bit       | 读/写 | 定时器线圈 (Q/iQ-F)              |
| SS   | bit       | 读/写 | (Q/iQ-F)                         |
| STS  | bit       | 读/写 | 保持定时器触点 (Q/iQ-F)    |
| SC   | bit       | 读/写 | (Q/iQ-F)                         |
| CS   | bit       | 读/写 | 计数器触点 (Q/iQ-F)         |
| CC   | bit       | 读/写 | 计数器线圈 (Q/iQ-F)            |
| TN   | 所有类型   | 读/写 | 定时器当前值 (Q/iQ-F)     |
| STN  | 所有类型   | 读/写 | 保持定时器 (Q/iQ-F)         |
| SN   | 所有类型   | 读/写 | (Q/iQ-F)                         |
| CN   | 所有类型   | 读/写 | 计数器当前值  (Q/iQ-F)  |
| D    | 所有类型   | 读/写 | 数据寄存器 (Q/iQ-F)           |
| DSH  |           |       |                                  |
| DSL  |           |      |                                  |
| SD   | 所有类型   | 读/写 | 专用寄存器Specical register (Q/iQ-F)       |
| W    | 所有类型   | 读/写 | 链接寄存器 (Q/iQ-F)           |
| WSH  |           |      |                                  |
| WSL  |           |      |                                  |
| SW   | 所有类型   | 读/写 | 链接专用寄存器 (Q/iQ-F)   |
| R    | 所有类型   | 读/写 | 文件寄存器 (Q/iQ-F)           |
| ZR   | 所有类型   | 读/写 | 文件寄存器 File register (Q/iQ-F)           |
| RSH  |           |       |                                  |
| ZRSH |          |       |                                  |
| RSL  |          |       |                                  |
| ZRSL |          |       |                                  |
| Z    | 所有类型  | 读/写  | 索引寄存器 Index register (Q/iQ-F)          |

*例如*：

| 地址   | 数据类型 | 说明 |
| ----- | ------- | ----- |
| X0    | bit     | X 区域，地址为 0    |
| X1    | bit     | X 区域，地址为 1    |
| Y0    | bit     | Y 区域，地址为 0    |
| Y1    | bit     | Y 区域，地址为 1    |
| D100  | int16   | D 区域，地址为 100  |
| D1000 | uint16  | D 区域，地址为 1000 |
| D200  | uint32  | D 区域，地址为 200  |
| D10   | float   | D 区域，地址为 10   |
| D20   | double  | D 区域，地址为 20   |

#### .BIT

选填，表示某一个地址的某一位。

| 地址  | 数据类型 | 说明  |
| ------- | --------- | --------- |
| D20.0 | bit | D 区域，地址为 20，第 0 位|
| D20.2 | bit | D 区域，地址为 20，第 2 位|

#### .LEN\[H]\[L]

当数据类型是 string 类型时，是必填项，**.LEN** 表示的是字符串长度，包含 **H** 和 **L** 两种字节顺序，不填，默认的是 **H** 的字节顺序。
*例如*：

| 地址       | 数据类型 | 说明 |
| --------- | ------- | ----- |
| D1002.16  | string  | D 区域，地址为 1002，字符串长度为 16，字节顺序为 L |
| D1003.16H | string  | D 区域，地址为 1003，字符串长度为 16，字节顺序为 H |

## IEC 60870-5-104

### 设备配置

| 字段 | 说明 |
| ------- | ------------- |
| **host** | 设备 IP |
| **port** | 设备端口号，默认为 2404 |
| **ca** |  公共地址 |
| **interval** | 站点问询时间间隔 |

### 支持的数据类型

* UINT16
* INT16
* FLOAT
* BIT

### 地址格式

> IOA</span>

| IEC 60870-5-104  类型 ID         | 数据类型  |
| :------------------------------ | :------------ |
| M_ME_NB_1、M_ME_TE_1            | uint16/int16 |
| M_ME_NC_1、M_ME_TF_1            | float        |
| M_SP_NA_1、M_SP_TB_1            | bit          |
| M_ME_NA_1、M_ME_TD_1、M_ME_ND_1 | uint16/int16 |

## KNXnet/IP

### 支持的数据类型

* BIT
* BOOL
* INT8
* UINT8
* INT16
* UINT16
* FLOAT

### 地址格式

两种地址格式：

* > GROUP_ADDRESS</span>

代表 KNX 组地址，只能在 Neuron 中写入，属于该组的 KNX 设备将对发送到该组的消息做出响应。

*例如*：

`0/0/1` 是一个 KNX 组地址，只在 Neuron 中写入，属于 `0/0/1` 组的 KNX 设备将对发送到 `0/0/1` 组的消息做出响应。

* > GROUP_ADDRESS,INDIVIDUAL_ADDRESS</span>

代表 KNX 组下的设备地址，只能在 Neuron 中读取。

*例如*：

`0/0/1,1.1.1` 代表 KNX 组地址 `0/0/1`下的设备地址 `1.1.1`，并且在 Neuron 中只读。

## BACnet/IP

### 设备配置

| 字段      | 说明                            |
|--------- | ------------------------------ |
| **host** | BACnet 设备的 ID                |
| **port** | BACnet 设备的端口号，默认为 47808 |

### 支持的数据类型

* FLOAT
* BIT

### 地址格式

> AREA ADDRESS</span>

| 区域  | 地址范围      | 属性    | 数据类型 |  备注        |
| ---- | ------------ | ------ | ------- | ----------- |
| AI   | 0 - 0x3fffff | 读     | float   | 模拟输入      |
| AO   | 0 - 0x3fffff | 读/写  | float   | 模拟输出      |
| AV   | 0 - 0x3fffff | 读/写  | float   | 模拟量        |
| BI   | 0 - 0x3fffff | 读     | bit     | 二进制输入     |
| BO   | 0 - 0x3fffff | 读/写  | bit      | 二进制输出    |
| BV   | 0 - 0x3fffff | 读/写  | bit      | 二进制值      |
| MSI  | 0 - 0x3fffff | 读     | bit      | 多状态输入    |
| MSO  | 0 - 0x3fffff | 读/写  | bit      | 多状态输出    |
| MSV  | 0 - 0x3fffff | 读/写  | bit      | 多状态值     |

*例如*：

| 地址    | 数据属性 | 说明              |
| ------ | ------- | ---------------- |
| AI0    | float   | AI 区域，地址为 0  |
| AI1    | float   | AI 区域，地址为 1  |
| BO10   | float   | BO 区域，地址为 10 |
| BO20   | float   | BO 区域，地址为 20 |
| AV30   | float   | AV 区域，地址为 30 |
| BI0    | bit     | BI 区域，地址为 0  |
| BI1    | bit     | BI 区域，地址为 1  |
| BV3    | bit     | BV 区域，地址为 3  |
| MSI10  | bit     | MAI 区域，地址为 10 |
| MSI20  | bit     | MSI 区域，地址为 20 |
| MSI30  | bit     | MSI 区域，地址为 30 |