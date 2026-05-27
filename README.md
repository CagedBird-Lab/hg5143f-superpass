# hg5143f-superpass

一键读取宁波移动 HG5143F / 同类中国移动 HG 系列光猫的超级管理员用户名和密码。

> 仅用于你自己拥有或被授权管理的设备。不同地区、运营商、设备型号和固件版本可能不同，不能保证 100% 成功。


## 相关记录

博客记录：

```text
https://cagedbird.cn/2026/05/27/ningbo-mobile-hg5143f-super-password/
```

## 来源与致谢

实现流程整理自：

- https://www.bilibili.com/opus/748434378355376185

原文采用 CC BY-NC-SA 4.0 协议。本文档和脚本仅复现自动化流程，不复制原文图片和大段正文。

## 用法

默认使用 `192.168.1.1`、从 ARP/邻居表获取该地址的 MAC、开启 telnet、登录并读取超管：

```bash
./hg5143f-superpass
```

手动指定光猫 IP：

```bash
./hg5143f-superpass 192.168.1.1
```

手动指定 MAC：

```bash
./hg5143f-superpass 192.168.1.1 --mac 24:b7:da:3b:24:e0
```

JSON 输出：

```bash
./hg5143f-superpass --json
```

如果已经手动开启 telnet：

```bash
./hg5143f-superpass --no-enable-telnet
```

## 流程

### 1. 获取 MAC

脚本默认从系统邻居表 / ARP 表里找网关 MAC。

等价手工命令：

```bash
arp -a
```

假设光猫网关是：

```text
192.168.1.1
```

MAC 是：

```text
24:b7:da:3b:24:e0
```

脚本会规范化成：

```text
24B7DA3B24E0
```

如果 ARP 输出里某段只有一位，例如：

```text
54:e0:5:2a:4f:20
```

会自动补零成：

```text
54E0052A4F20
```

### 2. 开启 Telnet

请求：

```text
http://192.168.1.1/cgi-bin/telnetenable.cgi?telnetenable=1&key=<MAC>
```

例如：

```text
http://192.168.1.1/cgi-bin/telnetenable.cgi?telnetenable=1&key=24B7DA3B24E0
```

成功时返回 HTML，里面包含：

```text
document.writeln('telnet开启');
```

### 3. 登录 Telnet

已知规律：

```text
用户名：admin
密码：  Fh@ + MAC 后 6 位大写十六进制
```

例如：

```text
MAC:  24B7DA3B24E0
密码: Fh@3B24E0
```

### 4. 获取超级管理员账号

脚本默认直接使用实测可用的 `cfg_cmd`：

```text
cfg_cmd get InternetGatewayDevice.DeviceInfo.X_CMCC_TeleComAccount.Username
cfg_cmd get InternetGatewayDevice.DeviceInfo.X_CMCC_TeleComAccount.Password
```

在这台宁波移动 HG5143F 上，`load_cli factory` 会返回 `Permission denied`，所以它不作为默认路径。只有在 `cfg_cmd` 失败时，脚本才尝试 `load_cli factory / show admin_name / show admin_pwd` 作为 fallback。

`get success!value=` 后面的内容就是超级管理员用户名和密码。

## 输出示例

```text
host=192.168.1.1
login_user=admin
login_password=Fh@XXXXXX
method=cfg_cmd
username=CMCCAdmin
password=<device-specific-password>
```

## 依赖

只需要 Python 3 标准库，不依赖 `expect`、`telnet` 命令或第三方包。
