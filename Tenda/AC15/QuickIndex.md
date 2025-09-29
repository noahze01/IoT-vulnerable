## Overview

The Tenda AC15 firmware has a stack-overflow vulnerability in `POST /goform/QuickIndex`. The `mit_linktype` POST parameter is read into local variable `s2` and then stored via `SetValue("wan1.ppoe.userid", v9)`. The stored value can later be retrieved with `GetValue("wan1.ppoe.userid",s)` through `GET /goform/GetDDNSCfg`. Because neither `SetValue` nor `GetValue` enforce input size limits, an attacker can supply an oversized `mit_linktype` string that overflows a stack buffer when copied, potentially causing crashes or leading to code execution.

## Affected version

V15.03.05.18

## Firmware download website 

https://www.tenda.com.cn/download/detail-2710.html

## Vulnerability details

**Endpoint:** `POST /goform/QuickIndex`

**Input handling:** the `PPPOEName` parameter is parsed into variable `v9`.

**Storage:** `SetValue("wan1.ppoe.userid", v9)` persists the input to the device configuration.

**condition**:The POST parameter `mit_linktype` needs to be set to 2

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_143317.png)

**Retrieval:** `GET /goform/fast_setting_get` later calls `GetValue("wan1.ppoe.userid",s)` and uses the stored value.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_142949.png)

**Root cause:** there are no input validation or length restrictions in `SetValue`/`GetValue`. When `wan1.ppoe.userid` is copied into a fixed-size local buffer, an oversized value leads to stack corruption.

**Impact:** an attacker-controlled `wan1.ppoe.userid` value can overflow the stack. This may result in denial-of-service (device reboot/crash), or — depending on mitigations such as stack canaries, NX, and ASLR — enable arbitrary code execution on affected firmware.

## Exploit

1. Use the HackBar tool to modify the `wan1.ppoe.userid` through the `/goform/QuickIndex` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_144123.png)

2. Trigger command injection through the `/goform/fast_setting_get` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_143058.png)

3. Successfully triggered attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_102621.png)

