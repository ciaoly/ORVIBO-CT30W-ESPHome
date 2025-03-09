![logo](./assets/logo.png)

# 基于ORVIBO CT30W的ESPHome万能遥控器

## 功能特性

本项目基于ORVIBO CT30W并进行了电路上的修改，通过修改固件的方式刷入ESPHome，实现支持红外和射频的遥控器，但是功能比较简陋, 如需订制请自己修改代码.

本仓库中的代码支持三款拼多多上的开灯器, 其中有两个433MHz射频的和一个红外的. 至于购买链接就不放了, 因为买来了也没法和我的代码通用.

可以参考我写的几篇文章来订制自己的遥控器: 

* [利用欧瑞博CT30W实现廉价版的开关灯机器人](https://blog.chaol.top/archives/97.html)
* [改装遥控器](https://blog.chaol.top/archives/99-1.html)
* [分析射频信号](https://blog.chaol.top/archives/99.html)
* [分析红外信号](https://blog.chaol.top/archives/100.html)

另外对于系统状态指示灯关联了所有ESPHome组件，可以指示设备的状态，灯光的含义如下：

- 当警告处于活动状态时，缓慢闪烁（大约每秒一次）；
- 当出现错误时，快速闪烁（每秒多次）；
- 其他情况保持关闭。

## Web UI

![Web UI](./assets/web-ui.png)

在设备成功启动并接入WiFi后，在浏览器中打开`http://<IP Address>`即可访问Web UI。

## 如何使用

1. 参照[改装遥控器说明](https://blog.chaol.top/archives/99-1.html), 焊接上射频模块
2. 参照[官方文档](https://esphome.io/guides/installing_esphome)完成ESPHome的安装；
3. 将此仓库拉取到本地：

```
git clone https://github.com/ciaoly/ORVIBO-CT30W-ESPHome.git
```

3. 重命名配置文件`secrets.yaml.example`为`secrets.yaml`，根据实际情况修改配置

其中`api_encryption_key`的配置可以使用[官方文档](https://esphome.io/components/api.html#configuration-variables)中所给出随机生成的密钥；`ota_passwd`与`wifi_ap_passwd`可以任意配置一个自己喜欢的密码；`wifi_ssid`与`wifi_passwd`根据实际情况填写即可。

4. 生成工程并尝试编译

```
esphome compile ir_controller.yaml
```

6. 连接USB转TTL串口模块

将USB转TTL串口模块**选择为3.3V模式**，并按照如下关系进行连接（使用杜邦线即可，可能需要焊接操作）：

| USB转TTL串口模块 | ORVIBO CT30W |
|------------------|--------------|
|TXD               |RX            |
|RXD               |TX            |
|GND               |GND           |

之后将USB转TTL串口模块连接到PC上即可（如果需要安装驱动请自行搜索学习）。

7. 编译并开始上传：

```
esphome run ir-controller.yaml
```

**注意**：在编译完成后选择通过串口上传到设备中前，需**将`GPIO0`在拉低情况下上电以进入刷机模式**。

8. 访问WebUI

待完成编译并上传到设备中后，根据提示将`RST`引脚拉低完成复位动作即可开始运行我们编写的程序，随后稍待片刻便可看到日志输出。

在日志中找到`wifi`组件的如下输出：

```
[14:26:55][C][wifi:600]: WiFi:
[14:26:55][C][wifi:428]:   Local MAC: <local MAC address>
[14:26:55][C][wifi:433]:   SSID: <SSID>
[14:26:55][C][wifi:436]:   IP Address: 192.168.1.125
[14:26:55][C][wifi:439]:   BSSID: <AP MAC address>
[14:26:55][C][wifi:441]:   Hostname: 'ir-controller'
[14:26:55][C][wifi:443]:   Signal strength: -49 dB ▂▄▆█
[14:26:55][C][wifi:447]:   Channel: 1
[14:26:55][C][wifi:448]:   Subnet: 255.255.255.0
[14:26:55][C][wifi:449]:   Gateway: 192.168.1.1
[14:26:55][C][wifi:450]:   DNS1: 192.168.1.1
[14:26:55][C][wifi:451]:   DNS2: 0.0.0.0
```

其中`IP Address`便是设备的IP地址，之后便可访问`http://<IP Address>`访问Web UI以控制设备。

## 自定义

本项目基于ORVIBO CT30W开发，因此所用引脚均根据CT30W中的硬件连接所定，但理论上只要是ESP8266都可以使用此项目。理论上`IRremoteESP8266`库中列出[支持的IR协议](https://github.com/crankyoldgit/IRremoteESP8266/blob/master/SupportedProtocols.md)都是可以修改此项目进行适配的，但是因为IR协议众多并且我也没有对应的设备测试，所以这里不再进行描述，如果有问题欢迎提issue与我交流。

### 引脚修改

本项目所使用的5个引脚的功能机修改位置如下：

- `GPIO14`：输出，用于控制红外发射二极管，即实现红外遥控的功能，修改`globals`组件`id`为`ir_send_pin`的`initial_value`的值即可；
- `GPIO5`：输入，用于读取红外接收二极管的电平，实现学码的功能，修改`globals`组件`id`为`ir_recv_pin`的`initial_value`的值即可；
- `GPIO4`：输入，用于读取按键电平，仅当按键按下时才开启红外接收功能，修改`binary_sensor`组件`pin`中的`number`的值即可；
- `GPIO15`：输出，用于控制LED状态，指示当前系统状态，修改`status_led`组件的`pin`的值即可。
- `GPIO13`: 输出, 连接射频模块

# 参考

- https://github.com/crankyoldgit/IRremoteESP8266
- https://esphome.io/guides/getting_started_command_line
- https://www.bilibili.com/opus/967982171394408465?jump_opus=1
- https://github.com/web1n/ORVIBO-CT30W
- https://www.amrzs.net/2024/11/17/orvibo_ct30w_esphome/
