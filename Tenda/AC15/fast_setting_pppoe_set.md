## Overview

The Tenda AC15 firmware has a stack-overflow vulnerability at `POST /goform/fast_setting_pppoe_set`. The handler reads the `password` POST parameter into a local variable `Var` and calls `SetValue("wan1.ppoe.pwd", Var)`. That configuration item is later exposed via `GetValue("wan1.ppoe.pwd",s)` in `GET /goform/fast_setting_get`. Neither `SetValue` nor `GetValue` enforce length or content limits, so an attacker-controlled `password` can overflow the stack via the `s` buffer when the stored value is used, potentially causing crashes or enabling code-execution vectors.

## Affected version

V15.03.05.18

## Firmware download website 

https://www.tenda.com.cn/download/detail-2710.html

## Vulnerability details

Endpoint: `POST /goform/fast_setting_pppoe_set`

Input handling: `password` from the POST body is parsed into a local variable named `Var`.

Storage: the code invokes `SetValue("wan1.ppoe.pwd", Var)` to persist the username to the device configuration.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_105753.png)

Retrieval: `GET /goform/fast_setting_get` calls `GetValue("wan1.ppoe.pwd",s)` and returns the stored value.

Root cause: `SetValue`/`GetValue` do not validate length or sanitize content; downstream code copies the retrieved value into a fixed-size stack buffer `s` without adequate bounds checking.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_110001.png)

## Exploit

1. Use the HackBar tool to modify the `password` through the `/goform/fast_setting_pppoe_set` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_110355.png)

2. Trigger Stack Overflow Vulnerability  through the `/goform/fast_setting_get` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_110532.png)

3. Successfully triggered attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_102621.png)