## 产品名词解释

名词 | 描述
-|-
产品 | 设备的集合，通常指一组具有相同功能的设备。物联网平台为每个产品颁发全局唯一的ProductKey。
设备 | 归属于某个产品下的具体设备。物联网平台为设备颁发产品内唯一的证书DeviceName。设备可以直接连接物联网平台，也可以作为子设备通过网关连接物联网平台。
产品证书 | ProductKey和ProductSecret的组合。
设备证书 | ProductKey、DeviceName、DeviceSecret的组合。
ProductKey | 是物联网平台为产品颁发的全局唯一标识。(产品公钥)
DeviceName | 在注册设备时，自定义的或系统生成的设备名称，具备产品维度内的唯一性。支持英文字母、数字、短划线（-）、下划线（_）、at（@）、英文句号（.）和英文冒号（:），长度限制为4~32个字符。为空时，由物联网平台生成一个产品内唯一标识符作为设备的DeviceName。
DeviceSecret | 物联网平台为设备颁发的设备密钥，用于认证加密。需与DeviceName成对使用。
ProductSecret | 由物联网平台颁发的产品密钥，通常与ProductKey成对出现。

## MQTT名词解释

名词 | 描述
-|-
QoS(Quality of Service) | The MQTT specification describes three Quality of Service (QoS) levels: <br/>1. QoS 0, delivered at most once<br/>2. QoS 1, delivered at least once<br/>3. QoS 2, delivered exactly once<br/>

QoS 0 and 1功能如下:

- A **PUBLISH** message with QoS 1 will be acknowledged by the PUBACK message after it has been successfully sent to Cloud Pub/Sub.

- **PUBLISH** messages with QoS 0 do not require PUBACK responses, and may be dropped if there is any jitter along the message delivery path (for example, if Cloud Pub/Sub is temporarily unavailable).

所以，对一些比较频繁(按秒或分钟的)更新的数据, 没有必要确保接收到每次更新，这种情况下建议使用QoS 0。

当对配置进行多次修改时，我们不需要保证每次修改都送达，只要确保最后一次修改后的配置送达即可，这种情况下建议使用QoS 1。

PUBLISH 
## 设备接入

### 创建产品

使用物联网平台的第一步是在控制台创建产品。产品是设备的集合，通常是一组具有相同功能定义的设备集合。例如，产品指同一个型号的产品，设备就是该型号下的某个设备。

### 创建设备

创建产品完成后，需在产品下添加设备，获取设备证书。

参数 | 描述
-|-
产品 | 选择产品。新创建的设备将继承该产品定义好的功能和特性。
DeviceName | 设置设备名称。 设备名称在产品内具有唯一性。支持英文字母、数字、短划线（-）、下划线（_）、at（@）、英文句号（.）和英文冒号（:），长度限制为4~32个字符。
备注名称 | 设置备注名称。

创建设备成功后，将自动弹出添加完成对话框。您可以查看、复制设备证书信息。设备证书由设备的ProductKey、DeviceName和DeviceSecret组成，是设备与物联网平台进行通信的重要身份认证

> 说明 DeviceName可以为空。为空时，由物联网平台生成一个产品内唯一标识符作为设备的DeviceName。

### 设备安全认证

设备接入物联网平台时，需使用密钥进行身份认证。

针对不同的使用环境，物联网平台提供了使用密钥认证的两种认证方案。

- 一机一密：每台设备烧录自己的设备证书（ProductKey、DeviceName和DeviceSecret）。
- 一型一密预注册：同一产品下设备烧录相同产品证书（ProductKey和ProductSecret）。开通产品的动态注册功能，设备通过动态注册获取DeviceSecret。

### 设备获取证书

物理设备可通过两种方式获取物联网平台颁发的设备证书（ProductKey、DeviceName和DeviceSecret）：设备厂商将证书烧录到设备上和设备上电联网后从云端获取证书。

- 烧录设备证书
	- 该方案是设备厂商获取到物联网平台颁发的设备证书后，在产线上将证书烧录到设备。设备上电联网之后，使用该证书连接到物联网平台。
- 设备从云端获取证书
	- 该方案是设备上电联网后，连接到云端服务器获取证书。在生产时，设备厂商无需为此类设备烧录设备证书。设备上电联网后，再从云端服务器获取物联网平台颁发的设备证书。

### 消息通信Topic

Topic是消息发布（Pub）者和订阅（Sub）者之间的传输中介。设备可通过Topic实现消息的发送和接收，从而实现服务端与设备端的通信。为方便海量设备基于Topic进行通信，简化授权操作，物联网平台定义了产品Topic类和设备Topic。本文介绍产品和设备Topic的定义、使用和分类。

#### Topic定义

产品Topic类：产品维度的Topic，是同一产品下不同设备的Topic集合。一个Topic类对一个ProductKey下所有设备通用。

以下是Topic类的使用说明：

- 定义Topic的功能。
	- Topic类格式以正斜线（/）进行分层，区分每个类目。例如：/${productKey}/${deviceName}/user/update。其中，${productKey}和${deviceName}两个类目为既定类目；
	- ${productKey}表示产品的标识符ProductKey。在指定产品的Topic类中，需替换为实际的ProductKey值。
	- ${deviceName}表示设备名称（DeviceName）。在产品Topic类中，${deviceName}是该产品下所有设备的名称变量，不需要替换为实际设备名称。

- 定义Topic的操作权限。
	- **发布**：产品下设备可以往该Topic发布消息。
	- **订阅**：产品下设备可以订阅该Topic，从而获取消息。
	- **发布和订阅**：同时具备发布和订阅的操作权限。


类别 | 说明
-|-
基础通信Topic | 物联网平台预定义的基础功能通信Topic，包含：<br /> 1. OTA升级相关Topic。<br /> 2. 配置更新相关Topic。<br /> 3. 广播Topic。
设备通信Topic | 1. 属性上报。<br /> 2. 属性设置。<br /> 3. 事件上报。<br /> 4. 服务调用。


### MQTT连接

#### 可变报头（variable header）：Keep Alive

CONNECT指令中需包含Keep Alive（保活时间）。保活心跳时间取值范围为30秒~1200秒，建议取值300秒以上。若网络不稳定，请将心跳时间设置长一些。如果心跳时间不在保活时间内，物联网平台会拒绝连接。

#### MQTT的CONNECT报文参数-设备注册

##### 1. 设备发送CONNECT报文，报文中包含动态注册参数，请求建立连接。

CONNECT报文的动态注册参数：

    mqttClientId: clientId+"|securemode=2,authType=register,random=xxxx,signmethod=hmacsha1|"
    mqttUserName: deviceName+"&"+productKey
    mqttPassword: sign_hmac(productSecret,content) 

参数说明：

- **mqttClientId**

参数取值中包含的详细参数如下表所示。

参数 | 说明
-|-
clientId | 客户端ID，可自定义，长度在64个字符内。建议使用设备的MAC地址或SN码，方便您识别区分不同的客户端。
authType | 固定取值为register
random | 随机数。您自定义随机数。
signMethod | 签名算法。(hmacmd5、hmacsha1、hmacsha256。)

- mqttUserName
	- 组成结构：deviceName+"&"+productKey
<br/>示例：device1&al123456789

- mqttPassword
	- 计算方法：sign_hmac(productSecret,content)<br/>
其中，content的值是提交给服务器的必需参数和值（deviceName、productKey、random）按照字母顺序排序、拼接（无拼接符号）的字符串。然后，将content的值通过mqttClientId中的signMethod指定的算法，使用产品的ProductSecret进行签名计算。 <br/>示例：hmac_sha1(h1nQFYPZS0mW****, deviceNamedevice1productKeyal123456789random123)

##### 2. 物联网平台返回CONNECT ACK。

1. 返回0，表示设备动态注册成功。
2. 返回其他值，表示设备动态注册失败。请根据返回的错误码，确定错误原因。

##### 3. 建立连接后，物联网平台通过推送证书的Topic返回认证参数。

1. 一型一密预注册认证方式：Topic为/ext/register，authType取值为register，返回DeviceSecret。

    {
      "productKey" : "xxxxx",
      "deviceName" : "yyyyy",
      "deviceSecret" : "zzzzz"
    }

#### MQTT的CONNECT报文参数-设备连接

使用设备证书（ProductKey、DeviceName和DeviceSecret）连接。

    mqttClientId: clientId+"|securemode=3,signmethod=hmacsha1,timestamp=132323232|"
    mqttUsername: deviceName+"&"+productKey
    mqttPassword: sign_hmac(deviceSecret,content)

- mqttClientId：格式中| |内为扩展参数。
- clientId：表示客户端ID，可自定义，长度不可超过64个字符。建议使用设备的MAC地址或SN码，方便您识别区分不同的客户端。
- securemode：表示目前安全模式，可选值有2（TLS直连模式）和3（TCP直连模式）。
- signmethod：表示签名算法类型。支持hmacmd5，hmacsha1和hmacsha256，默认为hmacmd5。
- timestamp：表示当前时间毫秒值，可以不传递。
- mqttPassword：sign签名需把提交给服务器的参数按字典排序后，根据signmethod加签。
- content的值为提交给服务器的参数（productKey、deviceName、timestamp和clientId），按照参数名称首字母字典排序， 然后将参数值依次拼接。

> 此处 productKey和 deviceName为必填参数， timestamp和 clientId为可选参数。若传入 timestamp或 clientId，必须与 mqttClientId中的设置相同。

示例：

假设clientId = 12345，deviceName = device， productKey = pk， timestamp = 789，signmethod=hmacsha1，deviceSecret=secret，那么使用TCP方式提交给MQTT的参数如下：

    mqttclientId=12345|securemode=3,signmethod=hmacsha1,timestamp=789|
    mqttUsername=device&pk
    mqttPassword=hmacsha1("secret","clientId12345deviceNamedeviceproductKeypktimestamp789").toHexString(); 

加密后的Password为二进制转16制字符串，示例结果为：

    FAFD82A3D602B37FB0FA8B7892F24A477F85****

### 协议

#### 上线流程

设备上线流程，可以按照设备类型，分为直连设备接入与子设备接入。主要包括：设备注册、上线和数据上报三个流程。

直连设备接入有两种方式：

- 使用一机一密方式提前烧录设备证书（ProductKey、DeviceName和DeviceSecret），注册设备，上线，然后上报数据。

- 使用一型一密动态注册提前烧录产品证书（ProductKey和ProductSecret），注册设备， 上线，然后上报数据。

子设备接入流程通过网关发起，具体接入方式有两种：

- 使用一机一密提前烧录设备证书（ProductKey、DeviceName和DeviceSecret），子设备上报设备证书给网关，网关添加拓扑关系，复用网关的通道上报数据。

- 使用动态注册方式提前烧录ProductKey，子设备上报ProductKey和DeviceName给网关，物联网平台校验DeviceName成功后，下发DeviceSecret。子设备将获得的设备证书信息上报网关，网关添加拓扑关系，通过网关的通道上报数据。

> 一机一密认证方法，即预先为每个设备烧录其唯一的设备证书（ProductKey、DeviceName和DeviceSecret）。当设备与物联网平台建立连接时，物联网平台对其携带的设备证书信息进行认证。认证通过，物联网平台激活设备，设备与物联网平台间才可传输数据。

#### 设备身份注册

子设备的MQTT动态注册

- 请求Topic：/sys/{productKey}/{deviceName}/thing/sub/register
- 响应Topic：/sys/{productKey}/{deviceName}/thing/sub/register_reply

请求数据格式：

```
{
  "id": "123",
  "version": "1.0",
  "sys":{
      "ack":0
  },
  "params": [
    {
      "deviceName": "deviceName1234",
      "productKey": "a1234******"
    }
  ],
}
```

响应数据格式：

```
{
  "id": "123",
  "code": 200,
  "data": [
    {
      "iotId": "12344",
      "productKey": "a1234******",
      "deviceName": "deviceName1234",
      "deviceSecret": "xxxxxx"
    }
  ]
}
```

参数说明如下表。

参数 | 类型 | 说明
-|-|-
id | int | 消息ID号
version | String | 协议版本号，目前协议版本号唯一取值为1
sys | Object | 扩展功能的参数，其下包含各功能字段。
ack | Integer | sys下的扩展功能字段，表示是否返回响应数据。<br /> 1: 云端返回响应数据。 <br />0: 云端不返回响应数据。
params | List | 子设备动态注册的参数。
deviceName | String | 子设备的名称。
productKey | String | 子设备的产品ProductKey。
productKey | String | 子设备的产品ProductKey。
iotId | String | 设备的唯一标识ID。
deviceSecret | String | 设备密钥。
code | Integer | 结果信息。

#### 管理拓扑关系

子设备身份注册后，需由网关向物联网平台上报网关与子设备的拓扑关系，然后进行子设备上线。

子设备上线过程中，物联网平台会校验子设备的身份和与网关的拓扑关系。所有校验通过，才会建立并绑定子设备逻辑通道至网关物理通道上。子设备与物联网平台的数据上下行通信与直连设备的通信协议一致，协议上不需要露出网关信息。

删除拓扑关系后，子设备不能再通过网关上线。系统将提示拓扑关系不存在，认证不通过等错误。

##### 添加设备拓扑关系

网关类型的设备，可以通过该Topic上行请求添加它和子设备之间的拓扑关系，返回成功添加拓扑关系的子设备。

- 请求Topic：/sys/{productKey}/{deviceName}/thing/topo/add
- 响应Topic：/sys/{productKey}/{deviceName}/thing/topo/add_reply

请求数据格式：

```
{
  "id": "123",
  "version": "1.0",
  "sys":{
      "ack":0
  },
  "params": [
    {
      "deviceName": "deviceName1234",
      "productKey": "1234556554",
      "sign": "xxxxxx",
      "signmethod": "hmacSha1",
      "timestamp": "1524448722000",
      "clientId": "xxxxxx"
    }
  ]
}
```
响应数据格式：

```
{
  "id": "123",
  "code": 200,
  "data": [
    {
      "deviceName": "deviceName1234",
      "productKey": "1234556554"
    }
  ]
}
```

##### 添加设备拓扑关系

网关类型的设备，可以通过该Topic上行请求删除它和子设备之间的拓扑关系，返回成功删除拓扑关系的子设备。

- 请求Topic：/sys/{productKey}/{deviceName}/thing/topo/delete
- 响应Topic：/sys/{productKey}/{deviceName}/thing/topo/delete_reply

请求数据格式：

```
{
  "id": "123",
  "version": "1.0",
  "sys":{
      "ack":0
  },
  "params": [
    {
      "deviceName": "deviceName1234",
      "productKey": "1234556554"
    }
  ]
}
```
响应数据格式：

```
{
  "id": "123",
  "code": 200,
  "data": [
    {
      "deviceName": "deviceName1234",
      "productKey": "1234556554"
    }
  ]
}
```

#### 获取设备的拓扑关系

- 请求Topic：/sys/{productKey}/{deviceName}/thing/topo/get
- 响应Topic：/sys/{productKey}/{deviceName}/thing/topo/get_reply

网关类型的设备，可以通过该Topic获取该设备和子设备的拓扑关系。

请求数据格式：

```
{
  "id": "123",
  "version": "1.0",
  "sys":{
      "ack":0
  }
}
```

响应数据格式：

```
{
  "id": "123",
  "code": 200,
  "data": [
    {
      "deviceName": "deviceName1234",
      "productKey": "1234556554"
    }
  ]
}
```

#### 发现设备列表上报

- 请求Topic：/sys/{productKey}/{deviceName}/thing/list/found
- 响应Topic：/sys/{productKey}/{deviceName}/thing/list/found_reply

在一些场景下，网关可以发现新接入的子设备。发现后，需将新接入子设备的信息上报云端，然后通过数据流转到第三方应用，选择将哪些子设备接入该网关。

请求数据格式：

```
{
  "id": "123",
  "version": "1.0",
  "sys":{
      "ack":0
  },
  "params": [
    {
      "deviceName": "deviceName1234",
      "productKey": "1234556554"
    }
  ]
}
```

响应数据格式：

```
{
  "id": "123",
  "code": 200
}
```

#### 通知网关添加设备拓扑关系

todo

- 请求Topic：/sys/{productKey}/{deviceName}/thing/topo/add/notify
- 响应Topic：/sys/{productKey}/{deviceName}/thing/topo/add/notify_reply

#### 设备上报属性或事件
