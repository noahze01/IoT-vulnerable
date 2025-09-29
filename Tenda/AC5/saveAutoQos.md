## Overview

The Tenda AC15 firmware contains a stack-overflow vulnerability at `POST /goform/saveAutoQos`. The handler reads the `enable` POST parameter into variable `nptr` and stores it via `SetValue("qos.auto.en", nptr)`. The value is later retrieved with `GetValue("qos.auto.en",nptr)` through `GET /goform/getAdvanceStatus`. Since neither `SetValue` nor `GetValue` enforce input length restrictions, an attacker can provide an oversized `enable` string that overflows a stack buffer when copied, leading to denial-of-service or potential code execution.

## Affected version

V15.03.05.18

## Firmware download website 

https://www.tenda.com.cn/download/detail-2710.html

## Vulnerability details

**Endpoint:** `POST /goform/saveAutoQos`

**Input handling:** the `enable` parameter is assigned to local variable `nptr`.

**Storage:** the code calls `SetValue("qos.auto.en", nptr)` to persist the value into configuration.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_144804.png)

**Retrieval:** the stored value is returned by `GetValue("qos.auto.en",nptr)` when `GET /goform/getAdvanceStatus` is called.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_145039.png)

**Root cause:** `SetValue` and `GetValue` do not validate input size or format. When the stored value is later copied into a fixed-size local buffer, overly long input can overwrite the stack.

**Impact:** a maliciously crafted `enable` value can cause a stack overflow, potentially resulting in a device crash or, depending on the presence of mitigations (stack canaries, NX, ASLR), arbitrary code execution.

## Exploit

1. Use the HackBar tool to modify the `enable` through the `/goform/saveAutoQos` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_145232.png)

2. Trigger command injection through the `/goform/getAdvanceStatus` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_145340.png)

3. Successfully triggered attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_102621.png)
