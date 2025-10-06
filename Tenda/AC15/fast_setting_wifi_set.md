## Overview

The Tenda AC15 firmware contains a stack-overflow vulnerability at `POST /goform/fast_setting_wifi_set`. The handler reads the `loginPwd` POST parameter into local `v17` and calls `SetValue("sys.userpass", v17)`. That configuration item is later returned by `GetValue("sys.userpass",s1)` in `GET /goform/SysToolChangePwd`. Because neither `SetValue` nor `GetValue` enforce length or content restrictions, an attacker-controlled `loginPwd` can overflow fixed-size stack buffers when the stored value is copied for use, potentially causing crashes or enabling control-flow hijacking.

## Affected version

V15.03.05.18

## Firmware download website 

https://www.tenda.com.cn/download/detail-2710.html

## Vulnerability details

Endpoint: `POST /goform/fast_setting_wifi_set`

Input: `loginPwd` from the POST body is parsed into variable `v17`.

Storage: `SetValue("sys.userpass", v17)` persists the value to configuration.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_141806.png)

Retrieval: `GET /goform/SysToolChangePwd` calls `GetValue("sys.userpass",s1)` and uses the stored value.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_141945.png)

Root cause: `SetValue`/`GetValue` perform no length or content validation. Downstream code copies the returned value into a fixed-size stack buffer without adequate bounds checking.

Impact: A maliciously long `loginPwd` can overflow the stack buffer, causing denial-of-service or — depending on firmware build-time/runtime mitigations (stack canaries, NX, ASLR) — possible code execution.

Exploit guidance: omitted. Do not request exploit payloads.

## Exploit

1. Use the HackBar tool to modify the `loginPwd` through the `/goform/fast_setting_wifi_set` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_142126.png)

2. Trigger Stack Overflow Vulnerability through the `/goform/SysToolChangePwd` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_142206.png)

3. Successfully triggered attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_102621.png)
