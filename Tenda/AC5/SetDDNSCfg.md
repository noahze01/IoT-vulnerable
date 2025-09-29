## Overview

The Tenda AC15 firmware exposes a stack-overflow vulnerability in `POST /goform/SetDDNSCfg`. The handler takes the `ddnsEn` POST parameter, assigns it to local variable `Var`, and stores it via `SetValue("adv.ddns1.en", a2)`. This configuration item can later be retrieved with `GetValue("not.notice.version",v39)` in `GET /goform/GetAdvanceStatus`. Since neither `SetValue` nor `GetValue` enforce size restrictions, an oversized `ddnsEn` value can overflow the stack buffer `v39`, leading to crashes or potential code execution.

## Affected version

V15.03.05.18

## Firmware download website 

https://www.tenda.com.cn/download/detail-2710.html

## Vulnerability details

**Endpoint:** `POST /goform/SetDDNSCfg`

**Input handling:** `ddnsEn` is assigned to local variable `a2`.

**Storage:** `SetValue("adv.ddns1.en", a2)` persists the parameter value into configuration.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_145839.png)

**Retrieval:** `GET /goform/GetAdvanceStatus` uses `GetValue("adv.ddns1.en",v39)` to return the stored data.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_150048.png)

**Root cause:** Both `SetValue` and `GetValue` lack input validation and do not enforce maximum lengths. Later use of the retrieved value copies it into a fixed-size stack buffer `v39` without bounds checking.

**Impact:** Attackers can supply an oversized `ddnsEn` string, overflowing the stack. This may cause denial-of-service (device reboot/crash) or, depending on mitigations (stack canaries, NX, ASLR), arbitrary code execution.

## Exploit

1. Use the HackBar tool to modify the `ddnsEn` through the `/goform/SetDDNSCfg` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_150533.png)

2. Trigger command injection through the `/goform/GetAdvanceStatus` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_150436.png)

3. Successfully triggered attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_102621.png)