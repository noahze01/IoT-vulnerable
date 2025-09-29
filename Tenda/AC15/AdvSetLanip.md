## Overview

The Tenda AC15 firmware exposes a command-injection weakness at `POST /goform/AdvSetLanip`. The handler reads the `lanIp` POST parameter into `s1` and calls `SetValue("lan.ip", s1)`. The `lan.ip` configuration can later be read via `GetValue` through `GET /goform/telnet`. Because neither `SetValue` nor `GetValue` enforce length or content restrictions on stored values, an attacker-controlled `lanIp` can include data that results in command injection when that value is later used in a shell context.

## Affected version

V15.03.05.18

## Firmware download website 

https://www.tenda.com.cn/download/detail-2710.html

## Vulnerability details

Endpoint: `POST /goform/AdvSetLanip`

Input: the POST parameter `lanIp` is parsed into variable `s1`.

Storage: the code invokes `SetValue("lan.ip", s1)` to save the value in the device configuration.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_103110.png)

Retrieval: `GetValue("lan.ip",v3)` is invoked by the `GET /goform/telnet` handler.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_104043.png)

Root cause: Neither `SetValue` nor `GetValue` validate input length or sanitize content. If any code path uses the stored `lan.ip` value in a shell command, script, or other unsafe context without escaping, an attacker-supplied `lanIp` can cause command injection.

Impact: Remote unauthenticated or authenticated attackers (depending on access controls) may be able to execute arbitrary shell commands, disrupt networking, or otherwise compromise the device.

## Exploit

1. Use the HackBar tool to modify the `lan.ip` through the `/goform/AdvSetLanip` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_104556.png)

2. Trigger command injection through the `/goform/telnet` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_105016.png)

3. Successfully triggered attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_102621.png)

