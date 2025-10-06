## Overview

The Tenda AC15 firmware has a stack-overflow vulnerability in `POST /goform/setNotUpgrade`. When the `action` parameter is set to `1`, the handler takes the `newVersion` POST parameter, stores it in local variable `Var`, and writes it via `SetValue("not.notice.version", Var)`. This configuration item is later retrieved with `GetValue("not.notice.version",s)` through `GET /goform/GetRouterStatus`. Because neither `SetValue` nor `GetValue` impose length restrictions, a large `newVersion` value can overflow the stack buffer `s`, leading to denial-of-service or potential remote code execution.

## Affected version

V15.03.05.18

## Firmware download website 

https://www.tenda.com.cn/download/detail-2710.html

## Vulnerability details

**Trigger condition:** Requires `action=1` for the vulnerable branch to execute.

**Endpoint:** `POST /goform/setNotUpgrade`

**Input handling:** The `newVersion` parameter is parsed into local variable `Var`.

**Storage:** `SetValue("not.notice.version", Var)` persists the supplied value into configuration.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_151001.png)

**Retrieval:** The stored value can later be accessed via `GetValue("not.notice.version",s)` in `GET /goform/GetRouterStatus`.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_151315.png)

**Root cause:** The firmware does not validate the length of `not.notice.version`. When the stored configuration value is later copied into a fixed-size local buffer `s`, overly long input overflows the stack.

**Impact:** A crafted request can corrupt the stack, resulting in router crashes or, if mitigations (stack canaries, NX, ASLR) are absent or bypassed, arbitrary code execution.

## Exploit

1. Use the HackBar tool to modify the `newVersion` through the `/goform/setNotUpgrade` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_151628.png)

2. Trigger Stack Overflow through the `/goform/GetRouterStatus` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_151701.png)

3. Successfully triggered attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_102621.png)